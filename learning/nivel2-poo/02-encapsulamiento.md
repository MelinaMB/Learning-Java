# 2.2 Encapsulamiento

El encapsulamiento consiste en **ocultar los detalles internos** de un objeto
y exponer solo lo que es necesario a través de una interfaz controlada.
Es como una cápsula: lo que está dentro está protegido del exterior.

## Modificadores de Acceso

| Modificador | Misma clase | Mismo paquete | Subclase | Cualquier clase |
|-------------|:-----------:|:-------------:|:--------:|:---------------:|
| `private`   | Sí          | No            | No       | No              |
| *(sin mod)* (default) | Sí | Sí         | No       | No              |
| `protected` | Sí          | Sí            | Sí       | No              |
| `public`    | Sí          | Sí            | Sí       | Sí              |

> **Regla de oro:** Haz los atributos `private` por defecto y expón solo
> lo necesario con métodos `public`.

---

## Getters y Setters

```java
public class Estudiante {

    // Atributos privados — no accesibles desde fuera
    private int legajo;
    private String nombre;
    private String apellido;
    private double promedio;
    private boolean activo;

    public Estudiante(String nombre, String apellido, int legajo) {
        this.nombre = nombre;
        this.apellido = apellido;
        this.legajo = legajo;
        this.promedio = 0.0;
        this.activo = true;
    }

    // Getter — solo lectura
    public int getLegajo() {
        return legajo;
    }

    // El legajo no tiene setter — no se puede cambiar una vez asignado

    public String getNombre() {
        return nombre;
    }

    public void setNombre(String nombre) {
        if (nombre == null || nombre.isBlank()) {
            throw new IllegalArgumentException("El nombre no puede estar vacío.");
        }
        this.nombre = nombre.trim();
    }

    public String getApellido() {
        return apellido;
    }

    public void setApellido(String apellido) {
        if (apellido == null || apellido.isBlank()) {
            throw new IllegalArgumentException("El apellido no puede estar vacío.");
        }
        this.apellido = apellido.trim();
    }

    public double getPromedio() {
        return promedio;
    }

    public void setPromedio(double promedio) {
        // Validación dentro del setter
        if (promedio < 0.0 || promedio > 10.0) {
            throw new IllegalArgumentException("El promedio debe estar entre 0 y 10. Valor: " + promedio);
        }
        this.promedio = promedio;
    }

    public boolean isActivo() {
        return activo;
    }

    public void darDeBaja() {
        this.activo = false;
    }

    // Método derivado — no expone datos directamente
    public String getNombreCompleto() {
        return nombre + " " + apellido;
    }

    public boolean aprobo() {
        return promedio >= 6.0;
    }

    @Override
    public String toString() {
        return String.format("Estudiante{legajo=%d, nombre='%s', promedio=%.2f, activo=%b}",
            legajo, getNombreCompleto(), promedio, activo);
    }
}
```

### Uso del encapsulamiento

```java
public class Main {
    public static void main(String[] args) {
        Estudiante e = new Estudiante("Ana", "García", 1001);

        // e.promedio = 8.5; // ERROR: promedio es private

        e.setPromedio(8.5);   // OK: a través del setter con validación
        System.out.println(e.getPromedio()); // 8.5

        try {
            e.setPromedio(15.0); // Lanza excepción
        } catch (IllegalArgumentException ex) {
            System.out.println("Error: " + ex.getMessage());
        }

        // El legajo no puede cambiar (no hay setter)
        System.out.println("Legajo: " + e.getLegajo()); // solo lectura

        e.darDeBaja();
        System.out.println("Activo: " + e.isActivo()); // false
    }
}
```

---

## Inmutabilidad

Una clase **inmutable** es aquella cuyos objetos **no pueden cambiar de estado**
una vez creados. Son más seguras en entornos concurrentes.

```java
public final class CalificacionFinal {
    // Atributos final — solo se asignan una vez en el constructor
    private final int legajoEstudiante;
    private final String materia;
    private final double nota;
    private final String fecha;

    public CalificacionFinal(int legajoEstudiante, String materia,
                              double nota, String fecha) {
        if (nota < 0 || nota > 10) throw new IllegalArgumentException("Nota inválida");
        this.legajoEstudiante = legajoEstudiante;
        this.materia = materia;
        this.nota = nota;
        this.fecha = fecha;
    }

    // Solo getters, sin setters
    public int getLegajoEstudiante() { return legajoEstudiante; }
    public String getMateria() { return materia; }
    public double getNota() { return nota; }
    public String getFecha() { return fecha; }

    public boolean estaAprobada() {
        return nota >= 6.0;
    }

    @Override
    public String toString() {
        return String.format("Calificacion{materia='%s', nota=%.1f, estado='%s'}",
            materia, nota, estaAprobada() ? "Aprobado" : "Reprobado");
    }
}
```

> **Reglas para una clase inmutable:**
> 1. La clase es `final` (no puede ser heredada)
> 2. Todos los atributos son `private final`
> 3. Sin métodos setter
> 4. El constructor inicializa todos los atributos
> 5. Si hay atributos de tipo mutable (como arrays o listas), devuelve copias en los getters

---

## `enum` para valores fijos

Cuando un dato tiene un conjunto cerrado de opciones, `enum` ayuda a evitar errores de texto libre.

```java
public enum EstadoAcademico {
    APROBADO,
    DESAPROBADO,
    REGULAR
}

public class Estudiante {
    private EstadoAcademico estado = EstadoAcademico.REGULAR;

    public void aprobar() {
        this.estado = EstadoAcademico.APROBADO;
    }

    public EstadoAcademico getEstado() {
        return estado;
    }
}
```

> **Idea clave:** usa `enum` cuando quieres opciones fijas como estados, tipos o categorías.

---

## Records (Java 16+)

Java 16 introdujo los **Records**, que son clases inmutables de datos con sintaxis concisa:

```java
// Equivale a una clase inmutable con constructor, getters, equals, hashCode y toString
public record CalificacionRecord(
    int legajoEstudiante,
    String materia,
    double nota,
    String fecha
) {
    // Validación en el constructor compacto
    public CalificacionRecord {
        if (nota < 0 || nota > 10) throw new IllegalArgumentException("Nota inválida: " + nota);
        materia = materia.trim();
    }

    // Puedes agregar métodos adicionales
    public boolean estaAprobada() {
        return nota >= 6.0;
    }
}

// Uso
var calificacion = new CalificacionRecord(1001, "Matemática", 8.5, "2024-03-15");
System.out.println(calificacion.nota());     // 8.5
System.out.println(calificacion.materia());  // "Matemática"
System.out.println(calificacion);
// CalificacionRecord[legajoEstudiante=1001, materia=Matemática, nota=8.5, fecha=2024-03-15]
```

---

## Paquetes y Organización

Los paquetes organizan las clases y controlan la visibilidad.

```java
// Archivo: src/main/java/com/escuela/modelo/Estudiante.java
package com.escuela.modelo;

public class Estudiante {
    // ...
}

// Archivo: src/main/java/com/escuela/servicio/EstudianteServicio.java
package com.escuela.servicio;

import com.escuela.modelo.Estudiante; // importar de otro paquete

public class EstudianteServicio {
    public void procesar(Estudiante estudiante) {
        // ...
    }
}
```

### Estructura de paquetes recomendada para el proyecto escolar

```
com.escuela/
├── modelo/          → Clases de datos (Estudiante, Curso, Profesor)
├── servicio/        → Lógica de negocio (EstudianteServicio)
├── repositorio/     → Acceso a datos (EstudianteRepositorio)
├── excepcion/       → Excepciones personalizadas
└── util/            → Utilidades (ValidadorNota, FormatUtil)
```

---

## Resumen

| Concepto | Clave |
|----------|-------|
| `private` | Solo accesible dentro de la misma clase |
| Getter | Método que lee un atributo privado |
| Setter | Método que modifica un atributo con validación |
| Inmutabilidad | Sin setters + atributos `final` |
| `record` | Clase inmutable concisa (Java 16+) |

**Siguiente:** [2.3 Herencia y Polimorfismo](03-herencia-polimorfismo.md)
