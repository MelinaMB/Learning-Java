# 3.2 Generics (Tipos Genéricos)

Los generics permiten escribir código que funciona con **cualquier tipo de dato**
de manera segura en tiempo de compilación, evitando casts y errores en tiempo de ejecución.

## ¿Por qué Generics?

```java
// Sin generics (Java antiguo) — problemático
List listaAntigua = new ArrayList();
listaAntigua.add("Ana");
listaAntigua.add(1001);         // mezcla tipos — sin error de compilación
listaAntigua.add(new Object());

String nombre = (String) listaAntigua.get(0); // cast manual
String error = (String) listaAntigua.get(1);  // ClassCastException en runtime!

// Con generics — seguro en tiempo de compilación
List<String> listaNueva = new ArrayList<>();
listaNueva.add("Ana");
// listaNueva.add(1001);        // ERROR de compilación — correcto!
String nombreSeguro = listaNueva.get(0); // sin cast necesario
```

---

## Clases Genéricas

```java
// T es el "type parameter" — convención de nombres:
// T = Type, E = Element, K = Key, V = Value, N = Number

public class Repositorio<T> {
    private List<T> elementos = new ArrayList<>();

    public void agregar(T elemento) {
        elementos.add(elemento);
    }

    public T obtener(int indice) {
        return elementos.get(indice);
    }

    public int cantidad() {
        return elementos.size();
    }

    public boolean contiene(T elemento) {
        return elementos.contains(elemento);
    }

    public List<T> obtenerTodos() {
        return Collections.unmodifiableList(elementos);
    }

    @Override
    public String toString() {
        return "Repositorio" + elementos.toString();
    }
}

// Uso con diferentes tipos
Repositorio<Estudiante> repoEstudiantes = new Repositorio<>();
repoEstudiantes.agregar(new Estudiante("Ana", "García", "123", "a@b.com", 1001, "Sistemas"));
repoEstudiantes.agregar(new Estudiante("Luis", "Pérez", "456", "l@b.com", 1002, "Redes"));

Estudiante primero = repoEstudiantes.obtener(0); // sin cast!

Repositorio<Curso> repoCursos = new Repositorio<>();
Repositorio<Double> repoNotas = new Repositorio<>();
```

---

## Múltiples parámetros de tipo

```java
public class Par<K, V> {
    private final K clave;
    private final V valor;

    public Par(K clave, V valor) {
        this.clave = clave;
        this.valor = valor;
    }

    public K getClave() { return clave; }
    public V getValor() { return valor; }

    @Override
    public String toString() {
        return "(" + clave + ", " + valor + ")";
    }
}

// Usar con distintos tipos
Par<Integer, String> legajoNombre = new Par<>(1001, "Ana García");
Par<String, Double> materiaPromedio = new Par<>("Matemática", 8.5);
Par<Estudiante, List<Calificacion>> registro = new Par<>(estudiante, calificaciones);

System.out.println(legajoNombre);     // (1001, Ana García)
System.out.println(materiaPromedio);  // (Matemática, 8.5)
```

---

## Métodos Genéricos

```java
public class UtilColecciones {

    // Método genérico — <T> antes del tipo de retorno
    public static <T> void imprimir(List<T> lista) {
        for (T elemento : lista) {
            System.out.println(elemento);
        }
    }

    // Retorna el primero que cumpla una condición
    public static <T> T buscarPrimero(List<T> lista, java.util.function.Predicate<T> condicion) {
        for (T elemento : lista) {
            if (condicion.test(elemento)) return elemento;
        }
        return null;
    }

    // Convierte lista de un tipo a otro
    public static <T, R> List<R> transformar(List<T> lista,
                                              java.util.function.Function<T, R> transformador) {
        List<R> resultado = new ArrayList<>();
        for (T elemento : lista) {
            resultado.add(transformador.apply(elemento));
        }
        return resultado;
    }
}

// Uso
List<Estudiante> estudiantes = List.of(
    new Estudiante("Ana", "García", "1", "a@b.com", 1001, "Sistemas"),
    new Estudiante("Luis", "Pérez", "2", "l@b.com", 1002, "Redes")
);

UtilColecciones.imprimir(estudiantes);

Estudiante encontrado = UtilColecciones.buscarPrimero(
    estudiantes,
    e -> e.getLegajo() == 1002
);

List<String> nombres = UtilColecciones.transformar(
    estudiantes,
    Estudiante::getNombreCompleto
);
System.out.println(nombres); // [Ana García, Luis Pérez]
```

---

## Tipos Acotados (Bounded Types)

```java
// <T extends Number> — T debe ser Number o una subclase de Number
public static <T extends Number> double sumar(List<T> lista) {
    double suma = 0;
    for (T elemento : lista) {
        suma += elemento.doubleValue();
    }
    return suma;
}

// Uso
List<Integer> enteros = List.of(1, 2, 3, 4, 5);
List<Double> decimales = List.of(1.5, 2.5, 3.5);

System.out.println(sumar(enteros));   // 15.0
System.out.println(sumar(decimales)); // 7.5
// sumar(List.of("hola")); // ERROR: String no extiende Number

// Múltiples bounds
public static <T extends Comparable<T> & Cloneable> T maximo(T a, T b) {
    return a.compareTo(b) >= 0 ? a : b;
}
```

### Repositorio con tipo acotado

```java
// Solo acepta entidades que tengan ID
public interface Identificable {
    int getId();
}

public class RepositorioBase<T extends Identificable> {
    private Map<Integer, T> almacen = new HashMap<>();

    public void guardar(T entidad) {
        almacen.put(entidad.getId(), entidad);
    }

    public T buscarPorId(int id) {
        return almacen.get(id);
    }

    public List<T> obtenerTodos() {
        return new ArrayList<>(almacen.values());
    }

    public boolean eliminar(int id) {
        return almacen.remove(id) != null;
    }

    public int cantidad() { return almacen.size(); }
}

// Uso
public class Estudiante implements Identificable {
    private int id;
    // ...
    @Override
    public int getId() { return id; }
}

RepositorioBase<Estudiante> repo = new RepositorioBase<>();
repo.guardar(new Estudiante(...));
Estudiante e = repo.buscarPorId(1001);
```

---

## Wildcards (Comodines)

Los wildcards se usan cuando el tipo exacto no importa o para mayor flexibilidad.

```java
// ? — cualquier tipo
public static void mostrarLista(List<?> lista) {
    for (Object elemento : lista) {
        System.out.println(elemento);
    }
}

// ? extends T — tipo desconocido que extiende T (upper bound)
// "puede leer, no puede escribir"
public static double calcularPromedioNotas(List<? extends Number> notas) {
    double suma = 0;
    for (Number n : notas) suma += n.doubleValue();
    return suma / notas.size();
}

// ? super T — tipo desconocido que es T o padre de T (lower bound)
// "puede escribir, lectura limitada"
public static void agregarEstudiantes(List<? super Estudiante> lista) {
    lista.add(new Estudiante("Test", "Test", "999", "t@t.com", 9999, "Test"));
}

// Ejemplo práctico
List<Integer> enteros = List.of(8, 7, 9, 6, 8);
List<Double> decimales = List.of(8.5, 7.0, 9.2, 6.5);

System.out.println(calcularPromedioNotas(enteros));   // 7.6
System.out.println(calcularPromedioNotas(decimales)); // 7.8

List<Persona> personas = new ArrayList<>();
agregarEstudiantes(personas); // funciona: Persona es super de Estudiante
```

> **Regla PECS:** Producer Extends, Consumer Super.
> Si la colección *produce* (lees de ella), usa `? extends T`.
> Si la colección *consume* (escribes en ella), usa `? super T`.

---

## Resumen

| Concepto | Sintaxis | Descripción |
|----------|----------|-------------|
| Tipo genérico | `class Caja<T>` | Clase que trabaja con cualquier tipo |
| Método genérico | `<T> void imprimir(T t)` | Método que trabaja con cualquier tipo |
| Upper bound | `<T extends Animal>` | T debe ser Animal o subclase |
| Wildcard | `List<?>` | Lista de tipo desconocido |
| Wildcard upper | `List<? extends Number>` | Lista de Number o subtipo |
| Wildcard lower | `List<? super Integer>` | Lista de Integer o supertipo |

**Siguiente:** [3.3 Excepciones](03-excepciones.md)
