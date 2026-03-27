# Soluciones Guiadas

Esta sección no reemplaza los ejercicios. Sirve como apoyo cuando quieras comparar tu enfoque con una posible solución.

## Nivel 1

### Variables y tipos
```java
public class GuiaNivel1 {
    public static void main(String[] args) {
        String nombre = "Ana García";
        int legajo = 10045;
        double promedio = 8.75;
        char turno = 'M';
        boolean activo = true;
        int anioIngreso = 2022;

        System.out.println("=== FICHA DE ESTUDIANTE ===");
        System.out.println("Nombre:    " + nombre);
        System.out.println("Legajo:    " + legajo);
        System.out.printf("Promedio:  %.2f%n", promedio);
        System.out.println("Turno:     " + turno);
        System.out.println("Activo:    " + activo);
        System.out.println("Ingresó:   " + anioIngreso);
    }
}
```

### Arrays 2D
```java
String[] estudiantes = {"Ana", "Luis", "Carlos", "Marta"};
String[] materias = {"Matemática", "Física", "Programación"};
double[][] notas = {
    {9.0, 8.5, 9.5},
    {7.0, 6.5, 8.0},
    {8.5, 7.5, 9.0},
    {6.0, 7.0, 6.5}
};

double[] promedios = new double[estudiantes.length];
for (int i = 0; i < estudiantes.length; i++) {
    double suma = 0;
    for (int j = 0; j < materias.length; j++) suma += notas[i][j];
    promedios[i] = suma / materias.length;
}
```

## Nivel 2

### Clase inmutable
```java
public final class Calificacion {
    private final int legajoEstudiante;
    private final String materia;
    private final double nota;

    public Calificacion(int legajoEstudiante, String materia, double nota) {
        if (nota < 0 || nota > 10) throw new IllegalArgumentException("Nota inválida");
        this.legajoEstudiante = legajoEstudiante;
        this.materia = materia;
        this.nota = nota;
    }

    public int getLegajoEstudiante() { return legajoEstudiante; }
    public String getMateria() { return materia; }
    public double getNota() { return nota; }
}
```

### Herencia e interfaces
```java
abstract class Persona {
    private final String nombre;
    protected Persona(String nombre) { this.nombre = nombre; }
    public String getNombre() { return nombre; }
}

interface Reportable {
    String generarReporte();
}
```

## Nivel 3

### Colecciones
```java
Map<Integer, Estudiante> porLegajo = new HashMap<>();
Set<String> materias = new HashSet<>();
List<Estudiante> estudiantes = new ArrayList<>();
```

### Streams
```java
List<String> nombres = estudiantes.stream()
    .filter(e -> e.getPromedio() >= 7)
    .map(Estudiante::getNombreCompleto)
    .sorted()
    .toList();
```

## Nivel 4

### Concurrencia
```java
ExecutorService pool = Executors.newFixedThreadPool(4);
pool.submit(() -> System.out.println("Procesando estudiante"));
pool.shutdown();
```

### IO/NIO
```java
try (Stream<String> lineas = Files.lines(Path.of("estudiantes.csv"))) {
    lineas.skip(1).forEach(System.out::println);
}
```

## Nivel 5

### Refactoring
```java
private static double promedio(double[] notas) {
    double suma = 0;
    for (double n : notas) suma += n;
    return suma / notas.length;
}
```

## Nivel 6

### Patrones
```java
Notificacion n = NotificacionFactory.crear("EMAIL");
n.enviar("ana@mail.com", "Examen mañana");
```

## Nivel 7

### Arquitectura
```java
EstudianteRepositorio repo = new EstudianteRepositorioEnMemoria();
EstudianteServicio servicio = new EstudianteServicioImpl(repo);
EstudianteControlador controlador = new EstudianteControlador(servicio);
```

## Nivel 8

### Testing
```java
@Test
void debeAprobarConPromedioAlto() {
    Estudiante e = new Estudiante("Ana", "García", "1", "a@b.com", 1001, "Sistemas");
    e.setPromedio(8.5);
    assertTrue(e.estaAprobado());
}
```

## Nivel 9

### Spring Boot
```java
@RestController
public class HolaController {
    @GetMapping("/hola")
    public String saludar() {
        return "Hola desde Spring Boot";
    }
}
```

### Base de datos
```java
public interface EstudianteRepository extends JpaRepository<Estudiante, Long> {
    Optional<Estudiante> findByEmail(String email);
}
```

## Recomendación

Si un ejercicio te resulta difícil, intenta resolver solo una parte:
- primero la estructura,
- después la lógica,
- y por último la presentación o el reporte.
