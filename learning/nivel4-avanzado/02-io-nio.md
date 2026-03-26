# 4.2 IO y NIO

## IO clásico — java.io

### Leer y escribir archivos de texto

```java
import java.io.*;
import java.nio.charset.StandardCharsets;

// Escribir archivo
public static void escribirEstudiantes(List<Estudiante> estudiantes, String ruta)
        throws IOException {
    try (BufferedWriter writer = new BufferedWriter(
            new OutputStreamWriter(new FileOutputStream(ruta), StandardCharsets.UTF_8))) {
        writer.write("legajo,nombre,apellido,carrera,promedio");
        writer.newLine();
        for (Estudiante e : estudiantes) {
            writer.write(String.format("%d,%s,%s,%s,%.2f",
                e.getLegajo(), e.getNombre(), e.getApellido(),
                e.getCarrera(), e.getPromedio()));
            writer.newLine();
        }
    }
}

// Leer archivo
public static List<String[]> leerCSV(String ruta) throws IOException {
    List<String[]> datos = new ArrayList<>();
    try (BufferedReader reader = new BufferedReader(
            new InputStreamReader(new FileInputStream(ruta), StandardCharsets.UTF_8))) {
        String linea;
        boolean primeraLinea = true;
        while ((linea = reader.readLine()) != null) {
            if (primeraLinea) { primeraLinea = false; continue; } // saltar encabezado
            datos.add(linea.split(","));
        }
    }
    return datos;
}
```

---

## NIO.2 — java.nio.file (Java 7+)

NIO.2 es la API moderna y recomendada para trabajar con el sistema de archivos.

```java
import java.nio.file.*;
import java.nio.charset.StandardCharsets;
import java.io.IOException;

// Path — referencia a un archivo o directorio
Path archivo = Path.of("datos", "estudiantes.csv");
Path ruta = Paths.get("/home/melina/escuela/datos/estudiantes.csv");

// Información del path
System.out.println(archivo.getFileName());   // estudiantes.csv
System.out.println(archivo.getParent());     // datos
System.out.println(archivo.toAbsolutePath()); // ruta absoluta

// Operaciones con Files
boolean existe = Files.exists(archivo);
boolean esDirectorio = Files.isDirectory(archivo);
long tamaño = Files.size(archivo);

// Crear directorios
Files.createDirectories(Path.of("datos", "reportes", "2024"));

// Leer todas las líneas
List<String> lineas = Files.readAllLines(archivo, StandardCharsets.UTF_8);
for (String linea : lineas) {
    System.out.println(linea);
}

// Leer como String (archivos pequeños)
String contenido = Files.readString(archivo, StandardCharsets.UTF_8); // Java 11+

// Escribir
List<String> lineasAEscribir = List.of("Ana,García,1001", "Luis,Pérez,1002");
Files.write(archivo, lineasAEscribir, StandardCharsets.UTF_8,
    StandardOpenOption.CREATE, StandardOpenOption.TRUNCATE_EXISTING);

// Escribir String
Files.writeString(archivo, "Contenido del archivo", StandardCharsets.UTF_8); // Java 11+

// Copiar, mover, eliminar
Path destino = Path.of("backup", "estudiantes_backup.csv");
Files.copy(archivo, destino, StandardCopyOption.REPLACE_EXISTING);
Files.move(archivo, destino, StandardCopyOption.REPLACE_EXISTING);
Files.delete(archivo);
Files.deleteIfExists(archivo); // no lanza excepción si no existe
```

### Listar y buscar archivos

```java
// Listar contenido de un directorio
try (var stream = Files.list(Path.of("datos"))) {
    stream.filter(p -> p.toString().endsWith(".csv"))
          .forEach(System.out::println);
}

// Búsqueda recursiva
try (var walk = Files.walk(Path.of("datos"), 3)) { // máx 3 niveles de profundidad
    walk.filter(Files::isRegularFile)
        .filter(p -> p.toString().endsWith(".csv"))
        .forEach(p -> System.out.println("Encontrado: " + p));
}

// Encontrar archivos que coincidan con patrón
try (var finder = Files.find(
        Path.of("datos"),
        5,
        (path, attrs) -> attrs.isRegularFile() && path.getFileName().toString().startsWith("reporte"))) {
    finder.forEach(System.out::println);
}
```

### Leer/escribir con Stream (archivos grandes)

```java
// Leer línea por línea como Stream (eficiente en memoria)
try (Stream<String> lineas = Files.lines(Path.of("estudiantes.csv"), StandardCharsets.UTF_8)) {
    List<Estudiante> estudiantes = lineas
        .skip(1) // saltar encabezado
        .map(linea -> linea.split(","))
        .filter(partes -> partes.length == 5)
        .map(partes -> new Estudiante(
            partes[1].trim(),              // nombre
            partes[2].trim(),              // apellido
            "dni-" + partes[0].trim(),     // dni
            partes[1].toLowerCase() + "@escuela.edu",
            Integer.parseInt(partes[0].trim()), // legajo
            partes[3].trim()               // carrera
        ))
        .collect(Collectors.toList());

    System.out.println("Estudiantes cargados: " + estudiantes.size());
}
```

---

## Serialización

La serialización convierte un objeto a bytes para guardarlo o enviarlo por red.

```java
import java.io.*;

public class Estudiante implements Serializable {
    private static final long serialVersionUID = 1L; // importante para versioning

    private int legajo;
    private String nombre;
    private transient String password; // transient = no se serializa

    // ... constructor, getters ...
}

// Serializar (objeto -> archivo)
Estudiante e = new Estudiante("Ana", "García", "123", "ana@mail.com", 1001, "Sistemas");
try (ObjectOutputStream oos = new ObjectOutputStream(
        new FileOutputStream("estudiante.ser"))) {
    oos.writeObject(e);
    System.out.println("Serializado correctamente.");
}

// Deserializar (archivo -> objeto)
try (ObjectInputStream ois = new ObjectInputStream(
        new FileInputStream("estudiante.ser"))) {
    Estudiante recuperado = (Estudiante) ois.readObject();
    System.out.println("Recuperado: " + recuperado.getNombreCompleto());
}
```

---

## Resumen

| Concepto | API | Descripción |
|----------|-----|-------------|
| Leer texto | `BufferedReader` | Leer línea a línea |
| Escribir texto | `BufferedWriter` | Escribir línea a línea |
| Ruta de archivo | `Path.of(...)` | Referencia a archivo/directorio |
| Operaciones | `Files.*` | Leer, escribir, copiar, mover, listar |
| Stream de líneas | `Files.lines()` | Procesar archivos grandes con Streams |
| Serialización | `ObjectOutputStream` | Convertir objeto a bytes |

**Siguiente:** [4.3 Anotaciones y Reflexión](03-anotaciones-reflection.md)
