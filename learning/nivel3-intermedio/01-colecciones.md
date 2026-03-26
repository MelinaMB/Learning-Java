# 3.1 Colecciones

El **Java Collections Framework** provee estructuras de datos listas para usar.
A diferencia de los arrays, las colecciones tienen tamaño dinámico.

## Jerarquía de colecciones

```
Iterable
  └── Collection
        ├── List          (orden + duplicados)
        │     ├── ArrayList
        │     └── LinkedList
        ├── Set           (sin duplicados)
        │     ├── HashSet
        │     ├── LinkedHashSet
        │     └── TreeSet
        └── Queue
              ├── LinkedList
              └── PriorityQueue

Map  (pares clave-valor)
  ├── HashMap
  ├── LinkedHashMap
  └── TreeMap
```

---

## List — Colección Ordenada

### ArrayList

```java
import java.util.ArrayList;
import java.util.List;

List<String> estudiantes = new ArrayList<>();

// Agregar
estudiantes.add("Ana García");
estudiantes.add("Luis Pérez");
estudiantes.add("Carlos Ruiz");
estudiantes.add(1, "Marta López");  // inserta en posición 1

System.out.println(estudiantes);
// [Ana García, Marta López, Luis Pérez, Carlos Ruiz]

// Acceder
System.out.println(estudiantes.get(0));     // Ana García
System.out.println(estudiantes.size());     // 4

// Modificar
estudiantes.set(0, "Ana María García");

// Buscar
System.out.println(estudiantes.contains("Luis Pérez"));  // true
System.out.println(estudiantes.indexOf("Luis Pérez"));   // 2

// Eliminar
estudiantes.remove("Carlos Ruiz");          // por valor
estudiantes.remove(0);                       // por índice

// Recorrer
for (String nombre : estudiantes) {
    System.out.println(nombre);
}

// Con índice
for (int i = 0; i < estudiantes.size(); i++) {
    System.out.println(i + ": " + estudiantes.get(i));
}
```

### Trabajar con objetos en List

```java
import java.util.ArrayList;
import java.util.List;

public class GestionEstudiantes {
    private List<Estudiante> estudiantes = new ArrayList<>();

    public void inscribir(Estudiante e) {
        estudiantes.add(e);
        System.out.println(e.getNombreCompleto() + " inscrito.");
    }

    public boolean darDeBaja(int legajo) {
        return estudiantes.removeIf(e -> e.getLegajo() == legajo);
    }

    public Estudiante buscarPorLegajo(int legajo) {
        for (Estudiante e : estudiantes) {
            if (e.getLegajo() == legajo) return e;
        }
        return null;
    }

    public List<Estudiante> obtenerAprobados() {
        List<Estudiante> aprobados = new ArrayList<>();
        for (Estudiante e : estudiantes) {
            if (e.getPromedio() >= 6.0) aprobados.add(e);
        }
        return aprobados;
    }

    public int getCantidad() { return estudiantes.size(); }
}
```

---

## Set — Sin Duplicados

```java
import java.util.HashSet;
import java.util.LinkedHashSet;
import java.util.TreeSet;
import java.util.Set;

// HashSet — sin orden garantizado, más rápido
Set<String> materias = new HashSet<>();
materias.add("Matemática");
materias.add("Física");
materias.add("Matemática"); // ignorado — duplicado
System.out.println(materias.size()); // 2

// LinkedHashSet — mantiene el orden de inserción
Set<String> carreras = new LinkedHashSet<>();
carreras.add("Sistemas");
carreras.add("Redes");
carreras.add("Programación");
System.out.println(carreras); // [Sistemas, Redes, Programación]

// TreeSet — orden natural (alfabético para Strings)
Set<Integer> legajos = new TreeSet<>();
legajos.add(1003);
legajos.add(1001);
legajos.add(1002);
System.out.println(legajos); // [1001, 1002, 1003]

// Operaciones de conjuntos
Set<String> grupoA = new HashSet<>(Set.of("Ana", "Luis", "Carlos"));
Set<String> grupoB = new HashSet<>(Set.of("Carlos", "Marta", "Pedro"));

// Intersección
Set<String> enAmbos = new HashSet<>(grupoA);
enAmbos.retainAll(grupoB);
System.out.println("En ambos: " + enAmbos); // [Carlos]

// Unión
Set<String> todos = new HashSet<>(grupoA);
todos.addAll(grupoB);
System.out.println("Todos: " + todos);

// Diferencia
Set<String> soloEnA = new HashSet<>(grupoA);
soloEnA.removeAll(grupoB);
System.out.println("Solo en A: " + soloEnA); // [Ana, Luis]
```

> Para que un Set de objetos personalizados detecte duplicados correctamente,
> la clase debe sobreescribir `equals()` y `hashCode()`.

---

## Map — Clave-Valor

```java
import java.util.HashMap;
import java.util.LinkedHashMap;
import java.util.TreeMap;
import java.util.Map;

// HashMap — acceso O(1), sin orden garantizado
Map<Integer, String> legajoNombre = new HashMap<>();
legajoNombre.put(1001, "Ana García");
legajoNombre.put(1002, "Luis Pérez");
legajoNombre.put(1003, "Carlos Ruiz");

// Acceder
System.out.println(legajoNombre.get(1001));           // Ana García
System.out.println(legajoNombre.get(9999));           // null
System.out.println(legajoNombre.getOrDefault(9999, "No encontrado")); // No encontrado

// Verificar existencia
System.out.println(legajoNombre.containsKey(1002));   // true
System.out.println(legajoNombre.containsValue("Ana García")); // true

// Eliminar
legajoNombre.remove(1003);

// Iterar
for (Map.Entry<Integer, String> entry : legajoNombre.entrySet()) {
    System.out.println(entry.getKey() + " -> " + entry.getValue());
}

// Solo claves o valores
for (Integer legajo : legajoNombre.keySet()) {
    System.out.println("Legajo: " + legajo);
}
for (String nombre : legajoNombre.values()) {
    System.out.println("Nombre: " + nombre);
}

// putIfAbsent — solo agrega si la clave no existe
legajoNombre.putIfAbsent(1001, "Otro nombre"); // no hace nada, ya existe
legajoNombre.putIfAbsent(1004, "Marta López"); // agrega

// computeIfAbsent — calcula y agrega si no existe
Map<String, List<String>> cursoEstudiantes = new HashMap<>();
cursoEstudiantes.computeIfAbsent("Sistemas", k -> new ArrayList<>())
                .add("Ana García");
cursoEstudiantes.computeIfAbsent("Sistemas", k -> new ArrayList<>())
                .add("Luis Pérez");
System.out.println(cursoEstudiantes); // {Sistemas=[Ana García, Luis Pérez]}
```

### Mapa de notas por estudiante

```java
Map<Integer, List<Double>> notasPorEstudiante = new HashMap<>();

// Agregar notas
int legajo = 1001;
notasPorEstudiante.computeIfAbsent(legajo, k -> new ArrayList<>()).add(8.5);
notasPorEstudiante.computeIfAbsent(legajo, k -> new ArrayList<>()).add(7.0);
notasPorEstudiante.computeIfAbsent(legajo, k -> new ArrayList<>()).add(9.0);

// Calcular promedio
List<Double> notas = notasPorEstudiante.get(legajo);
double promedio = notas.stream().mapToDouble(Double::doubleValue).average().orElse(0);
System.out.printf("Promedio de %d: %.2f%n", legajo, promedio);
```

---

## Queue — Colas

```java
import java.util.LinkedList;
import java.util.PriorityQueue;
import java.util.Queue;

// Cola FIFO — primero en entrar, primero en salir
Queue<String> colaInscripcion = new LinkedList<>();
colaInscripcion.offer("Ana");    // agrega al final
colaInscripcion.offer("Luis");
colaInscripcion.offer("Carlos");

System.out.println(colaInscripcion.peek());   // "Ana" — ver sin quitar
System.out.println(colaInscripcion.poll());   // "Ana" — quitar el primero
System.out.println(colaInscripcion.size());   // 2

// PriorityQueue — procesa por prioridad (orden natural o comparador)
PriorityQueue<Double> notasOrdenadas = new PriorityQueue<>();
notasOrdenadas.offer(7.5);
notasOrdenadas.offer(9.0);
notasOrdenadas.offer(6.5);
notasOrdenadas.offer(8.0);

// Procesa de menor a mayor
while (!notasOrdenadas.isEmpty()) {
    System.out.println(notasOrdenadas.poll()); // 6.5, 7.5, 8.0, 9.0
}
```

---

## Clase utilitaria Collections

```java
import java.util.Collections;
import java.util.List;
import java.util.ArrayList;

List<Double> notas = new ArrayList<>(List.of(7.5, 9.0, 6.5, 8.0, 5.5));

Collections.sort(notas);                    // ordena ascendente
Collections.sort(notas, Collections.reverseOrder()); // descendente
Collections.shuffle(notas);                // orden aleatorio
Collections.reverse(notas);               // invierte

System.out.println(Collections.min(notas)); // mínimo
System.out.println(Collections.max(notas)); // máximo
System.out.println(Collections.frequency(notas, 7.5)); // cuántas veces aparece

// Lista inmutable
List<String> inmutable = Collections.unmodifiableList(notas2);
// inmutable.add("x"); // lanza UnsupportedOperationException

// Lista sincronizada (thread-safe básico)
List<String> syncList = Collections.synchronizedList(new ArrayList<>());
```

---

## Colecciones inmutables (Java 9+)

```java
// List.of — inmutable
List<String> materias = List.of("Matemática", "Física", "Historia");
// materias.add("Química"); // UnsupportedOperationException

// Set.of — inmutable, sin duplicados
Set<String> turnos = Set.of("Mañana", "Tarde", "Noche");

// Map.of — inmutable
Map<String, Integer> horasPorMateria = Map.of(
    "Matemática", 6,
    "Física", 4,
    "Historia", 3
);

// Map.copyOf — copia inmutable de un mapa existente
Map<String, Integer> copiaInmutable = Map.copyOf(horasPorMateria);
```

---

## Resumen

| Colección | Características | Cuándo usar |
|-----------|----------------|-------------|
| `ArrayList` | Acceso rápido por índice, lento para insertar en medio | Lista ordenada de uso general |
| `LinkedList` | Inserción/eliminación rápida al inicio/fin | Colas, deques |
| `HashSet` | Sin duplicados, sin orden, O(1) | Eliminar duplicados, verificar existencia |
| `LinkedHashSet` | Sin duplicados, mantiene orden de inserción | Igual que HashSet pero con orden |
| `TreeSet` | Sin duplicados, orden natural | Datos ordenados sin duplicados |
| `HashMap` | Acceso O(1) por clave, sin orden | Búsqueda rápida por clave |
| `LinkedHashMap` | Orden de inserción | Caché con acceso por clave |
| `TreeMap` | Claves ordenadas | Datos ordenados por clave |

**Siguiente:** [3.2 Generics](02-generics.md)
