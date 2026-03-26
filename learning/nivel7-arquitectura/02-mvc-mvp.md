# 7.2 MVC y MVP

## MVC — Model-View-Controller

MVC es uno de los patrones de arquitectura más conocidos. Separa la aplicación
en tres componentes con responsabilidades distintas.

```
  Usuario
    |
    v
CONTROLLER  ----modifica---->  MODEL
    |                            |
    |                            v
    +----------actualiza------> VIEW
                                 |
                                 v
                              Usuario
```

| Componente | Responsabilidad |
|-----------|----------------|
| **Model** | Datos y lógica de negocio. No sabe nada de la UI. |
| **View** | Presentación. Solo muestra datos, no los procesa. |
| **Controller** | Intermediario. Recibe input del usuario, actualiza el modelo, selecciona la vista. |

---

### Implementación MVC — Sistema Escolar

```java
// MODEL — datos y lógica de negocio
public class EstudianteModel {
    private List<Estudiante> estudiantes = new ArrayList<>();

    public void agregar(Estudiante e) { estudiantes.add(e); }

    public Optional<Estudiante> buscarPorLegajo(int legajo) {
        return estudiantes.stream()
            .filter(e -> e.getLegajo() == legajo)
            .findFirst();
    }

    public List<Estudiante> listarTodos() {
        return Collections.unmodifiableList(estudiantes);
    }

    public List<Estudiante> listarAprobados() {
        return estudiantes.stream()
            .filter(e -> e.getPromedio() >= 6.0)
            .collect(Collectors.toList());
    }

    public double promedioGeneral() {
        return estudiantes.stream()
            .mapToDouble(Estudiante::getPromedio)
            .average().orElse(0);
    }
}

// VIEW — presentación en consola
public class EstudianteView {

    public void mostrarEstudiante(Estudiante e) {
        System.out.println("================================");
        System.out.println("Legajo:   " + e.getLegajo());
        System.out.println("Nombre:   " + e.getNombreCompleto());
        System.out.println("Carrera:  " + e.getCarrera());
        System.out.printf("Promedio: %.2f%n", e.getPromedio());
        System.out.println("Estado:   " + (e.getPromedio() >= 6 ? "Aprobado" : "Reprobado"));
        System.out.println("================================");
    }

    public void mostrarLista(List<Estudiante> lista, String titulo) {
        System.out.println("\n=== " + titulo + " ===");
        if (lista.isEmpty()) {
            System.out.println("  (sin resultados)");
            return;
        }
        System.out.printf("%-8s %-25s %-15s %s%n", "Legajo", "Nombre", "Carrera", "Promedio");
        System.out.println("-".repeat(60));
        for (Estudiante e : lista) {
            System.out.printf("%-8d %-25s %-15s %.2f%n",
                e.getLegajo(), e.getNombreCompleto(), e.getCarrera(), e.getPromedio());
        }
        System.out.println("-".repeat(60));
        System.out.println("Total: " + lista.size());
    }

    public void mostrarError(String mensaje) {
        System.out.println("[ERROR] " + mensaje);
    }

    public void mostrarExito(String mensaje) {
        System.out.println("[OK] " + mensaje);
    }

    public void mostrarEstadisticas(int total, int aprobados, double promedio) {
        System.out.println("\n--- Estadísticas ---");
        System.out.println("Total estudiantes: " + total);
        System.out.println("Aprobados: " + aprobados + "/" + total);
        System.out.printf("Promedio general: %.2f%n", promedio);
    }
}

// CONTROLLER — intermediario
public class EstudianteController {
    private final EstudianteModel modelo;
    private final EstudianteView vista;

    public EstudianteController(EstudianteModel modelo, EstudianteView vista) {
        this.modelo = modelo;
        this.vista = vista;
    }

    public void registrarEstudiante(String nombre, String apellido,
                                     String email, String carrera, double promedio) {
        try {
            if (nombre.isBlank() || apellido.isBlank())
                throw new IllegalArgumentException("Nombre y apellido son obligatorios.");

            Estudiante nuevo = new Estudiante(nombre, apellido, "dni", email, 0, carrera);
            // nuevo.setPromedio(promedio);
            modelo.agregar(nuevo);
            vista.mostrarExito("Estudiante registrado: " + nuevo.getNombreCompleto());
        } catch (IllegalArgumentException e) {
            vista.mostrarError(e.getMessage());
        }
    }

    public void buscarEstudiante(int legajo) {
        modelo.buscarPorLegajo(legajo)
            .ifPresentOrElse(
                vista::mostrarEstudiante,
                () -> vista.mostrarError("Estudiante con legajo " + legajo + " no encontrado.")
            );
    }

    public void listarTodos() {
        vista.mostrarLista(modelo.listarTodos(), "Todos los Estudiantes");
    }

    public void listarAprobados() {
        vista.mostrarLista(modelo.listarAprobados(), "Estudiantes Aprobados");
    }

    public void mostrarEstadisticas() {
        List<Estudiante> todos = modelo.listarTodos();
        int aprobados = (int) todos.stream().filter(e -> e.getPromedio() >= 6).count();
        vista.mostrarEstadisticas(todos.size(), aprobados, modelo.promedioGeneral());
    }
}

// Punto de entrada
public class AplicacionMVC {
    public static void main(String[] args) {
        EstudianteModel modelo = new EstudianteModel();
        EstudianteView vista = new EstudianteView();
        EstudianteController controlador = new EstudianteController(modelo, vista);

        // Simular interacciones del usuario
        controlador.registrarEstudiante("Ana", "García", "ana@mail.com", "Sistemas", 8.5);
        controlador.registrarEstudiante("Luis", "Pérez", "luis@mail.com", "Redes", 5.0);
        controlador.registrarEstudiante("Carlos", "Ruiz", "carlos@mail.com", "Sistemas", 7.5);

        controlador.listarTodos();
        controlador.listarAprobados();
        controlador.mostrarEstadisticas();
    }
}
```

---

## MVP — Model-View-Presenter

MVP es una variación de MVC donde:
- La **Vista** es completamente pasiva (no tiene lógica alguna)
- El **Presenter** maneja toda la lógica de presentación
- El Presenter y la Vista se comunican a través de interfaces

```java
// Interfaz que la Vista debe implementar
public interface EstudianteVista {
    void mostrarEstudiante(String nombre, String legajo, String promedio, String estado);
    void mostrarError(String mensaje);
    void mostrarListaEstudiantes(List<String[]> filas);
}

// Presenter — toda la lógica de presentación
public class EstudiantePresenter {
    private final EstudianteModel modelo;
    private final EstudianteVista vista;

    public EstudiantePresenter(EstudianteModel modelo, EstudianteVista vista) {
        this.modelo = modelo;
        this.vista = vista;
    }

    public void cargarEstudiante(int legajo) {
        modelo.buscarPorLegajo(legajo).ifPresentOrElse(
            e -> vista.mostrarEstudiante(
                e.getNombreCompleto(),
                String.valueOf(e.getLegajo()),
                String.format("%.2f", e.getPromedio()),
                e.getPromedio() >= 6 ? "Aprobado" : "Reprobado"
            ),
            () -> vista.mostrarError("Estudiante no encontrado: " + legajo)
        );
    }
}
```

---

## MVC vs MVP

| Aspecto | MVC | MVP |
|---------|-----|-----|
| Vista | Puede tener algo de lógica | Completamente pasiva |
| Controller/Presenter | Selecciona la vista | Actualiza directamente la vista |
| Testabilidad | Media | Alta (Presenter sin dependencia de UI) |
| Uso típico | Spring MVC, backends web | Android, desktop apps |

**Siguiente:** [7.3 Arquitectura Hexagonal y DDD](03-hexagonal-ddd.md)
