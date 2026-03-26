# 4.1 Concurrencia

La concurrencia permite que múltiples tareas se ejecuten de forma simultánea
(o aparentemente simultánea), mejorando el rendimiento de las aplicaciones.

## Threads (Hilos)

Un hilo es la unidad más pequeña de ejecución. Java soporta multithreading nativamente.

### Crear hilos: manera 1 — extender Thread

```java
public class TareaImportacion extends Thread {
    private final String nombreArchivo;

    public TareaImportacion(String nombreArchivo) {
        super("Importador-" + nombreArchivo);
        this.nombreArchivo = nombreArchivo;
    }

    @Override
    public void run() {
        System.out.println(Thread.currentThread().getName() + " — Importando: " + nombreArchivo);
        try {
            Thread.sleep(2000); // simula trabajo
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
            System.out.println("Importación interrumpida.");
        }
        System.out.println("Importación de " + nombreArchivo + " completada.");
    }
}

// Usar
TareaImportacion t1 = new TareaImportacion("estudiantes.csv");
TareaImportacion t2 = new TareaImportacion("cursos.csv");
t1.start(); // inicia en nuevo hilo
t2.start();
t1.join();  // espera a que termine t1
t2.join();  // espera a que termine t2
```

### Crear hilos: manera 2 — implementar Runnable

```java
public class TareaNotificacion implements Runnable {
    private final List<String> destinatarios;
    private final String mensaje;

    public TareaNotificacion(List<String> destinatarios, String mensaje) {
        this.destinatarios = destinatarios;
        this.mensaje = mensaje;
    }

    @Override
    public void run() {
        for (String dest : destinatarios) {
            System.out.println("Enviando notificación a " + dest + ": " + mensaje);
            // simular envío
        }
    }
}

Thread hilo = new Thread(new TareaNotificacion(destinatarios, "Examen mañana"), "Notificador");
hilo.start();

// Con lambda (más conciso)
Thread hiloLambda = new Thread(() -> {
    System.out.println("Tarea en segundo plano...");
}, "LambdaThread");
hiloLambda.start();
```

---

## ExecutorService

El `ExecutorService` gestiona un pool de hilos, evitando crear y destruir
hilos constantemente (operación costosa).

```java
import java.util.concurrent.*;

// Pool fijo de 4 hilos
ExecutorService executor = Executors.newFixedThreadPool(4);

// Enviar tareas
for (int i = 1; i <= 10; i++) {
    final int id = i;
    executor.submit(() -> {
        System.out.println("Procesando estudiante " + id +
            " en hilo: " + Thread.currentThread().getName());
        try { Thread.sleep(500); } catch (InterruptedException e) { Thread.currentThread().interrupt(); }
    });
}

// Cerrar el executor (no acepta nuevas tareas, espera que terminen las actuales)
executor.shutdown();
try {
    executor.awaitTermination(30, TimeUnit.SECONDS);
} catch (InterruptedException e) {
    Thread.currentThread().interrupt();
}
System.out.println("Todas las tareas completadas.");
```

### Tareas con resultado — Callable y Future

```java
import java.util.concurrent.*;

Callable<Double> calcularPromedio = () -> {
    // Simula cálculo costoso
    Thread.sleep(1000);
    return 8.5;
};

ExecutorService executor = Executors.newSingleThreadExecutor();
Future<Double> futuro = executor.submit(calcularPromedio);

// Hacer otro trabajo mientras se calcula...
System.out.println("Calculando promedio en segundo plano...");

// Obtener el resultado (bloquea hasta que esté disponible)
try {
    Double promedio = futuro.get(5, TimeUnit.SECONDS);
    System.out.println("Promedio: " + promedio);
} catch (ExecutionException e) {
    System.out.println("Error en el cálculo: " + e.getCause().getMessage());
} catch (TimeoutException e) {
    System.out.println("El cálculo tardó demasiado.");
    futuro.cancel(true);
} finally {
    executor.shutdown();
}
```

---

## Sincronización

Cuando múltiples hilos acceden a datos compartidos, pueden ocurrir **condiciones de carrera**.

```java
// Problema: contador no es thread-safe
public class ContadorEstudiantes {
    private int contador = 0;

    // Sin synchronized — puede dar resultados incorrectos con múltiples hilos
    public void incrementar() {
        contador++; // Operación NO atómica: read -> increment -> write
    }

    public int getContador() { return contador; }
}

// Solución 1: synchronized
public class ContadorSincronizado {
    private int contador = 0;

    public synchronized void incrementar() {
        contador++;
    }

    public synchronized int getContador() { return contador; }
}

// Solución 2: AtomicInteger (más eficiente)
import java.util.concurrent.atomic.AtomicInteger;

public class ContadorAtomico {
    private AtomicInteger contador = new AtomicInteger(0);

    public void incrementar() {
        contador.incrementAndGet(); // operación atómica
    }

    public int getContador() { return contador.get(); }
}
```

### Colecciones thread-safe

```java
import java.util.concurrent.*;

// ConcurrentHashMap — HashMap thread-safe
Map<Integer, Estudiante> mapaSeguro = new ConcurrentHashMap<>();

// CopyOnWriteArrayList — ArrayList thread-safe para muchas lecturas, pocas escrituras
List<String> listaSegura = new CopyOnWriteArrayList<>();

// BlockingQueue — cola thread-safe con bloqueo
BlockingQueue<String> cola = new LinkedBlockingQueue<>(100);
cola.put("Ana");         // bloquea si está llena
String item = cola.take(); // bloquea si está vacía
```

---

## CompletableFuture (Java 8+)

`CompletableFuture` permite composición de tareas asíncronas de forma fluida.

```java
import java.util.concurrent.CompletableFuture;

// Tarea asíncrona simple
CompletableFuture<Void> tarea = CompletableFuture.runAsync(() -> {
    System.out.println("Enviando emails...");
    // simular trabajo
});

// Con resultado
CompletableFuture<List<Estudiante>> futuroEstudiantes = CompletableFuture.supplyAsync(() -> {
    return repositorio.obtenerTodos(); // operación lenta (p.ej. base de datos)
});

// Encadenar operaciones
CompletableFuture<String> resultado = CompletableFuture
    .supplyAsync(() -> repositorio.buscarPorLegajo(1001))
    .thenApply(opt -> opt.orElseThrow(() -> new RuntimeException("No encontrado")))
    .thenApply(e -> "Estudiante: " + e.getNombreCompleto() + " | Promedio: " + e.getPromedio())
    .exceptionally(ex -> "Error: " + ex.getMessage());

System.out.println(resultado.get()); // Bloquea hasta obtener resultado

// Combinar múltiples futuros
CompletableFuture<List<Estudiante>> futuro1 = CompletableFuture.supplyAsync(() -> repositorio.buscarPorCarrera("Sistemas"));
CompletableFuture<List<Estudiante>> futuro2 = CompletableFuture.supplyAsync(() -> repositorio.buscarPorCarrera("Redes"));

CompletableFuture<Void> ambos = CompletableFuture.allOf(futuro1, futuro2);
ambos.thenRun(() -> System.out.println("Ambas consultas completadas.")).get();

// Ejecutar cuando cualquiera termine primero
CompletableFuture<List<Estudiante>> primero = CompletableFuture.anyOf(futuro1, futuro2)
    .thenApply(obj -> (List<Estudiante>) obj);
```

---

## Resumen

| Concepto | Descripción |
|----------|-------------|
| `Thread` | Unidad de ejecución — extiende Thread o implementa Runnable |
| `ExecutorService` | Pool de hilos para reutilización eficiente |
| `Callable` | Como Runnable pero retorna resultado |
| `Future<T>` | Resultado futuro de una tarea asíncrona |
| `synchronized` | Garantiza acceso exclusivo a un método/bloque |
| `AtomicInteger` | Operaciones atómicas sin sincronización manual |
| `CompletableFuture` | Composición fluida de tareas asíncronas (Java 8+) |

**Siguiente:** [4.2 IO y NIO](02-io-nio.md)
