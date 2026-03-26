# 1.2 Variables y Tipos de Datos

## ¿Qué es una variable?

Una variable es un **contenedor con nombre** que almacena un valor en memoria.
En Java, toda variable tiene:
- Un **tipo** (qué clase de dato puede guardar)
- Un **nombre** (identificador)
- Un **valor** (el dato almacenado)

```java
// tipo  nombre  = valor;
   int   edad    = 20;
   String nombre = "Ana García";
   double promedio = 8.75;
```

---

## Tipos Primitivos

Java tiene **8 tipos primitivos**. Son los más básicos y se almacenan directamente en la pila de memoria.

| Tipo | Tamaño | Rango | Ejemplo |
|------|--------|-------|---------|
| `byte` | 8 bits | -128 a 127 | `byte nota = 10;` |
| `short` | 16 bits | -32,768 a 32,767 | `short año = 2024;` |
| `int` | 32 bits | ~-2.1 mil millones a ~2.1 mil millones | `int id = 1001;` |
| `long` | 64 bits | muy grande | `long matricula = 20240001L;` |
| `float` | 32 bits | ~6-7 decimales | `float promedio = 8.5f;` |
| `double` | 64 bits | ~15-16 decimales | `double gpa = 3.75;` |
| `boolean` | 1 bit | `true` o `false` | `boolean activo = true;` |
| `char` | 16 bits | Carácter Unicode | `char inicial = 'A';` |

```java
public class TiposPrimitivos {
    public static void main(String[] args) {
        // Enteros
        byte codigoNivel = 1;
        short anioIngreso = 2023;
        int idEstudiante = 100567;
        long matriculaUnica = 20230100567L;  // L al final para long

        // Decimales
        float notaParcial = 7.5f;            // f al final para float
        double promedioGeneral = 8.333333;

        // Booleano
        boolean estaActivo = true;
        boolean aprobo = false;

        // Carácter
        char turno = 'M';  // M de mañana

        System.out.println("ID: " + idEstudiante);
        System.out.println("Promedio: " + promedioGeneral);
        System.out.println("Aprobó: " + aprobo);
    }
}
```

> **Consejo:** Usa `int` para números enteros en la mayoría de los casos,
> y `double` para decimales. `float` y `long` tienen usos más específicos.

---

## El tipo String

`String` no es un tipo primitivo, sino una **clase**. Sin embargo, es tan fundamental
que tiene un tratamiento especial en Java.

```java
String nombre = "Ana García";
String apellido = "García López";

// Concatenación con +
String nombreCompleto = nombre + " " + apellido;
System.out.println(nombreCompleto); // Ana García García López

// Métodos útiles de String
String curso = "  Programación I  ";

System.out.println(curso.length());          // 20 (con espacios)
System.out.println(curso.trim());            // "Programación I"
System.out.println(curso.trim().length());   // 14
System.out.println(curso.toUpperCase());     // "  PROGRAMACIÓN I  "
System.out.println(curso.toLowerCase());     // "  programación i  "
System.out.println(curso.contains("Prog")); // true
System.out.println(curso.trim().replace("I", "1")); // "Programación 1"

// Comparación de Strings
String s1 = "hola";
String s2 = "hola";
String s3 = new String("hola");

System.out.println(s1 == s2);          // true (mismo objeto en el pool)
System.out.println(s1 == s3);          // false (objetos distintos)
System.out.println(s1.equals(s3));     // true (mismo contenido)
System.out.println(s1.equalsIgnoreCase("HOLA")); // true
```

> **Error común:** Nunca compares Strings con `==`. Usa siempre `.equals()`.
> El operador `==` compara referencias de memoria, no el contenido.

### Métodos importantes de String

```java
String texto = "Sistema Escolar 2024";

// Extraer partes
System.out.println(texto.substring(0, 7));    // "Sistema"
System.out.println(texto.substring(8));       // "Escolar 2024"
System.out.println(texto.charAt(0));          // 'S'
System.out.println(texto.indexOf("Escolar")); // 8

// Verificar
System.out.println(texto.startsWith("Sistema")); // true
System.out.println(texto.endsWith("2024"));       // true
System.out.println(texto.isEmpty());              // false
System.out.println("".isEmpty());                 // true
System.out.println("   ".isBlank());              // true (Java 11+)

// Dividir
String nombres = "Ana,Luis,Carlos,Marta";
String[] partes = nombres.split(",");
for (String n : partes) {
    System.out.println(n);
}
// Ana
// Luis
// Carlos
// Marta

// Formatear
String info = String.format("Estudiante: %s, Nota: %.2f", "Ana", 8.5);
System.out.println(info); // "Estudiante: Ana, Nota: 8.50"
```

---

## Clases Wrapper (Envolturas)

Cada tipo primitivo tiene una clase wrapper que lo "envuelve" y le añade métodos útiles.

| Primitivo | Wrapper |
|-----------|---------|
| `int` | `Integer` |
| `double` | `Double` |
| `boolean` | `Boolean` |
| `char` | `Character` |
| `long` | `Long` |
| `float` | `Float` |
| `byte` | `Byte` |
| `short` | `Short` |

```java
// Autoboxing: primitivo -> wrapper automáticamente
int numero = 42;
Integer numeroObj = numero;  // Java lo convierte automáticamente

// Unboxing: wrapper -> primitivo automáticamente
Integer valorObj = 100;
int valorPrim = valorObj;    // Java lo convierte automáticamente

// Métodos útiles de los wrappers
String textoNumero = "95";
int nota = Integer.parseInt(textoNumero);    // String -> int
double promedio = Double.parseDouble("8.5"); // String -> double
boolean estado = Boolean.parseBoolean("true"); // String -> boolean

// Constantes útiles
System.out.println(Integer.MAX_VALUE);  // 2147483647
System.out.println(Integer.MIN_VALUE);  // -2147483648
System.out.println(Double.MAX_VALUE);   // 1.7976931348623157E308

// Convertir a String
String s = Integer.toString(42);    // "42"
String s2 = String.valueOf(3.14);   // "3.14"
```

---

## Conversión de Tipos (Type Casting)

### Widening (implícita — sin pérdida de datos)

```java
int entero = 100;
long numeroGrande = entero;    // int -> long (automático)
double decimal = entero;       // int -> double (automático)

System.out.println(decimal); // 100.0
```

### Narrowing (explícita — puede haber pérdida)

```java
double promedio = 8.75;
int notaEntera = (int) promedio;  // double -> int (se pierde el decimal)
System.out.println(notaEntera);   // 8 (no 9, trunca, no redondea)

long matricula = 20240001L;
int id = (int) matricula;         // Puede perder datos si es muy grande
```

### Casting entre tipos de referencia

```java
// Lo veremos en profundidad en el Nivel 2 (Herencia)
Object obj = "Hola";
String texto = (String) obj;  // Downcasting
```

---

## Variables: declaración, inicialización y alcance

```java
public class AlcanceVariables {

    // Variable de clase (campo estático)
    static int contadorEstudiantes = 0;

    // Variable de instancia (campo de objeto)
    int idEstudiante;

    public void ejemplo() {
        // Variable local — solo existe dentro de este método
        String nombreLocal = "Ana";

        // Variable en bloque — solo existe dentro del if
        if (true) {
            int notaBloque = 10;
            System.out.println(notaBloque); // OK
        }
        // System.out.println(notaBloque); // ERROR: no existe aquí

        System.out.println(nombreLocal); // OK
    }
}
```

### Constantes con `final`

```java
public class Constantes {
    // Convención: MAYÚSCULAS con guión bajo
    public static final int NOTA_MAXIMA = 10;
    public static final int NOTA_APROBACION = 6;
    public static final String NOMBRE_SISTEMA = "Sistema Escolar";

    public static void main(String[] args) {
        // NOTA_MAXIMA = 11; // ERROR: no se puede modificar una constante
        System.out.println("Nota para aprobar: " + NOTA_APROBACION);
    }
}
```

---

## `var` — Inferencia de tipo (Java 10+)

Desde Java 10, puedes usar `var` y el compilador infiere el tipo automáticamente.

```java
var nombre = "Ana García";          // infiere String
var edad = 20;                       // infiere int
var promedio = 8.5;                  // infiere double
var activo = true;                   // infiere boolean

// El tipo se fija en la declaración, NO puedes cambiarlo después:
// var nombre = "Ana";
// nombre = 42;  // ERROR: nombre es String, no int
```

> **Consejo:** Usa `var` cuando el tipo es obvio por el lado derecho de la asignación.
> Evítalo cuando haga el código menos legible.

---

## Ejemplo integrador — Datos de un Estudiante

```java
public class DatosEstudiante {
    public static void main(String[] args) {
        // Datos básicos
        int id = 1001;
        String nombre = "María López";
        String apellido = "López Martínez";
        int edad = 19;
        char turno = 'M';  // M = mañana, T = tarde, N = noche
        boolean becado = true;

        // Datos académicos
        double notaPrimer = 8.5;
        double notaSegundo = 7.0;
        double notaTercer = 9.0;
        double promedio = (notaPrimer + notaSegundo + notaTercer) / 3.0;

        final double NOTA_APROBACION = 6.0;
        boolean aprobo = promedio >= NOTA_APROBACION;

        // Mostrar reporte
        System.out.println("============================");
        System.out.println("  REPORTE DE ESTUDIANTE");
        System.out.println("============================");
        System.out.println("ID:       " + id);
        System.out.println("Nombre:   " + nombre + " " + apellido);
        System.out.println("Edad:     " + edad + " años");
        System.out.println("Turno:    " + turno);
        System.out.println("Becado:   " + (becado ? "Sí" : "No"));
        System.out.println("----------------------------");
        System.out.printf("Nota 1:   %.1f%n", notaPrimer);
        System.out.printf("Nota 2:   %.1f%n", notaSegundo);
        System.out.printf("Nota 3:   %.1f%n", notaTercer);
        System.out.printf("Promedio: %.2f%n", promedio);
        System.out.println("Estado:   " + (aprobo ? "APROBADO" : "REPROBADO"));
        System.out.println("============================");
    }
}
```

---

## Resumen

| Concepto | Puntos clave |
|----------|-------------|
| Tipos primitivos | 8 tipos: byte, short, int, long, float, double, boolean, char |
| String | Clase especial, usa `.equals()` para comparar |
| Wrappers | Integer, Double, etc. — permiten conversión y métodos extra |
| Casting | Widening automático, narrowing requiere cast explícito `(tipo)` |
| `final` | Crea constantes que no pueden modificarse |
| `var` | Inferencia de tipo local (Java 10+) |

**Siguiente:** [1.3 Operadores y Control de Flujo](03-operadores-control.md)
