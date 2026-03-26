# 1.1 Introducción a Java

## ¿Qué es Java?

Java es un lenguaje de programación **orientado a objetos**, **fuertemente tipado** y
**compilado/interpretado** creado por Sun Microsystems en 1995 (hoy propiedad de Oracle).
Su lema histórico es **"Write Once, Run Anywhere"** (WORA): el código compilado en una
máquina puede ejecutarse en cualquier sistema que tenga una JVM instalada.

### ¿Por qué aprender Java?

- Uno de los lenguajes más demandados en el mercado laboral
- Base del ecosistema Android (hasta hace poco)
- Ecosistema enorme: Spring, Hibernate, Maven, etc.
- Fuertemente tipado: te enseña disciplina de programación
- Millones de recursos de aprendizaje disponibles

---

## JDK, JRE y JVM — Las siglas explicadas

```
+--------------------------------------------+
|                   JDK                      |  <-- Java Development Kit
|  (todo lo necesario para DESARROLLAR)      |
|                                            |
|   +------------------------------------+   |
|   |              JRE                  |   |  <-- Java Runtime Environment
|   |  (todo lo necesario para EJECUTAR)|   |
|   |                                   |   |
|   |   +---------------------------+   |   |
|   |   |          JVM              |   |   |  <-- Java Virtual Machine
|   |   |  (ejecuta el bytecode)    |   |   |
|   |   +---------------------------+   |   |
|   +------------------------------------+   |
+--------------------------------------------+
```

| Componente | Descripción | ¿Quién lo necesita? |
|------------|-------------|---------------------|
| **JVM** | Máquina virtual que ejecuta bytecode `.class` | Todos |
| **JRE** | JVM + librerías estándar de Java | Usuarios finales |
| **JDK** | JRE + compilador (`javac`) + herramientas de desarrollo | Desarrolladores |

> **Consejo:** Siempre instala el **JDK**, no solo el JRE. Sin el compilador `javac`
> no puedes compilar tu código fuente.

---

## El proceso de compilación y ejecución

```
MiClase.java  --[javac]--->  MiClase.class  --[JVM]--->  Resultado
(código fuente)   (compilador)  (bytecode)    (ejecuta)
```

1. Escribes código fuente en un archivo `.java`
2. El compilador `javac` lo transforma en **bytecode** (archivo `.class`)
3. La JVM carga y ejecuta el bytecode en cualquier plataforma

---

## Tu primer programa: Hola Mundo

Crea un archivo llamado `HolaMundo.java`:

```java
public class HolaMundo {
    public static void main(String[] args) {
        System.out.println("¡Hola, Mundo!");
    }
}
```

Para compilar y ejecutar desde la terminal:

```bash
# Compilar
javac HolaMundo.java

# Ejecutar
java HolaMundo

# Resultado:
# ¡Hola, Mundo!
```

### Anatomía del programa

```java
public class HolaMundo {          // Declaración de la clase
    public static void main(String[] args) {  // Método principal (punto de entrada)
        System.out.println("¡Hola, Mundo!");  // Imprime en consola
    }
}
```

| Parte | Significado |
|-------|-------------|
| `public` | Modificador de acceso: visible desde cualquier lugar |
| `class` | Palabra clave que define una clase |
| `HolaMundo` | Nombre de la clase (debe coincidir con el nombre del archivo) |
| `static` | Pertenece a la clase, no a una instancia |
| `void` | El método no retorna ningún valor |
| `main` | Nombre del método de entrada obligatorio |
| `String[] args` | Parámetro: arreglo de argumentos de línea de comandos |
| `System.out.println` | Imprime una línea en la salida estándar |

---

## Primer programa relacionado con el proyecto escolar

```java
public class SistemaEscolar {
    public static void main(String[] args) {
        System.out.println("=== Sistema de Gestión Escolar ===");
        System.out.println("Versión: 1.0");
        System.out.println("Bienvenido al sistema");

        // Mostrar información de un estudiante (por ahora con datos fijos)
        String nombreEstudiante = "Ana García";
        int edadEstudiante = 20;
        String curso = "Programación I";

        System.out.println("\n--- Estudiante registrado ---");
        System.out.println("Nombre: " + nombreEstudiante);
        System.out.println("Edad: " + edadEstudiante + " años");
        System.out.println("Curso: " + curso);
    }
}
```

---

## Estructura de un proyecto Java típico

```
mi-proyecto/
├── src/
│   └── main/
│       └── java/
│           └── com/
│               └── escuela/
│                   ├── Main.java
│                   ├── modelo/
│                   │   └── Estudiante.java
│                   └── servicio/
│                       └── EstudianteServicio.java
├── pom.xml          (configuración Maven)
└── README.md
```

---

## `package` e `import`

Los paquetes sirven para organizar clases relacionadas y evitar nombres repetidos.
El `import` te permite usar clases de otros paquetes sin escribir el nombre completo.

```java
package com.escuela.modelo;

import java.util.ArrayList;
import java.util.List;

public class Curso {
    private List<String> estudiantes = new ArrayList<>();
}
```

> **Idea clave:** primero organiza el código con `package`, después importa lo que necesitas con `import`.

---

## Convenciones de nomenclatura desde el principio

| Elemento | Convención | Ejemplo |
|----------|-----------|---------|
| Clases | PascalCase | `Estudiante`, `CursoMateria` |
| Métodos | camelCase | `obtenerNombre()`, `calcularPromedio()` |
| Variables | camelCase | `nombreEstudiante`, `totalCursos` |
| Constantes | SCREAMING_SNAKE_CASE | `MAX_ESTUDIANTES`, `NOTA_APROBACION` |
| Paquetes | todo minúsculas | `com.escuela.modelo` |

> **Regla de oro:** El nombre del archivo `.java` debe ser exactamente igual
> al nombre de la clase pública que contiene, incluyendo mayúsculas y minúsculas.
> `Estudiante.java` contiene `public class Estudiante`.

---

## Comentarios en Java

```java
// Comentario de una sola línea

/*
 * Comentario
 * de múltiples
 * líneas
 */

/**
 * Comentario Javadoc — documentación oficial de la clase o método.
 * Herramientas como IntelliJ los usan para mostrar ayuda contextual.
 *
 * @param nombre El nombre del estudiante
 * @return El saludo personalizado
 */
public String saludar(String nombre) {
    return "Hola, " + nombre + "!";
}
```

---

## Resumen

- Java compila a bytecode que la JVM ejecuta en cualquier plataforma.
- Necesitas el JDK para desarrollar.
- Todo programa Java comienza en el método `public static void main(String[] args)`.
- El nombre del archivo debe coincidir con el nombre de la clase pública.

**Siguiente:** [1.2 Variables y Tipos de Datos](02-variables-tipos.md)
