# 5.2 Principios SOLID

SOLID es un acrónimo de 5 principios de diseño orientado a objetos que hacen
el software más mantenible, extensible y comprensible.

| Letra | Principio | En pocas palabras |
|-------|-----------|-------------------|
| **S** | Single Responsibility | Una clase = una razón para cambiar |
| **O** | Open/Closed | Abierto a extensión, cerrado a modificación |
| **L** | Liskov Substitution | Las subclases deben ser sustituibles por su padre |
| **I** | Interface Segregation | Interfaces pequeñas y específicas |
| **D** | Dependency Inversion | Depende de abstracciones, no de concretos |

---

## S — Principio de Responsabilidad Única (SRP)

Una clase debe tener **una sola razón para cambiar**.

```java
// MAL — EstudianteServicio hace demasiadas cosas
public class EstudianteServicio {
    public void guardarEstudiante(Estudiante e) { /* acceso a DB */ }
    public void enviarEmailBienvenida(Estudiante e) { /* envío de email */ }
    public String generarReportePDF(Estudiante e) { /* generación de PDF */ }
    public boolean validarDNI(String dni) { /* validación de formato */ }
}

// BIEN — cada clase tiene una responsabilidad
public class EstudianteRepositorio {
    public void guardar(Estudiante e) { /* acceso a DB */ }
    public Optional<Estudiante> buscarPorLegajo(int legajo) { ... }
}

public class NotificacionServicio {
    public void enviarBienvenida(Estudiante e) { /* envío de email */ }
    public void enviarAlerta(Estudiante e, String mensaje) { ... }
}

public class ReporteServicio {
    public byte[] generarPDF(Estudiante e) { /* generación de PDF */ }
    public String generarCSV(List<Estudiante> lista) { ... }
}

public class ValidadorDNI {
    public boolean esValido(String dni) { /* validación de formato */ }
}

public class EstudianteServicio {
    // Coordina las demás clases
    private final EstudianteRepositorio repo;
    private final NotificacionServicio notifServicio;
    private final ValidadorDNI validadorDNI;

    public void registrar(Estudiante e) {
        if (!validadorDNI.esValido(e.getDni()))
            throw new DniInvalidoException(e.getDni());
        repo.guardar(e);
        notifServicio.enviarBienvenida(e);
    }
}
```

---

## O — Principio Abierto/Cerrado (OCP)

Las clases deben estar **abiertas a extensión** pero **cerradas a modificación**.

```java
// MAL — cada vez que agrego un tipo de descuento, debo modificar esta clase
public class CalculadorDescuento {
    public double calcular(Estudiante e, String tipoDescuento) {
        if (tipoDescuento.equals("BECADO")) {
            return e.getMontoCuota() * 0.50;
        } else if (tipoDescuento.equals("HERMANOS")) {
            return e.getMontoCuota() * 0.20;
        } else if (tipoDescuento.equals("EMPLEADO")) {
            return e.getMontoCuota() * 0.30;
        }
        // Agregar nuevo tipo requiere modificar aquí — viola OCP
        return 0;
    }
}

// BIEN — extendemos sin modificar el código existente
public interface CalculadorDescuento {
    double calcular(double montoCuota);
    String getNombre();
}

public class DescuentoBecado implements CalculadorDescuento {
    @Override
    public double calcular(double montoCuota) { return montoCuota * 0.50; }
    @Override
    public String getNombre() { return "Becado (50%)"; }
}

public class DescuentoHermanos implements CalculadorDescuento {
    @Override
    public double calcular(double montoCuota) { return montoCuota * 0.20; }
    @Override
    public String getNombre() { return "Hermanos (20%)"; }
}

// Para agregar un nuevo descuento, creo una nueva clase — sin tocar las existentes
public class DescuentoEmpleado implements CalculadorDescuento {
    @Override
    public double calcular(double montoCuota) { return montoCuota * 0.30; }
    @Override
    public String getNombre() { return "Empleado (30%)"; }
}

public class ServicioCuota {
    public double calcularCuotaFinal(double montoCuota, CalculadorDescuento descuento) {
        double descuentoMonto = descuento.calcular(montoCuota);
        return montoCuota - descuentoMonto;
    }
}
```

---

## L — Principio de Sustitución de Liskov (LSP)

Los objetos de una subclase deben poder **reemplazar** a los de la superclase
sin alterar el comportamiento correcto del programa.

```java
// MAL — viola LSP
public class Rectangulo {
    protected int ancho, alto;
    public void setAncho(int ancho) { this.ancho = ancho; }
    public void setAlto(int alto) { this.alto = alto; }
    public int calcularArea() { return ancho * alto; }
}

public class Cuadrado extends Rectangulo {
    @Override
    public void setAncho(int ancho) {
        this.ancho = ancho;
        this.alto = ancho; // fuerza que sean iguales
    }
    @Override
    public void setAlto(int alto) {
        this.alto = alto;
        this.ancho = alto; // fuerza que sean iguales
    }
}

// Este código se rompe si usamos Cuadrado en lugar de Rectangulo:
Rectangulo r = new Cuadrado();
r.setAncho(5);
r.setAlto(3);
System.out.println(r.calcularArea()); // Espera 15, obtiene 9 — LSP violado!

// BIEN — modela correctamente la jerarquía
public interface Figura {
    int calcularArea();
}

public class Rectangulo implements Figura {
    private int ancho, alto;
    public Rectangulo(int ancho, int alto) { this.ancho = ancho; this.alto = alto; }
    @Override
    public int calcularArea() { return ancho * alto; }
}

public class Cuadrado implements Figura {
    private int lado;
    public Cuadrado(int lado) { this.lado = lado; }
    @Override
    public int calcularArea() { return lado * lado; }
}
```

```java
// Ejemplo escolar — LSP bien aplicado
public abstract class Evaluacion {
    protected String nombre;
    protected double peso; // peso en el promedio final

    public abstract double calcularNota();
    public double getPeso() { return peso; }
}

public class Parcial extends Evaluacion {
    private double notaTeorica;
    private double notaPractica;

    public Parcial(double notaTeorica, double notaPractica) {
        this.nombre = "Parcial";
        this.peso = 0.4;
        this.notaTeorica = notaTeorica;
        this.notaPractica = notaPractica;
    }

    @Override
    public double calcularNota() {
        return (notaTeorica * 0.6) + (notaPractica * 0.4);
    }
}

public class TrabajoPractico extends Evaluacion {
    private double nota;

    public TrabajoPractico(double nota) {
        this.nombre = "TP";
        this.peso = 0.2;
        this.nota = nota;
    }

    @Override
    public double calcularNota() { return nota; }
}

// Funciona con cualquier subclase de Evaluacion — LSP cumplido
public double calcularPromedioFinal(List<Evaluacion> evaluaciones) {
    return evaluaciones.stream()
        .mapToDouble(e -> e.calcularNota() * e.getPeso())
        .sum();
}
```

---

## I — Principio de Segregación de Interfaces (ISP)

Las interfaces deben ser **pequeñas y específicas**. Los clientes no deben
depender de métodos que no usan.

```java
// MAL — interfaz demasiado grande
public interface OperacionesEstudiante {
    void guardar(Estudiante e);
    void actualizar(Estudiante e);
    void eliminar(int legajo);
    List<Estudiante> listarTodos();
    Optional<Estudiante> buscarPorLegajo(int legajo);
    List<Estudiante> buscarPorCarrera(String carrera);
    void exportarCSV(String ruta);
    void exportarPDF(String ruta);
    void enviarNotificacion(int legajo, String msg);
    void generarReporte();
}

// BIEN — interfaces pequeñas y cohesivas
public interface EstudianteRepositorio {
    void guardar(Estudiante e);
    void actualizar(Estudiante e);
    void eliminar(int legajo);
    Optional<Estudiante> buscarPorLegajo(int legajo);
    List<Estudiante> listarTodos();
}

public interface EstudianteBuscador {
    List<Estudiante> buscarPorCarrera(String carrera);
    List<Estudiante> buscarPorPromedio(double minimo, double maximo);
}

public interface EstudianteExportador {
    void exportarCSV(List<Estudiante> lista, String ruta);
    void exportarPDF(List<Estudiante> lista, String ruta);
}

// Cada clase implementa solo lo que necesita
public class EstudianteRepositorioImpl implements EstudianteRepositorio, EstudianteBuscador {
    // Solo operaciones de datos
}

public class ReporteEstudiantesImpl implements EstudianteExportador {
    // Solo operaciones de exportación
}
```

---

## D — Principio de Inversión de Dependencias (DIP)

Los módulos de alto nivel no deben depender de módulos de bajo nivel.
**Ambos deben depender de abstracciones**.

```java
// MAL — ServicioInscripcion depende directamente de la implementación concreta
public class ServicioInscripcion {
    // Acoplado a MySQL — imposible cambiar a otro motor sin modificar esta clase
    private MySQLEstudianteRepositorio repositorio = new MySQLEstudianteRepositorio();

    public void inscribir(int legajo, String curso) {
        Estudiante e = repositorio.buscar(legajo);
        // ...
    }
}

// BIEN — depende de la abstracción (interfaz)
public class ServicioInscripcion {
    private final EstudianteRepositorio repositorio;  // INTERFAZ, no implementación
    private final NotificacionServicio notifServicio;

    // Inyección de dependencias — quien use este servicio elige la implementación
    public ServicioInscripcion(EstudianteRepositorio repositorio,
                                NotificacionServicio notifServicio) {
        this.repositorio = repositorio;
        this.notifServicio = notifServicio;
    }

    public void inscribir(int legajo, String curso) {
        Estudiante e = repositorio.buscarPorLegajo(legajo).orElseThrow();
        // ...
    }
}

// Ahora podemos usar cualquier implementación sin tocar ServicioInscripcion
// Para producción:
EstudianteRepositorio repo = new MySQLEstudianteRepositorio(conexion);
// Para testing:
EstudianteRepositorio repo = new EstudianteRepositorioMemoria();
// Para desarrollo local:
EstudianteRepositorio repo = new CSVEstudianteRepositorio("datos.csv");

ServicioInscripcion servicio = new ServicioInscripcion(repo, notifServicio);
```

---

## Resumen SOLID

| Principio | Detectar violación | Solución |
|-----------|-------------------|----------|
| SRP | La clase cambia por múltiples razones | Separar en clases más pequeñas |
| OCP | Agregar funcionalidad requiere modificar clase existente | Usar polimorfismo/estrategia |
| LSP | La subclase rompe invariantes del padre | Revisar jerarquía o usar composición |
| ISP | Clase implementa métodos que no usa | Dividir en interfaces más pequeñas |
| DIP | Clase crea sus propias dependencias con `new` | Inyección de dependencias |

**Siguiente:** [5.3 Refactoring](03-refactoring.md)
