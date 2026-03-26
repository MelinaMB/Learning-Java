# 3.4 Streams y Lambdas

Java 8 introdujo el paradigma funcional con lambdas y la Stream API,
que permiten escribir código más conciso, legible y expresivo.

## Expresiones Lambda

Una lambda es una **función anónima** — una función sin nombre que puede pasarse como argumento.

```java
// Sintaxis: (parámetros) -> { cuerpo }
// Si el cuerpo es una sola expresión, las llaves y return son opcionales

// Comparator con clase anónima (estilo antiguo)
Comparator<String> comp1 = new Comparator<String>() {
    @Override
    public int compare(String a, String b) {
        return a.compareTo(b);
    }
};

// Comparator con lambda (estilo moderno)
Comparator<String> comp2 = (a, b) -> a.compareTo(b);

// Runnable con lambda
Runnable tarea = () -> System.out.println("Tarea ejecutada");
tarea.run();

// Con un parámetro (los paréntesis son opcionales)
java.util.function.Consumer<String> imprimir = nombre -> System.out.println("Hola, " + nombre);
imprimir.accept("Ana");

// Con cuerpo de múltiples líneas
java.util.function.Function<Double, String> clasificar = nota -> {
    if (nota >= 9) return "Sobresaliente";
    if (nota >= 6) return "Aprobado";
    return "Reprobado";
};
```

---

## Interfaces Funcionales

Una interfaz funcional tiene exactamente un método abstracto.
El paquete `java.util.function` tiene las más comunes.

| Interfaz | Método | Descripción | Ejemplo |
|---------|--------|-------------|---------|
| `Predicate<T>` | `boolean test(T t)` | Condición | `nota -> nota >= 6` |
| `Function<T,R>` | `R apply(T t)` | Transformación | `e -> e.getNombre()` |
| `Consumer<T>` | `void accept(T t)` | Consumir sin retorno | `e -> System.out.println(e)` |
| `Supplier<T>` | `T get()` | Proveer sin entrada | `() -> new Estudiante(...)` |
| `BiFunction<T,U,R>` | `R apply(T t, U u)` | Dos entradas, un resultado | `(a, b) -> a + b` |
| `UnaryOperator<T>` | `T apply(T t)` | Entrada y salida del mismo tipo | `nota -> nota + 0.5` |
| `BinaryOperator<T>` | `T apply(T t1, T t2)` | Dos entradas del mismo tipo | `(a, b) -> a + b` |

```java
import java.util.function.*;

Predicate<Estudiante> estaAprobado = e -> e.getPromedio() >= 6.0;
Predicate<Estudiante> estaActivo = e -> e.isActivo();
Predicate<Estudiante> aprobadoYActivo = estaAprobado.and(estaActivo);
Predicate<Estudiante> reprobado = estaAprobado.negate();

Function<Estudiante, String> aNombre = Estudiante::getNombreCompleto;
Function<Estudiante, Double> aPromedio = Estudiante::getPromedio;
Function<Estudiante, String> aReporte = aNombre.andThen(n -> n.toUpperCase());

Consumer<Estudiante> mostrar = e -> System.out.println(e.getNombreCompleto());
Consumer<Estudiante> mostrarConDetalle = mostrar.andThen(
    e -> System.out.printf("  Promedio: %.2f%n", e.getPromedio()));

Supplier<List<Estudiante>> listaVacia = ArrayList::new;
```

---

## Referencias a Métodos

Una forma más concisa de referenciar métodos existentes como lambdas.

```java
// Referencia a método estático: Clase::metodo
Function<String, Integer> parsear = Integer::parseInt;

// Referencia a método de instancia de un objeto específico
Estudiante ana = new Estudiante("Ana", "García", "1", "a@b.com", 1001, "Sistemas");
Supplier<String> nombreAna = ana::getNombreCompleto;

// Referencia a método de instancia de tipo arbitrario: Clase::metodo
Function<Estudiante, String> obtenerNombre = Estudiante::getNombreCompleto;
Function<String, String> toUpper = String::toUpperCase;

// Referencia a constructor: Clase::new
// BiFunction<String, Integer, StringBuilder> crear = StringBuilder::new;
```

---

## Stream API

Un Stream es una secuencia de elementos que soporta operaciones de procesamiento
en cadena. Los streams son:
- **Perezosos** — las operaciones intermedias no se ejecutan hasta que hay una terminal
- **No reutilizables** — un stream solo se puede consumir una vez
- **No modifican la fuente** — crean nuevas colecciones/valores

```
Fuente --> [Operaciones Intermedias] --> Operación Terminal
(Collection, Array)  (filter, map, sorted)  (collect, forEach, reduce)
```

### Crear Streams

```java
import java.util.stream.*;

// De una colección
List<Estudiante> estudiantes = obtenerEstudiantes();
Stream<Estudiante> stream1 = estudiantes.stream();

// De un array
String[] nombres = {"Ana", "Luis", "Carlos"};
Stream<String> stream2 = Arrays.stream(nombres);

// Directamente
Stream<String> stream3 = Stream.of("Ana", "Luis", "Carlos");

// Stream vacío
Stream<Object> vacio = Stream.empty();

// Stream infinito
Stream<Integer> numeros = Stream.iterate(1, n -> n + 1); // 1, 2, 3, ...
Stream<Double> aleatorios = Stream.generate(Math::random);
```

### Operaciones Intermedias (retornan Stream)

```java
List<Estudiante> estudiantes = List.of(
    new Estudiante("Ana", "García", "1", "a@b.com", 1001, "Sistemas"),
    new Estudiante("Luis", "Pérez", "2", "l@b.com", 1002, "Redes"),
    new Estudiante("Carlos", "Ruiz", "3", "c@b.com", 1003, "Sistemas"),
    new Estudiante("Marta", "López", "4", "m@b.com", 1004, "Sistemas"),
    new Estudiante("Pedro", "Soto", "5", "p@b.com", 1005, "Redes")
);

// filter — filtra elementos que cumplen una condición
List<Estudiante> deSistemas = estudiantes.stream()
    .filter(e -> e.getCarrera().equals("Sistemas"))
    .collect(Collectors.toList());

// map — transforma cada elemento
List<String> nombres = estudiantes.stream()
    .map(Estudiante::getNombreCompleto)
    .collect(Collectors.toList());

// mapToDouble — para primitivos
double[] promedios = estudiantes.stream()
    .mapToDouble(Estudiante::getPromedio)
    .toArray();

// sorted — ordenar
List<Estudiante> ordenados = estudiantes.stream()
    .sorted(Comparator.comparing(Estudiante::getApellido))
    .collect(Collectors.toList());

// sorted con múltiples criterios
List<Estudiante> ordenadosMult = estudiantes.stream()
    .sorted(Comparator.comparing(Estudiante::getCarrera)
                      .thenComparing(Estudiante::getApellido))
    .collect(Collectors.toList());

// distinct — elimina duplicados
List<String> carrerasUnicas = estudiantes.stream()
    .map(Estudiante::getCarrera)
    .distinct()
    .collect(Collectors.toList());

// limit y skip — paginación
List<Estudiante> pagina1 = estudiantes.stream()
    .skip(0).limit(3)
    .collect(Collectors.toList());

List<Estudiante> pagina2 = estudiantes.stream()
    .skip(3).limit(3)
    .collect(Collectors.toList());

// peek — para debugging sin alterar el stream
List<Estudiante> resultado = estudiantes.stream()
    .filter(e -> e.getPromedio() >= 6.0)
    .peek(e -> System.out.println("Procesando: " + e.getNombreCompleto()))
    .collect(Collectors.toList());
```

### Operaciones Terminales

```java
// forEach — ejecuta acción sobre cada elemento
estudiantes.stream()
    .forEach(e -> System.out.println(e.getNombreCompleto()));

// count — cuenta elementos
long cantidadAprobados = estudiantes.stream()
    .filter(e -> e.getPromedio() >= 6.0)
    .count();

// findFirst / findAny
Optional<Estudiante> primeroDeSistemas = estudiantes.stream()
    .filter(e -> e.getCarrera().equals("Sistemas"))
    .findFirst();

// anyMatch / allMatch / noneMatch
boolean hayAprobados = estudiantes.stream()
    .anyMatch(e -> e.getPromedio() >= 6.0);

boolean todosTienenEmail = estudiantes.stream()
    .allMatch(e -> e.getEmail() != null && !e.getEmail().isBlank());

// min / max
Optional<Estudiante> mejorEstudiante = estudiantes.stream()
    .max(Comparator.comparingDouble(Estudiante::getPromedio));

// reduce — combina elementos en un resultado
double sumaPromedios = estudiantes.stream()
    .mapToDouble(Estudiante::getPromedio)
    .reduce(0, Double::sum);

// Estadísticas con primitivos
DoubleSummaryStatistics stats = estudiantes.stream()
    .mapToDouble(Estudiante::getPromedio)
    .summaryStatistics();

System.out.println("Promedio del curso: " + stats.getAverage());
System.out.println("Nota máxima: " + stats.getMax());
System.out.println("Nota mínima: " + stats.getMin());
System.out.println("Total estudiantes: " + stats.getCount());
```

### Collectors avanzados

```java
import java.util.stream.Collectors;

// toList, toSet, toMap
List<String> nombresList = estudiantes.stream()
    .map(Estudiante::getNombreCompleto)
    .collect(Collectors.toList());

Map<Integer, Estudiante> porLegajo = estudiantes.stream()
    .collect(Collectors.toMap(Estudiante::getLegajo, e -> e));

// groupingBy — agrupar por criterio
Map<String, List<Estudiante>> porCarrera = estudiantes.stream()
    .collect(Collectors.groupingBy(Estudiante::getCarrera));

// groupingBy con downstream
Map<String, Long> cantidadPorCarrera = estudiantes.stream()
    .collect(Collectors.groupingBy(Estudiante::getCarrera, Collectors.counting()));

Map<String, Double> promedioPorCarrera = estudiantes.stream()
    .collect(Collectors.groupingBy(
        Estudiante::getCarrera,
        Collectors.averagingDouble(Estudiante::getPromedio)
    ));

// partitioningBy — divide en dos grupos (true/false)
Map<Boolean, List<Estudiante>> aprobadosYReprobados = estudiantes.stream()
    .collect(Collectors.partitioningBy(e -> e.getPromedio() >= 6.0));

List<Estudiante> aprobados = aprobadosYReprobados.get(true);
List<Estudiante> reprobados = aprobadosYReprobados.get(false);

// joining — concatenar strings
String nombresJuntos = estudiantes.stream()
    .map(Estudiante::getNombreCompleto)
    .collect(Collectors.joining(", "));
System.out.println(nombresJuntos); // Ana García, Luis Pérez, ...

String nombresFormateados = estudiantes.stream()
    .map(Estudiante::getNombreCompleto)
    .collect(Collectors.joining(", ", "[", "]"));
System.out.println(nombresFormateados); // [Ana García, Luis Pérez, ...]
```

---

## Ejemplo completo — Reporte de curso

```java
public class ReporteCurso {
    public static void main(String[] args) {
        List<Estudiante> estudiantes = obtenerEstudiantes();

        System.out.println("=== REPORTE DEL CURSO ===\n");

        // Estadísticas generales
        DoubleSummaryStatistics stats = estudiantes.stream()
            .mapToDouble(Estudiante::getPromedio)
            .summaryStatistics();

        System.out.printf("Total estudiantes: %d%n", (long) stats.getCount());
        System.out.printf("Promedio del curso: %.2f%n", stats.getAverage());
        System.out.printf("Nota más alta: %.1f%n", stats.getMax());
        System.out.printf("Nota más baja: %.1f%n", stats.getMin());

        // Aprobados y reprobados
        Map<Boolean, List<Estudiante>> particion = estudiantes.stream()
            .collect(Collectors.partitioningBy(e -> e.getPromedio() >= 6.0));
        System.out.printf("Aprobados: %d | Reprobados: %d%n\n",
            particion.get(true).size(), particion.get(false).size());

        // Top 3
        System.out.println("--- TOP 3 ESTUDIANTES ---");
        estudiantes.stream()
            .sorted(Comparator.comparingDouble(Estudiante::getPromedio).reversed())
            .limit(3)
            .forEach(e -> System.out.printf("  %s — %.2f%n",
                e.getNombreCompleto(), e.getPromedio()));

        // Por carrera
        System.out.println("\n--- POR CARRERA ---");
        estudiantes.stream()
            .collect(Collectors.groupingBy(
                Estudiante::getCarrera,
                Collectors.averagingDouble(Estudiante::getPromedio)
            ))
            .entrySet().stream()
            .sorted(Map.Entry.<String, Double>comparingByValue().reversed())
            .forEach(entry ->
                System.out.printf("  %-15s promedio: %.2f%n",
                    entry.getKey(), entry.getValue()));
    }
}
```

---

## Resumen

| Concepto | Descripción |
|----------|-------------|
| Lambda | Función anónima `(params) -> cuerpo` |
| Referencia a método | `Clase::metodo` — lambda más concisa |
| `Predicate<T>` | Función que retorna boolean |
| `Function<T,R>` | Función que transforma T en R |
| `stream()` | Crea un stream desde una colección |
| `filter()` | Filtra elementos por condición |
| `map()` | Transforma cada elemento |
| `collect()` | Terminal: recolecta en una colección |
| `Collectors.groupingBy()` | Agrupa por criterio |

**Siguiente:** [3.5 Optional y Fechas](05-optional-fechas.md)
