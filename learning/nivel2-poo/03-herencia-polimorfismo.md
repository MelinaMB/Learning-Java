# 2.3 Herencia y Polimorfismo

## Herencia

La herencia permite que una clase (**subclase** o clase hija) adquiera los atributos
y métodos de otra clase (**superclase** o clase padre). Modela la relación **"es un"**.

```
Persona (superclase)
   ├── Estudiante (subclase)
   ├── Profesor (subclase)
   └── Administrador (subclase)
```

### Clase base: Persona

```java
public class Persona {
    private String nombre;
    private String apellido;
    private String dni;
    private String email;

    public Persona(String nombre, String apellido, String dni, String email) {
        this.nombre = nombre;
        this.apellido = apellido;
        this.dni = dni;
        this.email = email;
    }

    public String getNombreCompleto() {
        return nombre + " " + apellido;
    }

    public String getDni() { return dni; }
    public String getNombre() { return nombre; }
    public String getApellido() { return apellido; }
    public String getEmail() { return email; }

    public void setEmail(String email) {
        if (email == null || !email.contains("@")) {
            throw new IllegalArgumentException("Email inválido: " + email);
        }
        this.email = email;
    }

    public void presentarse() {
        System.out.println("Hola, soy " + getNombreCompleto() + ".");
    }

    @Override
    public String toString() {
        return String.format("Persona{nombre='%s', dni='%s'}",
            getNombreCompleto(), dni);
    }
}
```

### Subclase: Estudiante

```java
public class Estudiante extends Persona {  // "extends" indica herencia
    private int legajo;
    private double promedio;
    private String carrera;

    // Constructor debe llamar al constructor del padre con super()
    public Estudiante(String nombre, String apellido, String dni,
                      String email, int legajo, String carrera) {
        super(nombre, apellido, dni, email);  // Llama al constructor de Persona
        this.legajo = legajo;
        this.carrera = carrera;
        this.promedio = 0.0;
    }

    public int getLegajo() { return legajo; }
    public double getPromedio() { return promedio; }
    public String getCarrera() { return carrera; }

    public void setPromedio(double promedio) {
        if (promedio < 0 || promedio > 10)
            throw new IllegalArgumentException("Promedio inválido");
        this.promedio = promedio;
    }

    // Sobreescribir (override) el método del padre
    @Override
    public void presentarse() {
        // Llamamos al método del padre con super
        super.presentarse();
        System.out.println("Soy estudiante de " + carrera + ", legajo " + legajo + ".");
    }

    @Override
    public String toString() {
        return String.format("Estudiante{legajo=%d, nombre='%s', carrera='%s', promedio=%.2f}",
            legajo, getNombreCompleto(), carrera, promedio);
    }
}
```

### Subclase: Profesor

```java
public class Profesor extends Persona {
    private String especialidad;
    private double salario;
    private int horasSemanales;

    public Profesor(String nombre, String apellido, String dni,
                    String email, String especialidad, double salario) {
        super(nombre, apellido, dni, email);
        this.especialidad = especialidad;
        this.salario = salario;
        this.horasSemanales = 20;
    }

    public String getEspecialidad() { return especialidad; }
    public double getSalario() { return salario; }
    public int getHorasSemanales() { return horasSemanales; }

    public void aplicarAumento(double porcentaje) {
        salario += salario * (porcentaje / 100.0);
    }

    @Override
    public void presentarse() {
        super.presentarse();
        System.out.println("Soy profesor/a de " + especialidad + ".");
    }

    @Override
    public String toString() {
        return String.format("Profesor{nombre='%s', especialidad='%s'}",
            getNombreCompleto(), especialidad);
    }
}
```

---

## Polimorfismo

El polimorfismo ("muchas formas") permite tratar objetos de distintas subclases
de manera uniforme a través del tipo de la superclase.

```java
public class DemoPolimorfismo {
    public static void main(String[] args) {
        // Una variable de tipo Persona puede contener un Estudiante o Profesor
        Persona[] personas = {
            new Estudiante("Ana", "García", "12345678", "ana@mail.com", 1001, "Sistemas"),
            new Profesor("Roberto", "Sánchez", "87654321", "rsanchez@uni.com", "Matemática", 60000),
            new Estudiante("Luis", "Pérez", "11223344", "luis@mail.com", 1002, "Redes"),
        };

        // Llama al método correcto según el tipo real del objeto
        for (Persona p : personas) {
            p.presentarse();  // Polimorfismo en acción
            System.out.println();
        }
    }
}
```

Salida:
```
Hola, soy Ana García.
Soy estudiante de Sistemas, legajo 1001.

Hola, soy Roberto Sánchez.
Soy profesor/a de Matemática.

Hola, soy Luis Pérez.
Soy estudiante de Redes, legajo 1002.
```

---

## Casting de objetos

```java
Persona p = new Estudiante("Ana", "García", "123", "ana@mail.com", 1001, "Sistemas");

// Upcasting — automático (siempre seguro)
// (ya ocurrió en la asignación anterior)

// Verificar el tipo real antes de hacer downcasting
if (p instanceof Estudiante) {
    // Downcasting — explícito
    Estudiante e = (Estudiante) p;
    System.out.println("Legajo: " + e.getLegajo());
}

// Pattern matching con instanceof (Java 16+)
if (p instanceof Estudiante estudiante) {
    System.out.println("Es estudiante: " + estudiante.getCarrera());
} else if (p instanceof Profesor profesor) {
    System.out.println("Es profesor de: " + profesor.getEspecialidad());
}
```

---

## La clase Object

Toda clase en Java hereda implícitamente de `Object`. Esta clase provee métodos básicos:

| Método | Descripción |
|--------|-------------|
| `toString()` | Representación en String |
| `equals(Object o)` | Comparación por contenido |
| `hashCode()` | Código hash para uso en colecciones |
| `getClass()` | Retorna la clase del objeto |

```java
public class Estudiante extends Persona {
    private int legajo;

    // ... constructor y otros métodos ...

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;            // mismo objeto
        if (!(o instanceof Estudiante)) return false; // tipo diferente
        Estudiante otro = (Estudiante) o;
        return this.legajo == otro.legajo;     // igualdad por legajo
    }

    @Override
    public int hashCode() {
        return Integer.hashCode(legajo);
    }
}

// Uso
Estudiante e1 = new Estudiante("Ana", "García", "123", "a@b.com", 1001, "Sistemas");
Estudiante e2 = new Estudiante("Ana", "García", "123", "a@b.com", 1001, "Sistemas");
Estudiante e3 = new Estudiante("Luis", "Pérez", "456", "l@b.com", 1002, "Redes");

System.out.println(e1.equals(e2)); // true (mismo legajo)
System.out.println(e1.equals(e3)); // false (distinto legajo)
System.out.println(e1 == e2);      // false (distintos objetos en memoria)
```

---

## `final` en herencia

```java
// Clase final — no puede ser heredada
public final class NumeroDNI {
    private final String valor;
    public NumeroDNI(String valor) { this.valor = valor; }
    // ...
}

// Método final — no puede ser sobreescrito en subclases
public class Persona {
    public final String getDni() {
        return dni; // el DNI no debería cambiar su comportamiento
    }
}
```

---

## Ejemplo completo — Jerarquía de Personal Escolar

```java
// PersonalEscolar.java
public abstract class PersonalEscolar extends Persona {
    private String codigoEmpleado;
    private java.time.LocalDate fechaIngreso;

    public PersonalEscolar(String nombre, String apellido, String dni,
                           String email, String codigoEmpleado) {
        super(nombre, apellido, dni, email);
        this.codigoEmpleado = codigoEmpleado;
        this.fechaIngreso = java.time.LocalDate.now();
    }

    public String getCodigoEmpleado() { return codigoEmpleado; }
    public java.time.LocalDate getFechaIngreso() { return fechaIngreso; }

    // Método abstracto: cada tipo de empleado lo implementa diferente
    public abstract double calcularSalario();
}

// ProfesorTiempoCompleto.java
public class ProfesorTiempoCompleto extends PersonalEscolar {
    private double salarioBase;
    private int añosAntiguedad;

    public ProfesorTiempoCompleto(String nombre, String apellido, String dni,
                                   String email, double salarioBase) {
        super(nombre, apellido, dni, email, "PT-" + dni.substring(0, 4));
        this.salarioBase = salarioBase;
        this.añosAntiguedad = 0;
    }

    @Override
    public double calcularSalario() {
        double adicionalAntiguedad = salarioBase * (añosAntiguedad * 0.02);
        return salarioBase + adicionalAntiguedad;
    }
}

// ProfesorHorasExtra.java
public class ProfesorHorasExtra extends PersonalEscolar {
    private double valorHora;
    private int horasTrabajadas;

    public ProfesorHorasExtra(String nombre, String apellido, String dni,
                               String email, double valorHora) {
        super(nombre, apellido, dni, email, "PH-" + dni.substring(0, 4));
        this.valorHora = valorHora;
        this.horasTrabajadas = 0;
    }

    public void registrarHoras(int horas) {
        this.horasTrabajadas += horas;
    }

    @Override
    public double calcularSalario() {
        return valorHora * horasTrabajadas;
    }
}
```

---

## Resumen

| Concepto | Descripción |
|----------|-------------|
| `extends` | Indica herencia de una clase |
| `super()` | Llama al constructor o método del padre |
| `@Override` | Marca que se sobreescribe un método del padre |
| Polimorfismo | Tratar distintas subclases como el tipo padre |
| `instanceof` | Verificar el tipo real de un objeto |
| Upcasting | Asignar subclase a variable del tipo padre (automático) |
| Downcasting | Convertir de tipo padre al tipo hijo (explícito, puede fallar) |

**Siguiente:** [2.4 Interfaces y Clases Abstractas](04-interfaces-abstractas.md)
