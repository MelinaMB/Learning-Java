# 1.4 Métodos y Arrays

## Métodos

Un método es un bloque de código con nombre que realiza una tarea específica.
Los métodos permiten **reutilizar** código y **organizar** la lógica en piezas manejables.

### Anatomía de un método

```java
// modificador  tipoRetorno  nombre      (parámetros)
   public       static double calcularPromedio(double[] notas) {
       double suma = 0;
       for (double nota : notas) {
           suma += nota;
       }
       return suma / notas.length;  // valor de retorno
   }
```

### Método sin retorno (`void`)

```java
public static void mostrarBienvenida(String nombre) {
    System.out.println("Bienvenido/a, " + nombre + "!");
    System.out.println("Sistema de Gestión Escolar v2.0");
}
```

### Método con retorno

```java
public static double calcularPromedio(double nota1, double nota2, double nota3) {
    return (nota1 + nota2 + nota3) / 3.0;
}

public static String obtenerEstado(double promedio) {
    if (promedio >= 9.0) return "Sobresaliente";
    if (promedio >= 7.0) return "Bueno";
    if (promedio >= 6.0) return "Aprobado";
    return "Reprobado";
}
```

### Llamar un método

```java
public class GestorCalificaciones {
    public static void main(String[] args) {
        mostrarBienvenida("Ana García");

        double prom = calcularPromedio(8.5, 7.0, 9.0);
        String estado = obtenerEstado(prom);

        System.out.printf("Promedio: %.2f — Estado: %s%n", prom, estado);
    }

    public static void mostrarBienvenida(String nombre) {
        System.out.println("Bienvenido/a, " + nombre + "!");
    }

    public static double calcularPromedio(double n1, double n2, double n3) {
        return (n1 + n2 + n3) / 3.0;
    }

    public static String obtenerEstado(double promedio) {
        if (promedio >= 9.0) return "Sobresaliente";
        if (promedio >= 7.0) return "Bueno";
        if (promedio >= 6.0) return "Aprobado";
        return "Reprobado";
    }
}
```

---

## Sobrecarga de Métodos (Overloading)

Java permite tener múltiples métodos con el mismo nombre pero **diferentes parámetros**.

```java
public class CalculadoraNotas {

    // Versión con 2 notas
    public static double promedio(double n1, double n2) {
        return (n1 + n2) / 2.0;
    }

    // Versión con 3 notas
    public static double promedio(double n1, double n2, double n3) {
        return (n1 + n2 + n3) / 3.0;
    }

    // Versión con array
    public static double promedio(double[] notas) {
        double suma = 0;
        for (double n : notas) suma += n;
        return suma / notas.length;
    }

    public static void main(String[] args) {
        System.out.println(promedio(8.0, 7.0));               // 7.5
        System.out.println(promedio(8.0, 7.0, 9.0));          // 8.0
        System.out.println(promedio(new double[]{8, 7, 9, 6})); // 7.5
    }
}
```

---

## Varargs (número variable de argumentos)

```java
// El ... indica que acepta 0 o más argumentos del tipo indicado
public static double promedio(double... notas) {
    if (notas.length == 0) return 0;
    double suma = 0;
    for (double nota : notas) {
        suma += nota;
    }
    return suma / notas.length;
}

public static void main(String[] args) {
    System.out.println(promedio());                      // 0.0
    System.out.println(promedio(8.0));                   // 8.0
    System.out.println(promedio(8.0, 7.0, 9.0));        // 8.0
    System.out.println(promedio(7, 8, 9, 10, 6, 7, 8)); // 7.857...
}
```

> **Regla:** El parámetro varargs debe ser el último en la lista de parámetros.

---

## Recursión

Un método puede llamarse a sí mismo. Útil para problemas que se dividen en subproblemas del mismo tipo.

```java
// Calcular factorial: n! = n * (n-1) * (n-2) * ... * 1
public static long factorial(int n) {
    if (n <= 1) return 1;          // Caso base
    return n * factorial(n - 1);   // Llamada recursiva
}

// Calcular el n-ésimo número de Fibonacci
public static int fibonacci(int n) {
    if (n <= 1) return n;
    return fibonacci(n - 1) + fibonacci(n - 2);
}

public static void main(String[] args) {
    System.out.println(factorial(5));   // 120
    System.out.println(factorial(10));  // 3628800

    System.out.println(fibonacci(10));  // 55
}
```

---

## Arrays (Arreglos)

Un array es una colección de elementos **del mismo tipo** con tamaño **fijo**.

### Declarar y crear arrays

```java
// Declaración
int[] edades;
String[] nombres;
double[] notas;

// Crear con tamaño fijo (valores inicializados a 0/null/false)
edades = new int[5];       // 5 enteros, todos 0
nombres = new String[3];   // 3 Strings, todos null
notas = new double[4];     // 4 doubles, todos 0.0

// Declarar, crear e inicializar en una línea
int[] identificadores = {1001, 1002, 1003, 1004, 1005};
String[] materias = {"Matemática", "Física", "Química", "Historia"};
double[] calificaciones = {8.5, 7.0, 9.2, 6.5};
```

### Acceder y modificar elementos

```java
String[] estudiantes = new String[3];

// Asignar valores (índice comienza en 0)
estudiantes[0] = "Ana García";
estudiantes[1] = "Luis Pérez";
estudiantes[2] = "Carlos Ruiz";

// Leer valores
System.out.println(estudiantes[0]);          // Ana García
System.out.println(estudiantes[2]);          // Carlos Ruiz
System.out.println("Total: " + estudiantes.length); // Total: 3

// Modificar
estudiantes[1] = "Luis Alberto Pérez";
```

> **Error común:** Acceder a un índice fuera de rango lanza
> `ArrayIndexOutOfBoundsException`. Si el array tiene 3 elementos,
> el índice máximo es 2, no 3.

### Recorrer arrays

```java
double[] notas = {8.5, 7.0, 9.2, 6.5, 8.0};

// Con for clásico (cuando necesitas el índice)
for (int i = 0; i < notas.length; i++) {
    System.out.println("Nota " + (i + 1) + ": " + notas[i]);
}

// Con for-each (más limpio cuando no necesitas el índice)
for (double nota : notas) {
    System.out.println("Nota: " + nota);
}
```

---

## Operaciones comunes con arrays

```java
import java.util.Arrays;

public class OperacionesArrays {
    public static void main(String[] args) {
        double[] notas = {8.5, 7.0, 9.2, 6.5, 8.0, 7.5};

        // Encontrar mínimo y máximo
        double min = notas[0], max = notas[0];
        for (double nota : notas) {
            if (nota < min) min = nota;
            if (nota > max) max = nota;
        }
        System.out.println("Mínima: " + min);  // 6.5
        System.out.println("Máxima: " + max);  // 9.2

        // Calcular promedio
        double suma = 0;
        for (double nota : notas) suma += nota;
        double promedio = suma / notas.length;
        System.out.printf("Promedio: %.2f%n", promedio);

        // Ordenar (modifica el array original)
        Arrays.sort(notas);
        System.out.println("Ordenadas: " + Arrays.toString(notas));
        // [6.5, 7.0, 7.5, 8.0, 8.5, 9.2]

        // Buscar un valor (el array debe estar ordenado)
        int posicion = Arrays.binarySearch(notas, 8.0);
        System.out.println("Posición de 8.0: " + posicion); // 3

        // Copiar array
        double[] copia = Arrays.copyOf(notas, notas.length);
        double[] subcopia = Arrays.copyOfRange(notas, 1, 4); // [7.0, 7.5, 8.0]
        System.out.println("Copia: " + Arrays.toString(subcopia));

        // Rellenar con un valor
        double[] nuevasNotas = new double[5];
        Arrays.fill(nuevasNotas, 0.0);
        System.out.println("Rellenas: " + Arrays.toString(nuevasNotas));
    }
}
```

---

## Arrays Multidimensionales

```java
// Array 2D: tabla de notas [estudiante][materia]
double[][] notas = {
    {8.5, 9.0, 7.5},   // Ana:    Matemática, Física, Historia
    {7.0, 6.5, 8.0},   // Luis:   Matemática, Física, Historia
    {9.5, 8.5, 9.0}    // Carlos: Matemática, Física, Historia
};

String[] nombres = {"Ana", "Luis", "Carlos"};
String[] materias = {"Matemática", "Física", "Historia"};

// Recorrer e imprimir tabla
System.out.printf("%-10s %-12s %-10s %-10s %s%n",
    "Nombre", "Matemática", "Física", "Historia", "Promedio");
System.out.println("-".repeat(55));

for (int i = 0; i < nombres.length; i++) {
    double suma = 0;
    System.out.printf("%-10s", nombres[i]);
    for (int j = 0; j < materias.length; j++) {
        System.out.printf("%-12.1f", notas[i][j]);
        suma += notas[i][j];
    }
    System.out.printf("%.2f%n", suma / materias.length);
}
```

---

## Pasar arrays a métodos y retornarlos

```java
public class GestionNotas {

    // Recibe un array, no lo modifica (lectura)
    public static double calcularPromedio(double[] notas) {
        double suma = 0;
        for (double nota : notas) suma += nota;
        return suma / notas.length;
    }

    // Retorna un nuevo array
    public static double[] filtrarAprobados(double[] notas, double minima) {
        int cantidad = 0;
        for (double nota : notas) {
            if (nota >= minima) cantidad++;
        }
        double[] aprobados = new double[cantidad];
        int idx = 0;
        for (double nota : notas) {
            if (nota >= minima) aprobados[idx++] = nota;
        }
        return aprobados;
    }

    // Modifica el array recibido (los arrays se pasan por referencia)
    public static void agregarPuntoExtra(double[] notas, double puntos) {
        for (int i = 0; i < notas.length; i++) {
            notas[i] = Math.min(10.0, notas[i] + puntos); // máximo 10
        }
    }

    public static void main(String[] args) {
        double[] notas = {8.5, 5.0, 7.0, 4.5, 9.2, 6.0};

        System.out.printf("Promedio original: %.2f%n", calcularPromedio(notas));

        double[] aprobados = filtrarAprobados(notas, 6.0);
        System.out.println("Aprobados: " + java.util.Arrays.toString(aprobados));

        agregarPuntoExtra(notas, 0.5);
        System.out.println("Con punto extra: " + java.util.Arrays.toString(notas));
    }
}
```

> **Importante:** En Java, los arrays (y los objetos en general) se pasan **por referencia**.
> Si modificas el contenido dentro del método, el array original también cambia.
> Los tipos primitivos sí se pasan por valor (una copia).

---

## Resumen

| Concepto | Descripción |
|----------|-------------|
| Método | Bloque de código reutilizable con nombre |
| `void` | Método que no retorna valor |
| `return` | Sale del método y devuelve un valor |
| Sobrecarga | Mismo nombre, distintos parámetros |
| `varargs` | Acepta cantidad variable de argumentos (`tipo... nombre`) |
| Array | Colección de tamaño fijo del mismo tipo |
| `array.length` | Cantidad de elementos (no es un método, es una propiedad) |
| `Arrays.sort()` | Ordena un array en su lugar |
| Array 2D | `tipo[][] nombre` — tabla de filas y columnas |

**Siguiente:** [Ejercicios del Nivel 1](ejercicios.md)
