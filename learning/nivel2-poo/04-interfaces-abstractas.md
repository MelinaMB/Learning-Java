# 2.4 Interfaces y Clases Abstractas

## Clases Abstractas

Una clase abstracta es una clase que **no puede instanciarse directamente**.
Sirve como base común para un grupo de clases relacionadas.
Puede tener tanto métodos abstractos (sin implementación) como métodos concretos.

```java
// No puede hacer: new EntidadEscolar() — ERROR
public abstract class EntidadEscolar {
    private int id;
    private String nombre;
    private java.time.LocalDate fechaCreacion;

    public EntidadEscolar(int id, String nombre) {
        this.id = id;
        this.nombre = nombre;
        this.fechaCreacion = java.time.LocalDate.now();
    }

    // Métodos concretos — heredados tal cual
    public int getId() { return id; }
    public String getNombre() { return nombre; }
    public java.time.LocalDate getFechaCreacion() { return fechaCreacion; }

    // Método abstracto — OBLIGATORIO sobreescribir en subclases
    public abstract String getTipo();

    // Método abstracto
    public abstract String getDescripcionCompleta();

    // Método concreto que usa abstractos
    public void mostrarInfo() {
        System.out.println("=== " + getTipo() + " ===");
        System.out.println(getDescripcionCompleta());
        System.out.println("Creado: " + fechaCreacion);
    }
}
```

### Implementar la clase abstracta

```java
public class Curso extends EntidadEscolar {
    private String codigo;
    private int año;
    private int division;
    private int capacidadMaxima;

    public Curso(int id, String nombre, String codigo, int año, int division) {
        super(id, nombre);
        this.codigo = codigo;
        this.año = año;
        this.division = division;
        this.capacidadMaxima = 30;
    }

    @Override
    public String getTipo() {
        return "Curso";
    }

    @Override
    public String getDescripcionCompleta() {
        return String.format("%s — %s (Año: %d°, División: %d°, Cap: %d)",
            codigo, getNombre(), año, division, capacidadMaxima);
    }
}

public class Materia extends EntidadEscolar {
    private String area;
    private int horasSemanales;

    public Materia(int id, String nombre, String area, int horasSemanales) {
        super(id, nombre);
        this.area = area;
        this.horasSemanales = horasSemanales;
    }

    @Override
    public String getTipo() { return "Materia"; }

    @Override
    public String getDescripcionCompleta() {
        return String.format("%s | Área: %s | %d hs/semana",
            getNombre(), area, horasSemanales);
    }
}
```

---

## Interfaces

Una interfaz define un **contrato**: qué métodos debe implementar cualquier clase
que la adopte. No define *cómo*, sino *qué*.

```java
// Una interfaz define un contrato
public interface Evaluable {
    // Constante implícitamente public static final
    double NOTA_MINIMA_APROBACION = 6.0;

    // Métodos abstractos — implícitamente public abstract
    double calcularPromedio();
    boolean estaAprobado();
    String obtenerEstado();
}

public interface Exportable {
    String exportarCSV();
    String exportarJSON();
}

public interface Notificable {
    void enviarNotificacion(String mensaje);
    String getContacto();
}
```

### Implementar interfaces

```java
// Una clase puede implementar múltiples interfaces
public class Estudiante extends Persona implements Evaluable, Notificable {

    private int legajo;
    private String carrera;
    private double[] notas;
    private String email;

    public Estudiante(String nombre, String apellido, String dni,
                      String email, int legajo, String carrera) {
        super(nombre, apellido, dni, email);
        this.legajo = legajo;
        this.carrera = carrera;
        this.email = email;
        this.notas = new double[0];
    }

    public void agregarNota(double nota) {
        double[] nuevasNotas = new double[notas.length + 1];
        System.arraycopy(notas, 0, nuevasNotas, 0, notas.length);
        nuevasNotas[notas.length] = nota;
        this.notas = nuevasNotas;
    }

    // Implementación de Evaluable
    @Override
    public double calcularPromedio() {
        if (notas.length == 0) return 0.0;
        double suma = 0;
        for (double n : notas) suma += n;
        return suma / notas.length;
    }

    @Override
    public boolean estaAprobado() {
        return calcularPromedio() >= NOTA_MINIMA_APROBACION;
    }

    @Override
    public String obtenerEstado() {
        double prom = calcularPromedio();
        if (prom >= 9.0) return "Sobresaliente";
        if (prom >= 7.0) return "Bueno";
        if (prom >= 6.0) return "Aprobado";
        return "Reprobado";
    }

    // Implementación de Notificable
    @Override
    public void enviarNotificacion(String mensaje) {
        System.out.println("Email a " + getContacto() + ": " + mensaje);
    }

    @Override
    public String getContacto() {
        return email;
    }
}
```

---

## Métodos default en interfaces (Java 8+)

Las interfaces pueden tener métodos con implementación usando `default`.

```java
public interface Evaluable {
    double NOTA_MINIMA_APROBACION = 6.0;

    double calcularPromedio();
    boolean estaAprobado();

    // Método default — tiene implementación, puede sobreescribirse
    default String obtenerEstado() {
        double prom = calcularPromedio();
        if (prom >= 9.0) return "Sobresaliente";
        if (prom >= 8.0) return "Notable";
        if (prom >= 7.0) return "Bien";
        if (prom >= 6.0) return "Suficiente";
        return "Insuficiente";
    }

    default String obtenerReporte() {
        return String.format("Promedio: %.2f | Estado: %s | Aprobado: %b",
            calcularPromedio(), obtenerEstado(), estaAprobado());
    }

    // Método estático en interfaces (Java 8+)
    static boolean esNotaValida(double nota) {
        return nota >= 0.0 && nota <= 10.0;
    }
}
```

---

## ¿Clase Abstracta o Interfaz?

| Criterio | Clase Abstracta | Interfaz |
|----------|----------------|---------|
| Estado (atributos) | Puede tener | Solo constantes |
| Constructores | Puede tener | No |
| Herencia múltiple | Solo una | Múltiples `implements` |
| Relación | "es un" | "puede hacer" / "tiene capacidad de" |
| Modificadores | `public`, `protected`, `private` | Solo `public` |
| Cuándo usar | Clases muy relacionadas con estado compartido | Contrato para clases no relacionadas |

**Regla práctica:**
- Usa **clase abstracta** cuando las subclases comparten código real y son del mismo "tipo"
- Usa **interfaz** para definir capacidades que clases distintas pueden tener

```java
// Clase abstracta — Persona es la base real de Estudiante y Profesor
abstract class Persona { ... }
class Estudiante extends Persona { ... }
class Profesor extends Persona { ... }

// Interfaces — capacidades que pueden tener clases distintas
interface Evaluable { ... }      // Estudiante puede ser evaluado
interface Exportable { ... }     // Reporte, Ficha, y Curso pueden exportarse
interface Archivable { ... }     // Documentos y registros pueden archivarse

class Estudiante extends Persona implements Evaluable, Exportable { ... }
class Curso implements Exportable, Archivable { ... }
```

---

## Polimorfismo con interfaces

```java
public class SistemaReporte {

    // Trabaja con cualquier cosa que sea Evaluable
    public static void generarReporte(Evaluable entidad) {
        System.out.println(entidad.obtenerReporte());
    }

    // Trabaja con cualquier cosa que sea Notificable
    public static void notificar(Notificable contacto, String mensaje) {
        contacto.enviarNotificacion(mensaje);
    }

    public static void main(String[] args) {
        Estudiante ana = new Estudiante("Ana", "García", "123", "ana@mail.com", 1001, "Sistemas");
        ana.agregarNota(8.5);
        ana.agregarNota(7.0);
        ana.agregarNota(9.0);

        generarReporte(ana);   // Estudiante como Evaluable
        notificar(ana, "Examen el viernes a las 10:00");  // Estudiante como Notificable
    }
}
```

---

## Interfaces funcionales (preview para Nivel 3)

Una interfaz funcional tiene exactamente **un método abstracto**.
Se usan con expresiones lambda (tema del Nivel 3).

```java
@FunctionalInterface
public interface CalculadorNota {
    double calcular(double nota1, double nota2, double nota3);
}

// Uso con lambda
CalculadorNota promedio = (n1, n2, n3) -> (n1 + n2 + n3) / 3.0;
CalculadorNota mejorNota = (n1, n2, n3) -> Math.max(n1, Math.max(n2, n3));

System.out.println(promedio.calcular(8, 7, 9));    // 8.0
System.out.println(mejorNota.calcular(8, 7, 9));   // 9.0
```

---

## Resumen

| Concepto | Clave |
|----------|-------|
| Clase abstracta | No instanciable, puede tener métodos con y sin implementación |
| `abstract` en método | Debe ser implementado por las subclases |
| Interfaz | Contrato puro, implementación con `implements` |
| `default` en interfaz | Método con implementación en la interfaz (Java 8+) |
| Interfaz funcional | Una sola método abstracto, usada con lambdas |
| Herencia múltiple | Java no la permite con clases, sí con interfaces |

**Siguiente:** [Ejercicios del Nivel 2](ejercicios.md)
