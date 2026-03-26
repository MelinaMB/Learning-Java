# 1.3 Operadores y Control de Flujo

## Operadores en Java

### Operadores Aritméticos

```java
int a = 10, b = 3;

System.out.println(a + b);   // 13  suma
System.out.println(a - b);   // 7   resta
System.out.println(a * b);   // 30  multiplicación
System.out.println(a / b);   // 3   división entera (no 3.33!)
System.out.println(a % b);   // 1   módulo (resto de la división)

// Para división decimal, uno de los operandos debe ser double
double resultado = (double) a / b;
System.out.println(resultado); // 3.3333...

// División entera vs decimal
System.out.println(10 / 3);      // 3  (división entera)
System.out.println(10.0 / 3);    // 3.3333...
System.out.println(10 / 3.0);    // 3.3333...
```

### Operadores de Incremento/Decremento

```java
int nota = 7;

// Pre-incremento: incrementa y luego usa el valor
System.out.println(++nota); // 8

// Post-incremento: usa el valor y luego incrementa
System.out.println(nota++); // 8 (muestra 8, luego nota pasa a 9)
System.out.println(nota);   // 9

// Pre-decremento
System.out.println(--nota); // 8

// Post-decremento
System.out.println(nota--); // 8 (muestra 8, luego nota pasa a 7)
System.out.println(nota);   // 7
```

### Operadores de Asignación Compuesta

```java
int total = 100;
total += 50;   // total = total + 50  ->  150
total -= 20;   // total = total - 20  ->  130
total *= 2;    // total = total * 2   ->  260
total /= 4;    // total = total / 4   ->  65
total %= 10;   // total = total % 10  ->  5
```

### Operadores de Comparación

```java
int nota1 = 8, nota2 = 7;

System.out.println(nota1 == nota2);  // false  igual a
System.out.println(nota1 != nota2);  // true   distinto de
System.out.println(nota1 >  nota2);  // true   mayor que
System.out.println(nota1 <  nota2);  // false  menor que
System.out.println(nota1 >= nota2);  // true   mayor o igual que
System.out.println(nota1 <= nota2);  // false  menor o igual que
```

### Operadores Lógicos

```java
boolean estaInscrito = true;
boolean pagoCuota = false;
double promedio = 8.5;

// AND (&&): ambas condiciones deben ser verdaderas
boolean puedeRendirExamen = estaInscrito && pagoCuota;
System.out.println(puedeRendirExamen); // false

// OR (||): al menos una condición debe ser verdadera
boolean tieneAcceso = estaInscrito || pagoCuota;
System.out.println(tieneAcceso); // true

// NOT (!): invierte el valor booleano
boolean noInscrito = !estaInscrito;
System.out.println(noInscrito); // false

// Combinación
boolean aprobado = promedio >= 6.0 && estaInscrito && pagoCuota;
System.out.println(aprobado); // false (pagoCuota es false)
```

### Operador Ternario

```java
double promedio = 7.5;
String estado = promedio >= 6.0 ? "APROBADO" : "REPROBADO";
System.out.println(estado); // APROBADO

// Equivale a:
String estadoVerboso;
if (promedio >= 6.0) {
    estadoVerboso = "APROBADO";
} else {
    estadoVerboso = "REPROBADO";
}
```

---

## Sentencias Condicionales

### if / else if / else

```java
double promedio = 8.2;

if (promedio >= 9.0) {
    System.out.println("Excelente");
} else if (promedio >= 8.0) {
    System.out.println("Muy bien");
} else if (promedio >= 7.0) {
    System.out.println("Bien");
} else if (promedio >= 6.0) {
    System.out.println("Aprobado");
} else {
    System.out.println("Reprobado");
}
// Imprime: Muy bien
```

#### Ejemplo con múltiples condiciones en el sistema escolar

```java
public class EvaluarEstudiante {
    public static void main(String[] args) {
        String nombre = "Carlos Ruiz";
        double promedio = 7.8;
        int faltas = 3;
        int maxFaltas = 5;
        boolean pagoCuota = true;

        // Verificar si puede rendir examen final
        if (!pagoCuota) {
            System.out.println(nombre + " NO puede rendir: debe pagar la cuota.");
        } else if (faltas > maxFaltas) {
            System.out.println(nombre + " NO puede rendir: excedió las faltas (" + faltas + "/" + maxFaltas + ").");
        } else if (promedio < 4.0) {
            System.out.println(nombre + " NO puede rendir: promedio muy bajo (" + promedio + ").");
        } else {
            System.out.println(nombre + " SÍ puede rendir el examen final.");
            if (promedio >= 7.0) {
                System.out.println("Tiene buen promedio (" + promedio + "), está en ventaja.");
            }
        }
    }
}
```

### switch (clásico)

```java
int mes = 3;
String nombreMes;

switch (mes) {
    case 1:
        nombreMes = "Enero";
        break;
    case 2:
        nombreMes = "Febrero";
        break;
    case 3:
        nombreMes = "Marzo";
        break;
    // ... más casos
    case 12:
        nombreMes = "Diciembre";
        break;
    default:
        nombreMes = "Mes inválido";
        break;
}
System.out.println(nombreMes); // Marzo
```

> **Error común:** Olvidar el `break` provoca "fall-through" — se ejecutan los
> casos siguientes aunque no coincidan.

### switch expression (Java 14+)

```java
// Sintaxis moderna con ->
int dia = 3; // 1=Lunes ... 7=Domingo
String tipoDia = switch (dia) {
    case 1, 2, 3, 4, 5 -> "Día de clases";
    case 6              -> "Sábado — laboratorio optativo";
    case 7              -> "Domingo — sin clases";
    default             -> "Día inválido";
};
System.out.println(tipoDia); // Día de clases

// Con Strings
String turno = "mañana";
String horario = switch (turno.toLowerCase()) {
    case "mañana"  -> "07:00 - 13:00";
    case "tarde"   -> "13:00 - 19:00";
    case "noche"   -> "19:00 - 23:00";
    default        -> "Turno no reconocido";
};
System.out.println(horario); // 07:00 - 13:00
```

---

## Bucles (Loops)

### for

```java
// Formato: for (inicialización; condición; actualización)
for (int i = 1; i <= 5; i++) {
    System.out.println("Estudiante #" + i);
}

// Contar hacia atrás
for (int i = 10; i >= 1; i--) {
    System.out.println("Cuenta regresiva: " + i);
}

// Saltar de 2 en 2
for (int i = 0; i <= 10; i += 2) {
    System.out.println(i); // 0, 2, 4, 6, 8, 10
}
```

### while

```java
// Se ejecuta mientras la condición sea verdadera
int intentos = 0;
int maxIntentos = 3;
boolean autenticado = false;

while (intentos < maxIntentos && !autenticado) {
    intentos++;
    System.out.println("Intento " + intentos + " de acceso al sistema...");
    // Simula verificación
    if (intentos == 2) {
        autenticado = true;
    }
}

if (autenticado) {
    System.out.println("Acceso concedido.");
} else {
    System.out.println("Acceso denegado. Cuenta bloqueada.");
}
```

### do-while

```java
// Se ejecuta AL MENOS UNA VEZ antes de verificar la condición
int opcion;
do {
    System.out.println("\n=== MENÚ SISTEMA ESCOLAR ===");
    System.out.println("1. Ver estudiantes");
    System.out.println("2. Agregar estudiante");
    System.out.println("3. Ver cursos");
    System.out.println("0. Salir");
    System.out.println("Seleccione una opción: ");

    // En un programa real leerías la entrada; aquí simulamos:
    opcion = 0; // Simulamos que el usuario elige salir

} while (opcion != 0);
System.out.println("Hasta luego!");
```

### for-each (for mejorado)

```java
// Para iterar sobre arrays y colecciones
String[] estudiantes = {"Ana", "Luis", "Carlos", "Marta", "Pedro"};

for (String estudiante : estudiantes) {
    System.out.println("Estudiante: " + estudiante);
}

// Con arreglo de notas
double[] notas = {8.5, 7.0, 9.2, 6.5, 8.0};
double suma = 0;

for (double nota : notas) {
    suma += nota;
}

double promedio = suma / notas.length;
System.out.printf("Promedio de la clase: %.2f%n", promedio);
```

---

## Control de Bucles: break y continue

### break — sale del bucle inmediatamente

```java
String[] estudiantes = {"Ana", "Luis", "BAJA", "Carlos", "Marta"};

for (String estudiante : estudiantes) {
    if (estudiante.equals("BAJA")) {
        System.out.println("Se encontró un registro dado de baja. Deteniendo...");
        break; // Sale del bucle
    }
    System.out.println("Procesando: " + estudiante);
}
// Imprime: Ana, Luis, y luego el mensaje de baja
```

### continue — salta a la siguiente iteración

```java
double[] notas = {8.5, -1.0, 7.0, -1.0, 9.2}; // -1 indica ausente

double suma = 0;
int contadorValidas = 0;

for (double nota : notas) {
    if (nota < 0) {
        System.out.println("Nota inválida, omitiendo...");
        continue; // Salta al siguiente elemento
    }
    suma += nota;
    contadorValidas++;
}

if (contadorValidas > 0) {
    System.out.printf("Promedio (notas válidas): %.2f%n", suma / contadorValidas);
}
```

---

## Bucles Anidados

```java
// Tabla de calificaciones: estudiantes x materias
String[] estudiantes = {"Ana", "Luis", "Carlos"};
String[] materias = {"Matemática", "Lengua", "Historia"};
double[][] notas = {
    {9.0, 8.5, 7.0},   // notas de Ana
    {7.5, 8.0, 6.5},   // notas de Luis
    {8.0, 7.5, 9.5}    // notas de Carlos
};

System.out.printf("%-10s", "Estudiante");
for (String materia : materias) {
    System.out.printf("%-12s", materia);
}
System.out.println();
System.out.println("-".repeat(46));

for (int i = 0; i < estudiantes.length; i++) {
    System.out.printf("%-10s", estudiantes[i]);
    double suma = 0;
    for (int j = 0; j < materias.length; j++) {
        System.out.printf("%-12.1f", notas[i][j]);
        suma += notas[i][j];
    }
    System.out.printf("Prom: %.2f%n", suma / materias.length);
}
```

---

## Resumen

| Estructura | Uso |
|-----------|-----|
| `if/else if/else` | Decisiones con múltiples ramas |
| `switch` | Decisión según valor exacto de una variable |
| `for` | Bucle cuando sabes cuántas iteraciones |
| `while` | Bucle mientras se cumpla una condición |
| `do-while` | Bucle que se ejecuta al menos una vez |
| `for-each` | Recorrer arrays y colecciones |
| `break` | Salir del bucle |
| `continue` | Saltar a la siguiente iteración |

**Siguiente:** [1.4 Métodos y Arrays](04-metodos-arrays.md)
