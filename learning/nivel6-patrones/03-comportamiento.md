# 6.3 Patrones de Comportamiento

Los patrones de comportamiento se ocupan de la **comunicación y responsabilidades**
entre objetos.

---

## Observer (Observador)

**Propósito:** Define una dependencia uno-a-muchos. Cuando el sujeto cambia,
todos sus observadores son notificados automáticamente.

```java
// Interfaz observador
public interface ObservadorEstudiante {
    void alCambiarEstado(Estudiante estudiante, String evento);
}

// Sujeto (observable)
public class Estudiante {
    private String nombre;
    private double promedio;
    private List<ObservadorEstudiante> observadores = new ArrayList<>();

    public void agregarObservador(ObservadorEstudiante obs) { observadores.add(obs); }
    public void quitarObservador(ObservadorEstudiante obs) { observadores.remove(obs); }

    private void notificar(String evento) {
        for (ObservadorEstudiante obs : observadores) {
            obs.alCambiarEstado(this, evento);
        }
    }

    public void setPromedio(double promedio) {
        double anterior = this.promedio;
        this.promedio = promedio;
        if (anterior < 6.0 && promedio >= 6.0) notificar("PROMOVIDO");
        else if (anterior >= 6.0 && promedio < 6.0) notificar("REPROBADO");
        else notificar("NOTA_ACTUALIZADA");
    }

    public String getNombre() { return nombre; }
    public double getPromedio() { return promedio; }
    // constructor, etc...
}

// Observadores concretos
public class ServicioEmailObservador implements ObservadorEstudiante {
    @Override
    public void alCambiarEstado(Estudiante e, String evento) {
        switch (evento) {
            case "PROMOVIDO" ->
                System.out.println("[EMAIL] Felicidades " + e.getNombre() + "! Aprobaste.");
            case "REPROBADO" ->
                System.out.println("[EMAIL] " + e.getNombre() + ", debes presentarte a recuperatorio.");
        }
    }
}

public class RegistroAuditoriaObservador implements ObservadorEstudiante {
    @Override
    public void alCambiarEstado(Estudiante e, String evento) {
        System.out.printf("[AUDITORIA] Estudiante %s — Evento: %s — Promedio: %.2f%n",
            e.getNombre(), evento, e.getPromedio());
    }
}
```

---

## Strategy (Estrategia)

**Propósito:** Define una familia de algoritmos, los encapsula en clases separadas
y los hace intercambiables.

```java
// Interfaz estrategia
public interface EstrategiaCalificacion {
    double calcular(List<Double> notas);
    String getNombre();
}

// Estrategias concretas
public class MediaAritmetica implements EstrategiaCalificacion {
    @Override
    public double calcular(List<Double> notas) {
        return notas.stream().mapToDouble(Double::doubleValue).average().orElse(0);
    }
    @Override
    public String getNombre() { return "Media Aritmética"; }
}

public class MediaPonderada implements EstrategiaCalificacion {
    private final List<Double> pesos;

    public MediaPonderada(List<Double> pesos) { this.pesos = pesos; }

    @Override
    public double calcular(List<Double> notas) {
        double suma = 0, pesosTotal = 0;
        for (int i = 0; i < notas.size(); i++) {
            double peso = i < pesos.size() ? pesos.get(i) : 1.0;
            suma += notas.get(i) * peso;
            pesosTotal += peso;
        }
        return pesosTotal > 0 ? suma / pesosTotal : 0;
    }
    @Override
    public String getNombre() { return "Media Ponderada"; }
}

public class MejorDeN implements EstrategiaCalificacion {
    private final int n;
    public MejorDeN(int n) { this.n = n; }

    @Override
    public double calcular(List<Double> notas) {
        return notas.stream()
            .sorted(Comparator.reverseOrder())
            .limit(n)
            .mapToDouble(Double::doubleValue)
            .average().orElse(0);
    }
    @Override
    public String getNombre() { return "Mejor de " + n; }
}

// Contexto
public class MateriaCurso {
    private String nombre;
    private EstrategiaCalificacion estrategia;
    private List<Double> notas = new ArrayList<>();

    public MateriaCurso(String nombre, EstrategiaCalificacion estrategia) {
        this.nombre = nombre;
        this.estrategia = estrategia;
    }

    public void agregarNota(double nota) { notas.add(nota); }

    public double calcularPromedio() { return estrategia.calcular(notas); }

    public void cambiarEstrategia(EstrategiaCalificacion nuevaEstrategia) {
        this.estrategia = nuevaEstrategia;
    }
}

// Uso
MateriaCurso mate = new MateriaCurso("Matemática", new MediaAritmetica());
mate.agregarNota(8.0); mate.agregarNota(7.5); mate.agregarNota(9.0);
System.out.println("Promedio: " + mate.calcularPromedio()); // 8.17

mate.cambiarEstrategia(new MejorDeN(2));
System.out.println("Mejor 2: " + mate.calcularPromedio()); // 8.5
```

---

## Command (Comando)

**Propósito:** Encapsula una solicitud como objeto, permitiendo parametrizar,
encolar, loguear y deshacer operaciones.

```java
// Interfaz comando
public interface Comando {
    void ejecutar();
    void deshacer();
}

// Receptor
public class RegistroNotas {
    private Map<Integer, Double> notas = new HashMap<>();

    public void asignarNota(int legajo, double nota) {
        notas.put(legajo, nota);
        System.out.printf("Nota %.1f asignada al estudiante %d%n", nota, legajo);
    }

    public void quitarNota(int legajo) {
        notas.remove(legajo);
        System.out.println("Nota eliminada del estudiante " + legajo);
    }

    public Double obtenerNota(int legajo) { return notas.get(legajo); }
}

// Comandos concretos
public class ComandoAsignarNota implements Comando {
    private final RegistroNotas registro;
    private final int legajo;
    private final double nuevaNota;
    private Double notaAnterior;

    public ComandoAsignarNota(RegistroNotas registro, int legajo, double nota) {
        this.registro = registro;
        this.legajo = legajo;
        this.nuevaNota = nota;
    }

    @Override
    public void ejecutar() {
        notaAnterior = registro.obtenerNota(legajo); // guarda para deshacer
        registro.asignarNota(legajo, nuevaNota);
    }

    @Override
    public void deshacer() {
        if (notaAnterior != null) {
            registro.asignarNota(legajo, notaAnterior);
        } else {
            registro.quitarNota(legajo);
        }
    }
}

// Invocador con historial
public class GestorComandos {
    private final Deque<Comando> historial = new ArrayDeque<>();

    public void ejecutar(Comando comando) {
        comando.ejecutar();
        historial.push(comando);
    }

    public void deshacer() {
        if (!historial.isEmpty()) {
            Comando ultimo = historial.pop();
            ultimo.deshacer();
        } else {
            System.out.println("No hay operaciones para deshacer.");
        }
    }
}

// Uso
RegistroNotas registro = new RegistroNotas();
GestorComandos gestor = new GestorComandos();

gestor.ejecutar(new ComandoAsignarNota(registro, 1001, 8.5));
gestor.ejecutar(new ComandoAsignarNota(registro, 1001, 9.0)); // corrección
gestor.deshacer(); // vuelve a 8.5
```

---

## Template Method

**Propósito:** Define el **esqueleto de un algoritmo** en la superclase, dejando
que las subclases implementen los pasos específicos sin cambiar la estructura general.

```java
// Clase abstracta con el template
public abstract class GeneradorReporte {

    // Template method — define el algoritmo
    public final void generar(String ruta) {
        abrirArchivo(ruta);
        escribirEncabezado();
        escribirCuerpo();      // abstracto — cada subclase lo implementa
        escribirPie();
        cerrarArchivo(ruta);
        System.out.println("Reporte generado: " + ruta);
    }

    // Pasos con implementación por defecto
    private void abrirArchivo(String ruta) {
        System.out.println("Abriendo archivo: " + ruta);
    }

    protected void escribirEncabezado() {
        System.out.println("=== Reporte Sistema Escolar ===");
        System.out.println("Fecha: " + java.time.LocalDate.now());
    }

    // Paso abstracto — obligatorio implementar
    protected abstract void escribirCuerpo();

    protected void escribirPie() {
        System.out.println("=== Fin del reporte ===");
    }

    private void cerrarArchivo(String ruta) {
        System.out.println("Cerrando archivo.");
    }
}

// Subclases implementan solo el cuerpo
public class ReporteEstudiantes extends GeneradorReporte {
    private final List<Estudiante> estudiantes;

    public ReporteEstudiantes(List<Estudiante> estudiantes) {
        this.estudiantes = estudiantes;
    }

    @Override
    protected void escribirCuerpo() {
        System.out.println("Total estudiantes: " + estudiantes.size());
        for (Estudiante e : estudiantes) {
            System.out.printf("  %d | %-20s | %.2f%n",
                e.getLegajo(), e.getNombreCompleto(), e.getPromedio());
        }
    }
}

public class ReporteCalificaciones extends GeneradorReporte {
    @Override
    protected void escribirCuerpo() {
        System.out.println("Detalle de calificaciones...");
    }

    @Override
    protected void escribirEncabezado() {
        super.escribirEncabezado();
        System.out.println("Período: 2024");
    }
}

// Uso
new ReporteEstudiantes(lista).generar("reporte_estudiantes.txt");
```

---

## Chain of Responsibility

**Propósito:** Pasa una solicitud a lo largo de una cadena de manejadores.
Cada manejador decide si procesa la solicitud o la pasa al siguiente.

```java
// Manejador abstracto
public abstract class ManejadorInscripcion {
    private ManejadorInscripcion siguiente;

    public ManejadorInscripcion setSiguiente(ManejadorInscripcion sig) {
        this.siguiente = sig;
        return sig;
    }

    public void manejar(Estudiante e, Curso c) {
        if (siguiente != null) siguiente.manejar(e, c);
        else System.out.println("Inscripción aprobada para: " + e.getNombreCompleto());
    }
}

// Manejadores concretos
public class ValidarCuotaPagada extends ManejadorInscripcion {
    @Override
    public void manejar(Estudiante e, Curso c) {
        if (!e.isCuotaPagada()) {
            System.out.println("RECHAZADO: " + e.getNombreCompleto() + " tiene cuota pendiente.");
            return;
        }
        System.out.println("OK — Cuota pagada.");
        super.manejar(e, c);
    }
}

public class ValidarCapacidadCurso extends ManejadorInscripcion {
    @Override
    public void manejar(Estudiante e, Curso c) {
        if (c.estaLleno()) {
            System.out.println("RECHAZADO: Curso lleno.");
            return;
        }
        System.out.println("OK — Hay lugar en el curso.");
        super.manejar(e, c);
    }
}

public class ValidarPrerequisitos extends ManejadorInscripcion {
    @Override
    public void manejar(Estudiante e, Curso c) {
        if (!e.cumplePrerequisitos(c)) {
            System.out.println("RECHAZADO: No cumple con los prerequisitos.");
            return;
        }
        System.out.println("OK — Cumple prerequisitos.");
        super.manejar(e, c);
    }
}

// Construir cadena
ManejadorInscripcion cadena = new ValidarCuotaPagada();
cadena.setSiguiente(new ValidarCapacidadCurso())
      .setSiguiente(new ValidarPrerequisitos());

cadena.manejar(estudiante, curso);
```

---

## Resumen de Patrones de Comportamiento

| Patrón | Propósito | Cuándo usar |
|--------|-----------|-------------|
| **Observer** | Notificar cambios a múltiples objetos | Eventos, sistemas reactivos |
| **Strategy** | Algoritmos intercambiables | Múltiples formas de hacer algo |
| **Command** | Encapsular operaciones + undo/redo | Historial, colas de tareas |
| **Template Method** | Esqueleto de algoritmo | Pasos fijos, detalles variables |
| **Chain of Responsibility** | Pasar solicitud por cadena | Validaciones en secuencia |

**Siguiente:** [Nivel 7 — Arquitectura](../nivel7-arquitectura/README.md)
