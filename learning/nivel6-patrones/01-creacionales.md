# 6.1 Patrones Creacionales

Los patrones creacionales abstraen el proceso de instanciación de objetos,
haciéndolo más flexible y reutilizable.

---

## Singleton

**Propósito:** Garantizar que una clase tenga **una única instancia** y proveer
un punto de acceso global a ella.

**Cuándo usar:** Configuración de la aplicación, conexiones a bases de datos, logs.

```java
public class ConfiguracionSistema {
    private static ConfiguracionSistema instancia;

    private String nombreInstitucion;
    private int capacidadMaximaCurso;
    private double notaAprobacion;
    private String emailAdministrador;

    // Constructor privado — nadie puede instanciar desde afuera
    private ConfiguracionSistema() {
        this.nombreInstitucion = "Escuela Técnica N°1";
        this.capacidadMaximaCurso = 30;
        this.notaAprobacion = 6.0;
        this.emailAdministrador = "admin@escuela.edu";
    }

    // Método estático para obtener la única instancia
    public static synchronized ConfiguracionSistema getInstance() {
        if (instancia == null) {
            instancia = new ConfiguracionSistema();
        }
        return instancia;
    }

    // Getters
    public String getNombreInstitucion() { return nombreInstitucion; }
    public int getCapacidadMaximaCurso() { return capacidadMaximaCurso; }
    public double getNotaAprobacion() { return notaAprobacion; }
    public String getEmailAdministrador() { return emailAdministrador; }
}

// Uso
ConfiguracionSistema config = ConfiguracionSistema.getInstance();
System.out.println(config.getNombreInstitucion()); // Escuela Técnica N°1

// Es la misma instancia siempre
ConfiguracionSistema config2 = ConfiguracionSistema.getInstance();
System.out.println(config == config2); // true
```

---

## Factory Method

**Propósito:** Define una interfaz para crear objetos, pero permite a las subclases
decidir qué clase instanciar.

```java
// Producto
public interface Notificacion {
    void enviar(String destinatario, String mensaje);
    String getTipo();
}

// Productos concretos
public class NotificacionEmail implements Notificacion {
    @Override
    public void enviar(String destinatario, String mensaje) {
        System.out.println("[EMAIL] Para: " + destinatario + " | Msg: " + mensaje);
    }
    @Override
    public String getTipo() { return "EMAIL"; }
}

public class NotificacionSMS implements Notificacion {
    @Override
    public void enviar(String destinatario, String mensaje) {
        System.out.println("[SMS] Para: " + destinatario + " | Msg: " + mensaje.substring(0, Math.min(160, mensaje.length())));
    }
    @Override
    public String getTipo() { return "SMS"; }
}

public class NotificacionPush implements Notificacion {
    @Override
    public void enviar(String destinatario, String mensaje) {
        System.out.println("[PUSH] Para: " + destinatario + " | Msg: " + mensaje);
    }
    @Override
    public String getTipo() { return "PUSH"; }
}

// Fábrica
public class NotificacionFactory {
    public static Notificacion crear(String tipo) {
        return switch (tipo.toUpperCase()) {
            case "EMAIL" -> new NotificacionEmail();
            case "SMS"   -> new NotificacionSMS();
            case "PUSH"  -> new NotificacionPush();
            default -> throw new IllegalArgumentException("Tipo desconocido: " + tipo);
        };
    }
}

// Uso
Notificacion n = NotificacionFactory.crear("EMAIL");
n.enviar("ana@mail.com", "Examen mañana a las 9:00");

// Fácil agregar nuevos tipos sin cambiar código existente
```

---

## Builder

**Propósito:** Construir objetos complejos paso a paso. Permite crear diferentes
representaciones usando el mismo proceso de construcción.

**Cuándo usar:** Cuando el objeto tiene muchos parámetros opcionales en el constructor.

```java
public class Estudiante {
    // Atributos obligatorios
    private final String nombre;
    private final String apellido;
    private final String dni;

    // Atributos opcionales
    private final String email;
    private final String telefono;
    private final String carrera;
    private final int legajo;
    private final boolean becado;
    private final String turno;

    // Constructor privado — solo se usa desde el Builder
    private Estudiante(Builder builder) {
        this.nombre = builder.nombre;
        this.apellido = builder.apellido;
        this.dni = builder.dni;
        this.email = builder.email;
        this.telefono = builder.telefono;
        this.carrera = builder.carrera;
        this.legajo = builder.legajo;
        this.becado = builder.becado;
        this.turno = builder.turno;
    }

    // Getters...
    public String getNombre() { return nombre; }
    public String getNombreCompleto() { return nombre + " " + apellido; }
    public boolean isBecado() { return becado; }

    @Override
    public String toString() {
        return String.format("Estudiante{nombre='%s', dni='%s', carrera='%s', becado=%b}",
            getNombreCompleto(), dni, carrera, becado);
    }

    // Clase Builder estática interna
    public static class Builder {
        // Obligatorios
        private final String nombre;
        private final String apellido;
        private final String dni;

        // Opcionales con valores por defecto
        private String email = "";
        private String telefono = "";
        private String carrera = "Sin asignar";
        private int legajo = 0;
        private boolean becado = false;
        private String turno = "Mañana";

        public Builder(String nombre, String apellido, String dni) {
            this.nombre = nombre;
            this.apellido = apellido;
            this.dni = dni;
        }

        public Builder email(String email) { this.email = email; return this; }
        public Builder telefono(String tel) { this.telefono = tel; return this; }
        public Builder carrera(String carrera) { this.carrera = carrera; return this; }
        public Builder legajo(int legajo) { this.legajo = legajo; return this; }
        public Builder becado(boolean becado) { this.becado = becado; return this; }
        public Builder turno(String turno) { this.turno = turno; return this; }

        public Estudiante build() {
            // Validaciones
            if (nombre == null || nombre.isBlank())
                throw new IllegalStateException("El nombre es obligatorio.");
            return new Estudiante(this);
        }
    }
}

// Uso — muy legible con muchos parámetros opcionales
Estudiante ana = new Estudiante.Builder("Ana", "García", "12345678")
    .email("ana@mail.com")
    .carrera("Sistemas")
    .legajo(1001)
    .becado(true)
    .turno("Mañana")
    .build();

// Solo los obligatorios
Estudiante temp = new Estudiante.Builder("Luis", "Pérez", "87654321").build();
```

---

## Prototype

**Propósito:** Crear nuevos objetos **copiando** (clonando) un objeto existente.

```java
public class PlantillaCurso implements Cloneable {
    private String nombre;
    private int capacidad;
    private int horasSemanales;
    private List<String> materiasBase;

    public PlantillaCurso(String nombre, int capacidad, int horasSemanales) {
        this.nombre = nombre;
        this.capacidad = capacidad;
        this.horasSemanales = horasSemanales;
        this.materiasBase = new ArrayList<>();
    }

    public void agregarMateria(String materia) {
        materiasBase.add(materia);
    }

    // Clone profundo — copia también los objetos internos
    @Override
    public PlantillaCurso clone() {
        try {
            PlantillaCurso copia = (PlantillaCurso) super.clone();
            copia.materiasBase = new ArrayList<>(this.materiasBase); // copia la lista
            return copia;
        } catch (CloneNotSupportedException e) {
            throw new RuntimeException("Error al clonar", e);
        }
    }

    public void setNombre(String nombre) { this.nombre = nombre; }
    public String getNombre() { return nombre; }
}

// Uso
PlantillaCurso plantillaSistemas = new PlantillaCurso("Sistemas", 30, 40);
plantillaSistemas.agregarMateria("Programación");
plantillaSistemas.agregarMateria("Bases de Datos");
plantillaSistemas.agregarMateria("Redes");

// Crear variante sin repetir todo
PlantillaCurso cursoNocturno = plantillaSistemas.clone();
cursoNocturno.setNombre("Sistemas Nocturno");
```

---

## Resumen de Patrones Creacionales

| Patrón | Propósito | Cuándo usar |
|--------|-----------|-------------|
| **Singleton** | Una sola instancia | Configuración, log, conexión a BD |
| **Factory Method** | Delegar creación a subclases | Cuando el tipo depende del contexto |
| **Builder** | Construcción paso a paso | Objetos con muchos parámetros opcionales |
| **Prototype** | Clonar objetos existentes | Copia de configuraciones o plantillas |

**Siguiente:** [6.2 Patrones Estructurales](02-estructurales.md)
