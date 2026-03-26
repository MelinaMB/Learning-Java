# 3.3 Excepciones

Las excepciones son el mecanismo de Java para manejar situaciones de error de manera
controlada, separando la lógica normal del manejo de errores.

## Jerarquía de Excepciones

```
Throwable
├── Error                    (errores graves del sistema — no atrapar)
│     ├── OutOfMemoryError
│     └── StackOverflowError
└── Exception
      ├── IOException        (Checked — debes manejar)
      ├── SQLException       (Checked)
      ├── ParseException     (Checked)
      └── RuntimeException   (Unchecked — opcional manejar)
            ├── NullPointerException
            ├── ArrayIndexOutOfBoundsException
            ├── ClassCastException
            ├── NumberFormatException
            └── IllegalArgumentException
```

| Tipo | Descripción | Obligatorio manejar |
|------|-------------|---------------------|
| **Checked** | Condiciones externas (archivos, red, BD) | Sí (`try/catch` o `throws`) |
| **Unchecked** (`RuntimeException`) | Errores de programación | No (pero conviene) |
| **Error** | Problemas graves de la JVM | No (no se pueden recuperar) |

---

## try / catch / finally

```java
public class EjemploExcepciones {
    public static void main(String[] args) {
        // Ejemplo básico
        try {
            int resultado = 10 / 0;  // ArithmeticException
        } catch (ArithmeticException e) {
            System.out.println("Error: " + e.getMessage()); // / by zero
        }

        // Múltiples catch
        String[] nombres = {"Ana", "Luis"};
        try {
            System.out.println(nombres[5]);  // ArrayIndexOutOfBoundsException
            Integer.parseInt("no-es-numero"); // NumberFormatException
        } catch (ArrayIndexOutOfBoundsException e) {
            System.out.println("Índice inválido: " + e.getMessage());
        } catch (NumberFormatException e) {
            System.out.println("Formato de número inválido: " + e.getMessage());
        } catch (Exception e) {
            // Captura cualquier otra excepción (siempre al final)
            System.out.println("Error inesperado: " + e.getMessage());
        } finally {
            // Se ejecuta SIEMPRE, con o sin excepción
            System.out.println("Bloque finally ejecutado.");
        }

        // Multi-catch (Java 7+)
        try {
            procesarEntrada("abc");
        } catch (NumberFormatException | IllegalArgumentException e) {
            System.out.println("Error de entrada: " + e.getMessage());
        }
    }
}
```

### finally con recursos (try-with-resources)

```java
// El recurso se cierra automáticamente al salir del bloque try
// La clase debe implementar AutoCloseable
try (var lector = new java.io.BufferedReader(
        new java.io.FileReader("estudiantes.csv"))) {
    String linea;
    while ((linea = lector.readLine()) != null) {
        System.out.println(linea);
    }
} catch (java.io.IOException e) {
    System.out.println("Error leyendo archivo: " + e.getMessage());
}
// lector.close() se llama automáticamente
```

---

## Declarar y propagar excepciones

```java
// "throws" indica que el método PUEDE lanzar esa excepción
// El que llame debe manejarla o propagarla
public static Estudiante buscarEstudiante(int legajo) throws EstudianteNoEncontradoException {
    for (Estudiante e : repositorio) {
        if (e.getLegajo() == legajo) return e;
    }
    throw new EstudianteNoEncontradoException("Estudiante con legajo " + legajo + " no encontrado.");
}

// Quien llama debe manejarla
public static void main(String[] args) {
    try {
        Estudiante e = buscarEstudiante(9999);
        System.out.println(e.getNombreCompleto());
    } catch (EstudianteNoEncontradoException ex) {
        System.out.println("Error: " + ex.getMessage());
    }
}
```

---

## Excepciones Personalizadas

```java
// Excepción checked
public class EstudianteNoEncontradoException extends Exception {
    private final int legajoBuscado;

    public EstudianteNoEncontradoException(String mensaje, int legajo) {
        super(mensaje);
        this.legajoBuscado = legajo;
    }

    public int getLegajoBuscado() {
        return legajoBuscado;
    }
}

// Excepción unchecked (RuntimeException)
public class NotaInvalidaException extends RuntimeException {
    private final double notaIngresada;

    public NotaInvalidaException(double nota) {
        super(String.format("Nota inválida: %.2f. Debe estar entre 0 y 10.", nota));
        this.notaIngresada = nota;
    }

    public double getNotaIngresada() {
        return notaIngresada;
    }
}

// Excepción de capacidad de curso
public class CursoLlenoException extends RuntimeException {
    private final String nombreCurso;
    private final int capacidadMaxima;

    public CursoLlenoException(String nombreCurso, int capacidadMaxima) {
        super(String.format("El curso '%s' está lleno (capacidad: %d).", nombreCurso, capacidadMaxima));
        this.nombreCurso = nombreCurso;
        this.capacidadMaxima = capacidadMaxima;
    }
}

// Uso en el servicio
public class InscripcionServicio {
    private final Map<String, List<Estudiante>> cursos = new HashMap<>();
    private final int CAPACIDAD_MAX = 30;

    public void inscribir(String nombreCurso, Estudiante estudiante) {
        List<Estudiante> inscriptos = cursos.computeIfAbsent(nombreCurso, k -> new ArrayList<>());

        if (inscriptos.size() >= CAPACIDAD_MAX) {
            throw new CursoLlenoException(nombreCurso, CAPACIDAD_MAX);
        }

        inscriptos.add(estudiante);
        System.out.println(estudiante.getNombreCompleto() + " inscrito en " + nombreCurso);
    }

    public void registrarNota(int legajo, double nota) {
        if (nota < 0 || nota > 10) {
            throw new NotaInvalidaException(nota);
        }
        // ... lógica de registro
    }
}
```

---

## Buenas prácticas con excepciones

```java
public class ServicioEstudiante {

    // BIEN: usa excepción específica con información útil
    public Estudiante buscar(int legajo) {
        return repositorio.findById(legajo)
            .orElseThrow(() -> new EstudianteNoEncontradoException(
                "No existe estudiante con legajo: " + legajo, legajo));
    }

    // MAL: atrapar y silenciar la excepción
    public void procesarMal() {
        try {
            hacerAlgo();
        } catch (Exception e) {
            // No hagas esto — oculta el error
        }
    }

    // MAL: atrapar Exception genérica cuando puedes ser específico
    public void procesarMalOtro() {
        try {
            int[] arr = new int[5];
            arr[10] = 1;
        } catch (Exception e) {  // demasiado amplio
            System.out.println("Error");
        }
    }

    // BIEN: loguear y relanzar si no puedes manejar el error aquí
    public void procesarBien() {
        try {
            hacerAlgo();
        } catch (IOException e) {
            System.err.println("Error de IO al procesar: " + e.getMessage());
            throw new RuntimeException("Error al procesar datos", e); // preserva la causa
        }
    }
}
```

> **Reglas de oro para excepciones:**
> 1. Usa excepciones checked para condiciones recuperables (el que llama puede hacer algo)
> 2. Usa unchecked para errores de programación o condiciones imposibles de recuperar
> 3. Nunca silencies una excepción con un catch vacío
> 4. Incluye información útil en el mensaje
> 5. Usa try-with-resources para recursos que deben cerrarse

---

## Resumen

| Concepto | Descripción |
|----------|-------------|
| `try/catch` | Ejecuta código y captura errores |
| `finally` | Se ejecuta siempre, con o sin error |
| `try-with-resources` | Cierra recursos automáticamente |
| `throw` | Lanza una excepción manualmente |
| `throws` | Declara que un método puede lanzar una excepción |
| Checked | Debe manejarse; extiende `Exception` |
| Unchecked | Opcional; extiende `RuntimeException` |

**Siguiente:** [3.4 Streams y Lambdas](04-streams-lambdas.md)
