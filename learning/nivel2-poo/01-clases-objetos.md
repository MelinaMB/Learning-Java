# 2.1 Clases y Objetos

## ¿Qué es una clase?

Una clase es un **molde o plantilla** que define la estructura y el comportamiento
de un tipo de objeto. Es como el plano de una casa: el plano en sí no es una casa,
pero describe cómo construir todas las casas del mismo tipo.

## ¿Qué es un objeto?

Un objeto es una **instancia concreta** de una clase. Si la clase es el plano,
el objeto es la casa construida a partir de ese plano.

```
Clase Estudiante (el molde)
        |
        |--- new Estudiante() ---> objeto ana  (Ana García, legajo 1001)
        |--- new Estudiante() ---> objeto luis (Luis Pérez, legajo 1002)
        |--- new Estudiante() ---> objeto carlos (Carlos Ruiz, legajo 1003)
```

---

## Definir una clase

```java
// Archivo: Estudiante.java
public class Estudiante {

    // ATRIBUTOS (estado del objeto)
    String nombre;
    String apellido;
    int legajo;
    double promedio;
    boolean activo;

    // MÉTODOS (comportamiento del objeto)
    public String getNombreCompleto() {
        return nombre + " " + apellido;
    }

    public boolean aprobo() {
        return promedio >= 6.0;
    }

    public void mostrarInfo() {
        System.out.println("=== Estudiante ===");
        System.out.println("Nombre:   " + getNombreCompleto());
        System.out.println("Legajo:   " + legajo);
        System.out.printf("Promedio: %.2f%n", promedio);
        System.out.println("Estado:   " + (aprobo() ? "Aprobado" : "Reprobado"));
    }
}
```

---

## Crear objetos con `new`

```java
public class Main {
    public static void main(String[] args) {
        // Crear un objeto (instanciar la clase)
        Estudiante ana = new Estudiante();

        // Asignar valores a los atributos
        ana.nombre = "Ana";
        ana.apellido = "García";
        ana.legajo = 1001;
        ana.promedio = 8.75;
        ana.activo = true;

        // Llamar métodos del objeto
        System.out.println(ana.getNombreCompleto()); // Ana García
        System.out.println(ana.aprobo());            // true
        ana.mostrarInfo();

        // Crear otro objeto independiente
        Estudiante luis = new Estudiante();
        luis.nombre = "Luis";
        luis.apellido = "Pérez";
        luis.legajo = 1002;
        luis.promedio = 5.5;
        luis.activo = true;

        luis.mostrarInfo(); // datos de Luis, no de Ana
    }
}
```

---

## Constructores

Un constructor es un método especial que se ejecuta **automáticamente** al crear
un objeto con `new`. Sirve para inicializar el objeto con valores iniciales.

```java
public class Estudiante {

    String nombre;
    String apellido;
    int legajo;
    double promedio;
    boolean activo;

    // Constructor por defecto (sin parámetros)
    public Estudiante() {
        this.activo = true;
        this.promedio = 0.0;
    }

    // Constructor con parámetros
    public Estudiante(String nombre, String apellido, int legajo) {
        this.nombre = nombre;
        this.apellido = apellido;
        this.legajo = legajo;
        this.activo = true;
        this.promedio = 0.0;
    }

    // Constructor completo
    public Estudiante(String nombre, String apellido, int legajo, double promedio) {
        this.nombre = nombre;
        this.apellido = apellido;
        this.legajo = legajo;
        this.promedio = promedio;
        this.activo = true;
    }
}
```

### Usar los constructores

```java
// Constructor por defecto
Estudiante e1 = new Estudiante();
e1.nombre = "Ana";
e1.legajo = 1001;

// Constructor con parámetros
Estudiante e2 = new Estudiante("Luis", "Pérez", 1002);

// Constructor completo
Estudiante e3 = new Estudiante("Carlos", "Ruiz", 1003, 8.5);
```

---

## La palabra clave `this`

`this` hace referencia al **objeto actual** (la instancia en la que se está ejecutando el método).

```java
public class Curso {
    String nombre;
    int capacidadMaxima;
    int inscriptos;

    public Curso(String nombre, int capacidadMaxima) {
        // "this.nombre" = el atributo del objeto
        // "nombre" = el parámetro del constructor
        this.nombre = nombre;
        this.capacidadMaxima = capacidadMaxima;
        this.inscriptos = 0;
    }

    // this también puede llamar a otro constructor de la misma clase
    public Curso(String nombre) {
        this(nombre, 30); // llama al constructor de arriba con capacidad=30
    }

    public boolean estaLleno() {
        return this.inscriptos >= this.capacidadMaxima;
    }

    public boolean inscribir() {
        if (estaLleno()) {
            System.out.println("El curso " + this.nombre + " está lleno.");
            return false;
        }
        this.inscriptos++;
        return true;
    }

    @Override
    public String toString() {
        return String.format("Curso{nombre='%s', inscriptos=%d/%d}",
            nombre, inscriptos, capacidadMaxima);
    }
}
```

---

## El método `toString()`

`toString()` define cómo se representa el objeto como texto.
Java lo llama automáticamente cuando concatenas un objeto con String.

```java
public class Estudiante {
    // ... atributos y métodos anteriores ...

    @Override
    public String toString() {
        return String.format("Estudiante{legajo=%d, nombre='%s %s', promedio=%.2f}",
            legajo, nombre, apellido, promedio);
    }
}

// Uso
Estudiante ana = new Estudiante("Ana", "García", 1001, 8.75);
System.out.println(ana);  // Llama automáticamente a toString()
// Imprime: Estudiante{legajo=1001, nombre='Ana García', promedio=8.75}

String texto = "El estudiante es: " + ana; // también llama toString()
```

---

## `equals()` y `hashCode()`

`equals()` compara si dos objetos representan lo mismo. `hashCode()` ayuda a que Java los encuentre bien en estructuras como `HashSet` o `HashMap`.

```java
public class Estudiante {
    private int legajo;
    private String nombre;

    public Estudiante(int legajo, String nombre) {
        this.legajo = legajo;
        this.nombre = nombre;
    }

    @Override
    public boolean equals(Object obj) {
        if (this == obj) return true;
        if (!(obj instanceof Estudiante)) return false;
        Estudiante otro = (Estudiante) obj;
        return legajo == otro.legajo;
    }

    @Override
    public int hashCode() {
        return Integer.hashCode(legajo);
    }
}
```

> **Idea clave:** si dos objetos deben considerarse iguales por su contenido, conviene sobrescribir ambos métodos juntos.

---

## Atributos y métodos estáticos

Los miembros `static` pertenecen a la **clase**, no a los objetos individuales.

```java
public class Estudiante {
    // Atributo estático — compartido por todos los objetos
    private static int contadorEstudiantes = 0;
    public static final double NOTA_APROBACION = 6.0;

    private String nombre;
    private int legajo;

    public Estudiante(String nombre) {
        contadorEstudiantes++;           // se incrementa con cada nuevo objeto
        this.legajo = contadorEstudiantes;
        this.nombre = nombre;
    }

    // Método estático — se llama sobre la clase, no sobre un objeto
    public static int getCantidadEstudiantes() {
        return contadorEstudiantes;
    }

    public static void main(String[] args) {
        System.out.println(Estudiante.getCantidadEstudiantes()); // 0

        Estudiante a = new Estudiante("Ana");
        Estudiante b = new Estudiante("Luis");
        Estudiante c = new Estudiante("Carlos");

        System.out.println(Estudiante.getCantidadEstudiantes()); // 3
        System.out.println("Nota de aprobación: " + Estudiante.NOTA_APROBACION);
    }
}
```

---

## Ejemplo integrador — Clase Profesor

```java
public class Profesor {
    private static int ultimoId = 1000;

    private int id;
    private String nombre;
    private String apellido;
    private String especialidad;
    private double salario;

    public Profesor(String nombre, String apellido, String especialidad) {
        this.id = ++ultimoId;
        this.nombre = nombre;
        this.apellido = apellido;
        this.especialidad = especialidad;
        this.salario = 50000.0; // salario base
    }

    public String getNombreCompleto() {
        return "Prof. " + nombre + " " + apellido;
    }

    public void aplicarAumento(double porcentaje) {
        if (porcentaje < 0 || porcentaje > 100) {
            System.out.println("Porcentaje inválido: " + porcentaje);
            return;
        }
        double aumento = salario * (porcentaje / 100.0);
        salario += aumento;
        System.out.printf("%s recibió un aumento del %.0f%% (+$%.2f). Nuevo salario: $%.2f%n",
            getNombreCompleto(), porcentaje, aumento, salario);
    }

    @Override
    public String toString() {
        return String.format("Profesor{id=%d, nombre='%s', especialidad='%s', salario=%.2f}",
            id, getNombreCompleto(), especialidad, salario);
    }

    public static void main(String[] args) {
        Profesor p1 = new Profesor("María", "González", "Matemática");
        Profesor p2 = new Profesor("Roberto", "Sánchez", "Programación");

        System.out.println(p1);
        System.out.println(p2);

        p1.aplicarAumento(10);
        p2.aplicarAumento(15);
    }
}
```

---

## Resumen

| Concepto | Descripción |
|----------|-------------|
| Clase | Plantilla que define atributos y métodos |
| Objeto | Instancia concreta de una clase (`new Clase()`) |
| Constructor | Método especial de inicialización, mismo nombre que la clase |
| `this` | Referencia al objeto actual |
| `static` | Pertenece a la clase, no a instancias |
| `toString()` | Representación textual del objeto |

**Siguiente:** [2.2 Encapsulamiento](02-encapsulamiento.md)
