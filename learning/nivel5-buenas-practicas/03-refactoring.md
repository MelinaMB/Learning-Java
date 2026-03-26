# 5.3 Refactoring

El refactoring es el proceso de **restructurar código existente** sin cambiar
su comportamiento externo, con el objetivo de mejorar su diseño interno.

> "Primero haz que funcione. Luego haz que sea correcto. Luego haz que sea rápido." — Kent Beck

---

## Code Smells (Malos Olores)

Un "code smell" es una señal de que algo en el código puede mejorarse.

### 1. Método largo

```java
// MAL — método de 60+ líneas que hace todo
public void procesarBoletin(int legajoEstudiante, int año, int trimestre) {
    // 10 líneas validando datos
    // 15 líneas calculando promedios
    // 10 líneas determinando estado
    // 15 líneas generando PDF
    // 10 líneas enviando email
}

// BIEN — extraer métodos
public void procesarBoletin(int legajoEstudiante, int año, int trimestre) {
    validarDatos(legajoEstudiante, año, trimestre);
    double promedio = calcularPromedio(legajoEstudiante, año, trimestre);
    EstadoAcademico estado = determinarEstado(promedio);
    byte[] pdf = generarBoletin(legajoEstudiante, promedio, estado);
    enviarBoletin(legajoEstudiante, pdf);
}
```

### 2. Clase Dios (God Class)

```java
// MAL — una clase que lo sabe y hace todo
public class SistemaEscolar {
    // 50 atributos...
    // 100 métodos...
    public void inscribir() { ... }
    public void calcularNota() { ... }
    public void enviarEmail() { ... }
    public void generarReporte() { ... }
    public void conectarBD() { ... }
}

// BIEN — responsabilidades distribuidas
public class ServicioInscripcion { ... }
public class ServicioCalificaciones { ... }
public class NotificacionServicio { ... }
public class ReporteServicio { ... }
public class ConexionBD { ... }
```

### 3. Código duplicado

```java
// MAL — lógica duplicada en dos métodos
public String obtenerEstadoParcial(double notaParcial) {
    if (notaParcial >= 9.0) return "Sobresaliente";
    if (notaParcial >= 7.0) return "Bueno";
    if (notaParcial >= 6.0) return "Aprobado";
    return "Reprobado";
}

public String obtenerEstadoFinal(double notaFinal) {
    if (notaFinal >= 9.0) return "Sobresaliente";
    if (notaFinal >= 7.0) return "Bueno";
    if (notaFinal >= 6.0) return "Aprobado";
    return "Reprobado";
}

// BIEN — extraer a método compartido
public String obtenerEstado(double nota) {
    if (nota >= 9.0) return "Sobresaliente";
    if (nota >= 7.0) return "Bueno";
    if (nota >= 6.0) return "Aprobado";
    return "Reprobado";
}
```

### 4. Números mágicos

```java
// MAL
if (promedio >= 6.0 && faltas <= 5) { ... }
double descuento = monto * 0.15;

// BIEN
private static final double PROMEDIO_MINIMO = 6.0;
private static final int MAX_FALTAS = 5;
private static final double PORCENTAJE_DESCUENTO = 0.15;

if (promedio >= PROMEDIO_MINIMO && faltas <= MAX_FALTAS) { ... }
double descuento = monto * PORCENTAJE_DESCUENTO;
```

### 5. Cadenas de condiciones largas

```java
// MAL
public double calcularDescuento(Estudiante e) {
    if (e.isBecado()) {
        return 0.5;
    } else if (e.getTieneHermanosInscritos()) {
        return 0.2;
    } else if (e.esHijoDeEmpleado()) {
        return 0.3;
    } else if (e.tieneDiscapacidad()) {
        return 1.0;
    } else {
        return 0.0;
    }
}

// BIEN — usando patrón Strategy/polimorfismo
// (Ver Nivel 6 — Patrones de Diseño)
public interface PoliticaDescuento {
    boolean aplica(Estudiante e);
    double porcentaje();
}

public class DescuentoBeca implements PoliticaDescuento {
    public boolean aplica(Estudiante e) { return e.isBecado(); }
    public double porcentaje() { return 0.5; }
}
```

---

## Técnicas de Refactoring

### Extract Method — Extraer Método

```java
// ANTES
public void mostrarReporte(Estudiante e) {
    System.out.println("=".repeat(40));
    System.out.println("  REPORTE ACADÉMICO");
    System.out.println("=".repeat(40));
    System.out.println("Nombre: " + e.getNombreCompleto());
    System.out.println("Legajo: " + e.getLegajo());

    double suma = 0;
    for (double n : e.getNotas()) suma += n;
    double promedio = suma / e.getNotas().size();
    System.out.printf("Promedio: %.2f%n", promedio);

    if (promedio >= 6.0) {
        System.out.println("Estado: APROBADO");
    } else {
        System.out.println("Estado: REPROBADO");
    }
    System.out.println("=".repeat(40));
}

// DESPUÉS
public void mostrarReporte(Estudiante e) {
    imprimirEncabezado("REPORTE ACADÉMICO");
    imprimirDatosPersonales(e);
    imprimirResultadoAcademico(e);
    imprimirPie();
}

private void imprimirEncabezado(String titulo) {
    System.out.println("=".repeat(40));
    System.out.println("  " + titulo);
    System.out.println("=".repeat(40));
}

private void imprimirDatosPersonales(Estudiante e) {
    System.out.println("Nombre: " + e.getNombreCompleto());
    System.out.println("Legajo: " + e.getLegajo());
}

private void imprimirResultadoAcademico(Estudiante e) {
    double promedio = e.calcularPromedio();
    System.out.printf("Promedio: %.2f%n", promedio);
    System.out.println("Estado: " + (promedio >= 6.0 ? "APROBADO" : "REPROBADO"));
}

private void imprimirPie() {
    System.out.println("=".repeat(40));
}
```

### Replace Conditional with Polymorphism

```java
// ANTES
public double calcularSalario(PersonalEscolar p) {
    if (p.getTipo().equals("TIEMPO_COMPLETO")) {
        return p.getSalarioBase() * (1 + p.getAntiguedad() * 0.02);
    } else if (p.getTipo().equals("HORAS")) {
        return p.getHorasTrabajadas() * p.getValorHora();
    } else if (p.getTipo().equals("DIRECTIVO")) {
        return p.getSalarioBase() + 5000;
    }
    throw new IllegalStateException("Tipo desconocido: " + p.getTipo());
}

// DESPUÉS
public abstract class PersonalEscolar {
    public abstract double calcularSalario(); // cada subclase lo implementa
}

public class ProfesorTiempoCompleto extends PersonalEscolar {
    @Override
    public double calcularSalario() {
        return salarioBase * (1 + antiguedad * 0.02);
    }
}

public class ProfesorPorHoras extends PersonalEscolar {
    @Override
    public double calcularSalario() {
        return horasTrabajadas * valorHora;
    }
}
```

---

## Resumen

| Code Smell | Señal | Técnica |
|-----------|-------|---------|
| Método largo | > 20 líneas, varias responsabilidades | Extract Method |
| Clase Dios | Muchos atributos y métodos sin cohesión | Extract Class |
| Código duplicado | Misma lógica en varios lugares | Extract Method / Template Method |
| Números mágicos | Literales sin nombre en el código | Replace Magic Number with Constant |
| Cadenas de if | `if/else if` sobre tipo de objeto | Replace Conditional with Polymorphism |
| Comentarios innecesarios | Explican código obvio | Rename / Extract Method |

**Siguiente:** [Nivel 6 — Patrones de Diseño](../nivel6-patrones/README.md)
