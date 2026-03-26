# 4.3 Anotaciones y Reflexión

## Anotaciones Integradas en Java

Las anotaciones son metadatos que se pueden agregar a clases, métodos, campos, etc.

```java
public class Estudiante {

    @Override                    // El método sobreescribe uno del padre
    public String toString() { ... }

    @Deprecated                  // Método obsoleto — usar alternativa
    public void metodoViejo() { ... }

    @SuppressWarnings("unchecked") // Suprime advertencias del compilador
    public void metodoConWarning() { ... }

    @FunctionalInterface         // Marca una interfaz como funcional
    // (ya visto en Nivel 3)
}
```

## Crear Anotaciones Personalizadas

```java
import java.lang.annotation.*;

// @interface define la anotación
@Retention(RetentionPolicy.RUNTIME)  // disponible en tiempo de ejecución
@Target({ElementType.METHOD, ElementType.FIELD}) // dónde se puede usar
public @interface Validar {
    double min() default 0.0;
    double max() default 10.0;
    String mensaje() default "Valor fuera de rango";
}

@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE)
public @interface Entidad {
    String tabla();
    String esquema() default "public";
}

// Usar las anotaciones
@Entidad(tabla = "estudiantes", esquema = "escuela")
public class Estudiante {

    @Validar(min = 0, max = 10, mensaje = "La nota debe estar entre 0 y 10")
    private double promedio;

    @Validar(min = 1, max = 9999)
    private int legajo;
}
```

---

## Reflexión (Reflection)

La reflexión permite inspeccionar y manipular clases, métodos y campos
en tiempo de ejecución.

```java
import java.lang.reflect.*;

public class InspectorClase {

    public static void inspeccionarClase(Class<?> clase) {
        System.out.println("=== Clase: " + clase.getName() + " ===");

        // Superclase e interfaces
        System.out.println("Hereda de: " + clase.getSuperclass().getName());
        for (Class<?> interfaz : clase.getInterfaces()) {
            System.out.println("Implementa: " + interfaz.getName());
        }

        // Campos
        System.out.println("\n-- Campos --");
        for (Field campo : clase.getDeclaredFields()) {
            System.out.printf("  %s %s %s%n",
                Modifier.toString(campo.getModifiers()),
                campo.getType().getSimpleName(),
                campo.getName());
        }

        // Métodos
        System.out.println("\n-- Métodos --");
        for (Method metodo : clase.getDeclaredMethods()) {
            System.out.printf("  %s %s %s(%s)%n",
                Modifier.toString(metodo.getModifiers()),
                metodo.getReturnType().getSimpleName(),
                metodo.getName(),
                parametrosAString(metodo.getParameterTypes()));
        }

        // Constructores
        System.out.println("\n-- Constructores --");
        for (Constructor<?> constructor : clase.getDeclaredConstructors()) {
            System.out.printf("  %s(%s)%n",
                constructor.getName(),
                parametrosAString(constructor.getParameterTypes()));
        }
    }

    private static String parametrosAString(Class<?>[] tipos) {
        StringBuilder sb = new StringBuilder();
        for (int i = 0; i < tipos.length; i++) {
            if (i > 0) sb.append(", ");
            sb.append(tipos[i].getSimpleName());
        }
        return sb.toString();
    }
}

// Uso
InspectorClase.inspeccionarClase(Estudiante.class);
```

### Instanciar clases y llamar métodos por reflexión

```java
// Crear instancia dinámicamente
Class<?> clase = Class.forName("com.escuela.modelo.Estudiante");
Constructor<?> constructor = clase.getDeclaredConstructor(
    String.class, String.class, String.class, String.class, int.class, String.class);
Object instancia = constructor.newInstance("Ana", "García", "123", "a@b.com", 1001, "Sistemas");

// Llamar método por nombre
Method metodo = clase.getDeclaredMethod("getNombreCompleto");
String nombre = (String) metodo.invoke(instancia);
System.out.println(nombre); // Ana García

// Acceder a campo privado
Field campo = clase.getDeclaredField("promedio");
campo.setAccessible(true); // permite acceso a privados
double promedio = (double) campo.get(instancia);
campo.set(instancia, 9.5); // modificar
```

### Validador con reflexión y anotaciones

```java
public class Validador {

    public static void validar(Object objeto) throws IllegalAccessException {
        Class<?> clase = objeto.getClass();

        for (Field campo : clase.getDeclaredFields()) {
            if (campo.isAnnotationPresent(Validar.class)) {
                Validar anotacion = campo.getAnnotation(Validar.class);
                campo.setAccessible(true);
                Object valor = campo.get(objeto);

                if (valor instanceof Number) {
                    double num = ((Number) valor).doubleValue();
                    if (num < anotacion.min() || num > anotacion.max()) {
                        throw new IllegalStateException(
                            "Campo '" + campo.getName() + "': " + anotacion.mensaje() +
                            " (valor: " + num + ", rango: [" + anotacion.min() + ", " + anotacion.max() + "])");
                    }
                }
            }
        }
        System.out.println("Objeto válido: " + objeto.getClass().getSimpleName());
    }

    public static void main(String[] args) throws Exception {
        Estudiante e = new Estudiante("Ana", "García", "123", "a@b.com", 1001, "Sistemas");
        e.setPromedio(8.5);
        validar(e);  // Objeto válido: Estudiante

        e.setPromedio(11.0); // nota inválida
        validar(e);  // lanza IllegalStateException
    }
}
```

---

## Resumen

| Concepto | Descripción |
|----------|-------------|
| Anotación | Metadato que agrega información a código |
| `@Retention` | Dónde está disponible (SOURCE, CLASS, RUNTIME) |
| `@Target` | Dónde puede aplicarse (TYPE, METHOD, FIELD, ...) |
| `Class<?>` | Objeto que representa una clase en runtime |
| `Field` | Campo de una clase (por reflexión) |
| `Method` | Método de una clase (por reflexión) |
| `setAccessible(true)` | Permite acceder a miembros privados |

**Siguiente:** [Ejercicios del Nivel 4](ejercicios.md)
