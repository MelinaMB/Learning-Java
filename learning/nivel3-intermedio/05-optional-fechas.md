# 3.5 Optional y Fechas

## Optional

`Optional<T>` es un contenedor que puede tener un valor o estar vacío.
Elimina la necesidad de verificar `null` manualmente y hace explícito que un
valor puede estar ausente.

```java
import java.util.Optional;

// Crear Optional
Optional<String> conValor = Optional.of("Ana García");
Optional<String> vacio = Optional.empty();
Optional<String> posibleNull = Optional.ofNullable(null); // no lanza NPE

// Verificar y obtener
System.out.println(conValor.isPresent());  // true
System.out.println(vacio.isPresent());     // false
System.out.println(vacio.isEmpty());       // true (Java 11+)

// Obtener el valor — lanza NoSuchElementException si está vacío
String valor = conValor.get();             // "Ana García"

// Formas seguras de obtener
String nombre = conValor.orElse("Sin nombre");      // valor por defecto
String nombre2 = vacio.orElse("Sin nombre");        // "Sin nombre"
String nombre3 = vacio.orElseGet(() -> generarDefault()); // proveedor lazy
String nombre4 = conValor.orElseThrow(
    () -> new IllegalStateException("Nombre requerido")); // excepción custom
```

### Operaciones funcionales sobre Optional

```java
Optional<Estudiante> estudiante = repositorio.buscarPorLegajo(1001);

// map — transforma si hay valor
Optional<String> nombre = estudiante.map(Estudiante::getNombreCompleto);

// filter — filtra si hay valor
Optional<Estudiante> aprobado = estudiante.filter(e -> e.getPromedio() >= 6.0);

// ifPresent — ejecuta si hay valor
estudiante.ifPresent(e -> System.out.println("Encontrado: " + e.getNombreCompleto()));

// ifPresentOrElse (Java 9+)
estudiante.ifPresentOrElse(
    e -> System.out.println("Encontrado: " + e.getNombreCompleto()),
    () -> System.out.println("Estudiante no encontrado")
);

// flatMap — para Optional dentro de Optional
Optional<String> email = estudiante
    .flatMap(e -> Optional.ofNullable(e.getEmail()))
    .filter(em -> !em.isBlank());

// Encadenamiento
String resultado = repositorio.buscarPorLegajo(1001)
    .filter(Estudiante::isActivo)
    .map(Estudiante::getNombreCompleto)
    .map(String::toUpperCase)
    .orElse("Estudiante no encontrado o inactivo");
```

### Optional en servicios

```java
public class EstudianteServicio {
    private final Map<Integer, Estudiante> repositorio = new HashMap<>();

    // Retorna Optional en lugar de null
    public Optional<Estudiante> buscarPorLegajo(int legajo) {
        return Optional.ofNullable(repositorio.get(legajo));
    }

    public Optional<Estudiante> buscarPorEmail(String email) {
        return repositorio.values().stream()
            .filter(e -> email.equals(e.getEmail()))
            .findFirst();
    }

    // Uso del servicio
    public void procesarEstudiante(int legajo) {
        buscarPorLegajo(legajo)
            .filter(Estudiante::isActivo)
            .map(e -> {
                System.out.println("Procesando: " + e.getNombreCompleto());
                return e;
            })
            .orElseThrow(() -> new EstudianteNoEncontradoException(
                "No se encontró estudiante activo con legajo: " + legajo, legajo));
    }
}
```

> **Cuándo NO usar Optional:**
> - En parámetros de métodos (usa sobrecarga o valores por defecto)
> - En atributos de clases (usa null o valores centinela)
> - En colecciones (usa colecciones vacías en su lugar)
> **Cuándo SÍ:**
> - Como tipo de retorno de métodos que pueden no encontrar un resultado

---

## API de Fechas y Horas (Java 8+)

Java 8 introdujo `java.time` como reemplazo moderno de las antiguas
`Date` y `Calendar` (que eran confusas y mutables).

### Clases principales

| Clase | Descripción | Ejemplo |
|-------|-------------|---------|
| `LocalDate` | Solo fecha (sin hora, sin zona) | Fecha de nacimiento |
| `LocalTime` | Solo hora | Hora de entrada |
| `LocalDateTime` | Fecha + hora (sin zona) | Fecha de inscripción |
| `ZonedDateTime` | Fecha + hora + zona horaria | Eventos internacionales |
| `Instant` | Timestamp (instante en el tiempo) | Auditoría |
| `Duration` | Diferencia en horas/minutos/segundos | Duración de un examen |
| `Period` | Diferencia en años/meses/días | Años de antigüedad |

### LocalDate

```java
import java.time.LocalDate;
import java.time.Month;
import java.time.DayOfWeek;

// Crear fechas
LocalDate hoy = LocalDate.now();
LocalDate fechaEspecifica = LocalDate.of(2024, 3, 15);
LocalDate desdeCadena = LocalDate.parse("2024-03-15"); // ISO 8601 por defecto

System.out.println(hoy);               // 2024-03-15 (o la fecha actual)
System.out.println(fechaEspecifica);   // 2024-03-15

// Componentes
System.out.println(fechaEspecifica.getYear());        // 2024
System.out.println(fechaEspecifica.getMonth());       // MARCH
System.out.println(fechaEspecifica.getMonthValue());  // 3
System.out.println(fechaEspecifica.getDayOfMonth());  // 15
System.out.println(fechaEspecifica.getDayOfWeek());   // FRIDAY

// Operaciones (inmutables — devuelven nueva fecha)
LocalDate mañana = hoy.plusDays(1);
LocalDate semanaProxima = hoy.plusWeeks(1);
LocalDate mesProximo = hoy.plusMonths(1);
LocalDate añoProximo = hoy.plusYears(1);
LocalDate ayer = hoy.minusDays(1);

// Comparar
System.out.println(fechaEspecifica.isBefore(hoy));  // depende de cuándo ejecutas
System.out.println(fechaEspecifica.isAfter(hoy));
System.out.println(fechaEspecifica.isEqual(hoy));

// Calcular período entre fechas
LocalDate nacimiento = LocalDate.of(2004, 5, 20);
LocalDate ahora = LocalDate.of(2024, 3, 15);
java.time.Period edad = java.time.Period.between(nacimiento, ahora);
System.out.printf("Edad: %d años, %d meses, %d días%n",
    edad.getYears(), edad.getMonths(), edad.getDays());
```

### LocalDateTime

```java
import java.time.LocalDateTime;

LocalDateTime ahora = LocalDateTime.now();
LocalDateTime examen = LocalDateTime.of(2024, 7, 10, 9, 0, 0);

System.out.println(ahora);   // 2024-03-15T14:30:45.123
System.out.println(examen);  // 2024-07-10T09:00

// Extraer partes
System.out.println(examen.toLocalDate()); // 2024-07-10
System.out.println(examen.toLocalTime()); // 09:00

// Calcular diferencia en horas
java.time.Duration diferencia = java.time.Duration.between(ahora, examen);
System.out.println("Horas para el examen: " + diferencia.toHours());
System.out.println("Días para el examen: " + diferencia.toDays());
```

### DateTimeFormatter

```java
import java.time.format.DateTimeFormatter;
import java.util.Locale;

LocalDate fecha = LocalDate.of(2024, 3, 15);

// Formatos predefinidos
System.out.println(fecha.format(DateTimeFormatter.ISO_DATE));         // 2024-03-15
System.out.println(fecha.format(DateTimeFormatter.BASIC_ISO_DATE));   // 20240315

// Formato personalizado
DateTimeFormatter formato = DateTimeFormatter.ofPattern("dd/MM/yyyy");
System.out.println(fecha.format(formato));  // 15/03/2024

DateTimeFormatter formatoLargo = DateTimeFormatter.ofPattern(
    "EEEE, d 'de' MMMM 'de' yyyy", new Locale("es", "AR"));
System.out.println(fecha.format(formatoLargo)); // viernes, 15 de marzo de 2024

// Parsear String a fecha
String cadena = "15/03/2024";
LocalDate fechaParseada = LocalDate.parse(cadena, formato);
System.out.println(fechaParseada); // 2024-03-15

// Para LocalDateTime
DateTimeFormatter formatoCompleto = DateTimeFormatter.ofPattern("dd/MM/yyyy HH:mm");
LocalDateTime inscripcion = LocalDateTime.parse("15/03/2024 14:30", formatoCompleto);
System.out.println(inscripcion.format(formatoCompleto)); // 15/03/2024 14:30
```

### Aplicación en el sistema escolar

```java
public class Inscripcion {
    private final int legajoEstudiante;
    private final String codigoCurso;
    private final LocalDateTime fechaInscripcion;
    private LocalDate fechaInicioClases;
    private LocalDate fechaFinCurso;

    public Inscripcion(int legajoEstudiante, String codigoCurso,
                       LocalDate fechaInicio, LocalDate fechaFin) {
        this.legajoEstudiante = legajoEstudiante;
        this.codigoCurso = codigoCurso;
        this.fechaInscripcion = LocalDateTime.now();
        this.fechaInicioClases = fechaInicio;
        this.fechaFinCurso = fechaFin;
    }

    public boolean estaVigente() {
        LocalDate hoy = LocalDate.now();
        return !hoy.isBefore(fechaInicioClases) && !hoy.isAfter(fechaFinCurso);
    }

    public long getDiasRestantes() {
        return java.time.temporal.ChronoUnit.DAYS.between(LocalDate.now(), fechaFinCurso);
    }

    public String getFechaInscripcionFormateada() {
        return fechaInscripcion.format(
            DateTimeFormatter.ofPattern("dd/MM/yyyy 'a las' HH:mm"));
    }
}
```

---

## Resumen

| Concepto | Descripción |
|----------|-------------|
| `Optional.of(v)` | Crea Optional con valor no nulo |
| `Optional.ofNullable(v)` | Crea Optional que acepta null |
| `orElse(default)` | Valor si Optional está vacío |
| `map()` | Transforma el valor si existe |
| `filter()` | Filtra el valor si existe |
| `LocalDate` | Fecha sin hora (inmutable) |
| `LocalDateTime` | Fecha y hora (inmutable) |
| `DateTimeFormatter` | Formatear/parsear fechas |
| `Period` | Diferencia en años/meses/días |
| `Duration` | Diferencia en horas/minutos/segundos |

**Siguiente:** [Ejercicios del Nivel 3](ejercicios.md)
