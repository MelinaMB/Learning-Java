# Ejercicios — Nivel 1: Fundamentos

Practica lo aprendido resolviendo estos ejercicios. Intenta resolverlos sin mirar la solución.

---

## Ejercicio 1 — Variables y Tipos

**Enunciado:** Crea un programa que declare variables para representar un estudiante
con los siguientes datos y los muestre por consola:

- Nombre completo (String)
- Legajo / matrícula (int)
- Promedio general (double)
- Turno (char: 'M', 'T' o 'N')
- Está activo (boolean)
- Año de ingreso (int)

La salida debe ser:

```
=== FICHA DE ESTUDIANTE ===
Nombre:    Ana García
Legajo:    10045
Promedio:  8.75
Turno:     M
Activo:    true
Ingresó:   2022
```

<details>
<summary>Ver solución</summary>

```java
public class Ejercicio1 {
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
</details>

---

## Ejercicio 2 — Operadores y Control de Flujo

**Enunciado:** Escribe un método `clasificarNota(double nota)` que reciba una nota
del 0 al 10 y devuelva su clasificación según esta tabla:

| Rango | Clasificación |
|-------|---------------|
| 9.0 - 10.0 | "Sobresaliente" |
| 8.0 - 8.9 | "Notable" |
| 7.0 - 7.9 | "Bien" |
| 6.0 - 6.9 | "Suficiente" |
| 0.0 - 5.9 | "Insuficiente" |
| otro | "Nota inválida" |

<details>
<summary>Ver solución</summary>

```java
public class Ejercicio2 {

    public static String clasificarNota(double nota) {
        if (nota < 0 || nota > 10) {
            return "Nota inválida";
        } else if (nota >= 9.0) {
            return "Sobresaliente";
        } else if (nota >= 8.0) {
            return "Notable";
        } else if (nota >= 7.0) {
            return "Bien";
        } else if (nota >= 6.0) {
            return "Suficiente";
        } else {
            return "Insuficiente";
        }
    }

    public static void main(String[] args) {
        double[] pruebas = {10.0, 9.5, 8.3, 7.1, 6.0, 4.5, -1.0, 11.0};
        for (double nota : pruebas) {
            System.out.printf("%.1f -> %s%n", nota, clasificarNota(nota));
        }
    }
}
```
</details>

---

## Ejercicio 3 — Bucles

**Enunciado:** Dado un array de notas `{7.5, 8.0, 5.5, 9.0, 6.0, 4.5, 8.5, 7.0}`,
calcula y muestra:

1. El promedio de todas las notas
2. La nota más alta y la más baja
3. Cuántos estudiantes aprobaron (nota >= 6.0)
4. Cuántos reprobaron

<details>
<summary>Ver solución</summary>

```java
public class Ejercicio3 {
    public static void main(String[] args) {
        double[] notas = {7.5, 8.0, 5.5, 9.0, 6.0, 4.5, 8.5, 7.0};

        double suma = 0, min = notas[0], max = notas[0];
        int aprobados = 0, reprobados = 0;

        for (double nota : notas) {
            suma += nota;
            if (nota < min) min = nota;
            if (nota > max) max = nota;
            if (nota >= 6.0) aprobados++;
            else reprobados++;
        }

        double promedio = suma / notas.length;

        System.out.printf("Promedio:   %.2f%n", promedio);
        System.out.printf("Máxima:     %.1f%n", max);
        System.out.printf("Mínima:     %.1f%n", min);
        System.out.printf("Aprobados:  %d/%d%n", aprobados, notas.length);
        System.out.printf("Reprobados: %d/%d%n", reprobados, notas.length);
    }
}
```
</details>

---

## Ejercicio 4 — Métodos

**Enunciado:** Crea una clase `UtilNotas` con los siguientes métodos estáticos:

- `double promedio(double... notas)` — promedio de todas las notas recibidas
- `boolean aprobo(double promedio)` — retorna true si el promedio >= 6.0
- `String menciones(double promedio)` — retorna "Con distinción" si promedio >= 9, "Con mérito" si >= 8, "" si no
- `void imprimirReporte(String nombre, double... notas)` — imprime un reporte completo

<details>
<summary>Ver solución</summary>

```java
public class UtilNotas {

    public static double promedio(double... notas) {
        if (notas.length == 0) return 0;
        double suma = 0;
        for (double n : notas) suma += n;
        return suma / notas.length;
    }

    public static boolean aprobo(double promedio) {
        return promedio >= 6.0;
    }

    public static String menciones(double promedio) {
        if (promedio >= 9.0) return "Con distinción";
        if (promedio >= 8.0) return "Con mérito";
        return "";
    }

    public static void imprimirReporte(String nombre, double... notas) {
        double prom = promedio(notas);
        String estado = aprobo(prom) ? "APROBADO" : "REPROBADO";
        String mencion = menciones(prom);

        System.out.println("Estudiante: " + nombre);
        System.out.printf("Promedio:   %.2f%n", prom);
        System.out.println("Estado:     " + estado + (mencion.isEmpty() ? "" : " — " + mencion));
        System.out.println("---");
    }

    public static void main(String[] args) {
        imprimirReporte("Ana García", 9.0, 8.5, 9.5);
        imprimirReporte("Luis Pérez", 7.0, 6.5, 7.5);
        imprimirReporte("Carlos Ruiz", 5.0, 4.5, 5.5);
    }
}
```
</details>

---

## Ejercicio 5 — Arrays 2D (Desafío)

**Enunciado:** Dada la siguiente tabla de notas de 4 estudiantes en 3 materias,
escribe un programa que calcule:

1. El promedio de cada estudiante
2. El promedio de cada materia (promedio de la columna)
3. El estudiante con mejor promedio general

```java
String[] estudiantes = {"Ana", "Luis", "Carlos", "Marta"};
String[] materias = {"Matemática", "Física", "Programación"};
double[][] notas = {
    {9.0, 8.5, 9.5},
    {7.0, 6.5, 8.0},
    {8.5, 7.5, 9.0},
    {6.0, 7.0, 6.5}
};
```

<details>
<summary>Ver solución</summary>

```java
public class Ejercicio5 {
    public static void main(String[] args) {
        String[] estudiantes = {"Ana", "Luis", "Carlos", "Marta"};
        String[] materias = {"Matemática", "Física", "Programación"};
        double[][] notas = {
            {9.0, 8.5, 9.5},
            {7.0, 6.5, 8.0},
            {8.5, 7.5, 9.0},
            {6.0, 7.0, 6.5}
        };

        // Promedios por estudiante
        double[] promediosEst = new double[estudiantes.length];
        for (int i = 0; i < estudiantes.length; i++) {
            double suma = 0;
            for (int j = 0; j < materias.length; j++) suma += notas[i][j];
            promediosEst[i] = suma / materias.length;
            System.out.printf("%-10s promedio: %.2f%n", estudiantes[i], promediosEst[i]);
        }

        System.out.println();

        // Promedios por materia
        for (int j = 0; j < materias.length; j++) {
            double suma = 0;
            for (int i = 0; i < estudiantes.length; i++) suma += notas[i][j];
            System.out.printf("%-15s promedio: %.2f%n", materias[j], suma / estudiantes.length);
        }

        System.out.println();

        // Mejor estudiante
        int mejorIdx = 0;
        for (int i = 1; i < promediosEst.length; i++) {
            if (promediosEst[i] > promediosEst[mejorIdx]) mejorIdx = i;
        }
        System.out.printf("Mejor estudiante: %s (%.2f)%n",
            estudiantes[mejorIdx], promediosEst[mejorIdx]);
    }
}
```
</details>

---

## Ejercicio 6 — Proyecto Integrador Nivel 1

**Enunciado:** Crea un programa que simule la carga de 5 estudiantes con sus notas.
El programa debe:

1. Usar arrays para guardar nombres y notas (una nota por estudiante)
2. Calcular el promedio del curso
3. Mostrar los aprobados (nota >= 6) y reprobados
4. Indicar cuántos estudiantes están por encima del promedio del curso
5. Mostrar el ranking (de mayor a menor nota)

> **Pista:** Para el ranking puedes copiar el array y ordenarlo con `Arrays.sort()`,
> o bien ordenarlo manualmente con un algoritmo de burbuja.

---

## Checklist de Nivel 1

Antes de pasar al Nivel 2, asegúrate de poder responder estas preguntas:

- [ ] ¿Cuáles son los 8 tipos primitivos y para qué sirve cada uno?
- [ ] ¿Por qué no debo usar `==` para comparar Strings?
- [ ] ¿Qué diferencia hay entre `++i` y `i++`?
- [ ] ¿Cuándo usarías `while` en lugar de `for`?
- [ ] ¿Qué es la sobrecarga de métodos?
- [ ] ¿Qué es un `ArrayIndexOutOfBoundsException` y cómo evitarlo?
- [ ] ¿Los arrays se pasan por valor o por referencia?

Si puedes responderlas, estás listo para el [Nivel 2 — POO](../nivel2-poo/README.md).
