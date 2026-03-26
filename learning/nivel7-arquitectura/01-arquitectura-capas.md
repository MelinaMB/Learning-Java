# 7.1 Arquitectura de Capas (Layered Architecture)

La arquitectura de capas organiza el sistema en capas horizontales donde
cada capa tiene una responsabilidad específica y solo se comunica con
la capa inmediatamente inferior.

## Las Capas Típicas

```
+------------------------------------------+
|        PRESENTACIÓN / UI                 |   Controllers, Views
|  (Recibe solicitudes, muestra respuestas)|
+------------------------------------------+
                    |
                    v
+------------------------------------------+
|        SERVICIO / NEGOCIO                |   Services, Use Cases
|  (Lógica del dominio y reglas de negocio)|
+------------------------------------------+
                    |
                    v
+------------------------------------------+
|        ACCESO A DATOS                    |   Repositories, DAOs
|  (Lectura/escritura de la persistencia)  |
+------------------------------------------+
                    |
                    v
+------------------------------------------+
|        INFRAESTRUCTURA / MODELO          |   Entities, DB, Files
|  (Base de datos, archivos, API externas) |
+------------------------------------------+
```

---

## Estructura de Paquetes

```
com.escuela/
├── controlador/         → Capa de Presentación
│     ├── EstudianteControlador.java
│     ├── CursoControlador.java
│     └── dto/
│           ├── EstudianteDTO.java
│           └── InscripcionDTO.java
│
├── servicio/            → Capa de Negocio
│     ├── EstudianteServicio.java
│     ├── impl/
│     │     └── EstudianteServicioImpl.java
│     └── InscripcionServicio.java
│
├── repositorio/         → Capa de Acceso a Datos
│     ├── EstudianteRepositorio.java  (interfaz)
│     └── impl/
│           └── EstudianteRepositorioImpl.java
│
├── modelo/              → Entidades del dominio
│     ├── Estudiante.java
│     ├── Curso.java
│     └── Calificacion.java
│
└── excepcion/           → Excepciones personalizadas
      ├── EstudianteNoEncontradoException.java
      └── CursoLlenoException.java
```

---

## Implementación Completa

### Capa Modelo

```java
// modelo/Estudiante.java
public class Estudiante {
    private int legajo;
    private String nombre;
    private String apellido;
    private String email;
    private String carrera;
    private double promedio;
    private boolean activo;

    // Constructor, getters, setters, toString...
}
```

### Capa Repositorio (Acceso a Datos)

```java
// repositorio/EstudianteRepositorio.java (interfaz)
public interface EstudianteRepositorio {
    void guardar(Estudiante estudiante);
    Optional<Estudiante> buscarPorLegajo(int legajo);
    List<Estudiante> buscarPorCarrera(String carrera);
    List<Estudiante> listarTodos();
    void actualizar(Estudiante estudiante);
    boolean eliminar(int legajo);
}

// repositorio/impl/EstudianteRepositorioEnMemoria.java
public class EstudianteRepositorioEnMemoria implements EstudianteRepositorio {
    private final Map<Integer, Estudiante> almacen = new HashMap<>();
    private int contadorLegajo = 1000;

    @Override
    public void guardar(Estudiante e) {
        if (e.getLegajo() == 0) {
            // Asignar legajo automático
            // e.setLegajo(++contadorLegajo);
        }
        almacen.put(e.getLegajo(), e);
    }

    @Override
    public Optional<Estudiante> buscarPorLegajo(int legajo) {
        return Optional.ofNullable(almacen.get(legajo));
    }

    @Override
    public List<Estudiante> buscarPorCarrera(String carrera) {
        return almacen.values().stream()
            .filter(e -> e.getCarrera().equalsIgnoreCase(carrera))
            .collect(Collectors.toList());
    }

    @Override
    public List<Estudiante> listarTodos() {
        return new ArrayList<>(almacen.values());
    }

    @Override
    public void actualizar(Estudiante e) {
        if (!almacen.containsKey(e.getLegajo()))
            throw new RuntimeException("Estudiante no encontrado: " + e.getLegajo());
        almacen.put(e.getLegajo(), e);
    }

    @Override
    public boolean eliminar(int legajo) {
        return almacen.remove(legajo) != null;
    }
}
```

### Capa Servicio (Lógica de Negocio)

```java
// servicio/EstudianteServicio.java
public interface EstudianteServicio {
    Estudiante registrar(String nombre, String apellido, String email, String carrera);
    Optional<Estudiante> buscar(int legajo);
    void actualizarNota(int legajo, double nota);
    List<Estudiante> obtenerAprobados();
    void darDeBaja(int legajo);
}

// servicio/impl/EstudianteServicioImpl.java
public class EstudianteServicioImpl implements EstudianteServicio {
    private final EstudianteRepositorio repositorio;
    private static int contadorLegajo = 1000;
    private static final double NOTA_APROBACION = 6.0;

    public EstudianteServicioImpl(EstudianteRepositorio repositorio) {
        this.repositorio = repositorio;
    }

    @Override
    public Estudiante registrar(String nombre, String apellido, String email, String carrera) {
        validarDatos(nombre, apellido, email);

        Estudiante nuevo = new Estudiante();
        // nuevo.setLegajo(++contadorLegajo);
        // nuevo.setNombre(nombre);
        // ... configurar propiedades

        repositorio.guardar(nuevo);
        System.out.println("Estudiante registrado: " + nuevo.getNombreCompleto());
        return nuevo;
    }

    @Override
    public Optional<Estudiante> buscar(int legajo) {
        return repositorio.buscarPorLegajo(legajo);
    }

    @Override
    public void actualizarNota(int legajo, double nota) {
        if (nota < 0 || nota > 10) throw new IllegalArgumentException("Nota inválida: " + nota);

        Estudiante e = repositorio.buscarPorLegajo(legajo)
            .orElseThrow(() -> new RuntimeException("Estudiante no encontrado: " + legajo));

        // e.setPromedio(nota);
        repositorio.actualizar(e);
    }

    @Override
    public List<Estudiante> obtenerAprobados() {
        return repositorio.listarTodos().stream()
            .filter(e -> e.getPromedio() >= NOTA_APROBACION)
            .collect(Collectors.toList());
    }

    @Override
    public void darDeBaja(int legajo) {
        boolean eliminado = repositorio.eliminar(legajo);
        if (!eliminado) throw new RuntimeException("Estudiante no encontrado: " + legajo);
    }

    private void validarDatos(String nombre, String apellido, String email) {
        if (nombre == null || nombre.isBlank()) throw new IllegalArgumentException("Nombre requerido");
        if (apellido == null || apellido.isBlank()) throw new IllegalArgumentException("Apellido requerido");
        if (email == null || !email.contains("@")) throw new IllegalArgumentException("Email inválido");
    }
}
```

### Capa Controlador (Presentación)

```java
// controlador/EstudianteControlador.java
public class EstudianteControlador {
    private final EstudianteServicio servicio;

    public EstudianteControlador(EstudianteServicio servicio) {
        this.servicio = servicio;
    }

    // En una app real, estos métodos responderían a solicitudes HTTP
    public String registrar(String nombre, String apellido, String email, String carrera) {
        try {
            Estudiante e = servicio.registrar(nombre, apellido, email, carrera);
            return "OK: Estudiante registrado con legajo " + e.getLegajo();
        } catch (IllegalArgumentException ex) {
            return "ERROR: " + ex.getMessage();
        }
    }

    public String obtenerEstudiante(int legajo) {
        return servicio.buscar(legajo)
            .map(e -> "OK: " + e.toString())
            .orElse("NOT_FOUND: No existe estudiante con legajo " + legajo);
    }

    public String listarAprobados() {
        List<Estudiante> aprobados = servicio.obtenerAprobados();
        return "OK: " + aprobados.size() + " aprobados";
    }
}
```

### Ensamblado (Composición raíz)

```java
// Main.java — punto de entrada y composición
public class Main {
    public static void main(String[] args) {
        // Composición raíz — crea y conecta todas las capas
        EstudianteRepositorio repositorio = new EstudianteRepositorioEnMemoria();
        EstudianteServicio servicio = new EstudianteServicioImpl(repositorio);
        EstudianteControlador controlador = new EstudianteControlador(servicio);

        // Simular operaciones
        System.out.println(controlador.registrar("Ana", "García", "ana@mail.com", "Sistemas"));
        System.out.println(controlador.obtenerEstudiante(1001));
        System.out.println(controlador.listarAprobados());
    }
}
```

---

## Reglas de la Arquitectura de Capas

1. Cada capa solo conoce a la capa inmediatamente inferior.
2. Las capas superiores dependen de **interfaces**, no de implementaciones.
3. El modelo (entidades) puede ser conocido por todas las capas.
4. La composición de dependencias ocurre en el `main` o en un framework de IoC.

---

**Siguiente:** [7.2 MVC y MVP](02-mvc-mvp.md)
