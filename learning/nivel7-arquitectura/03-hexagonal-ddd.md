# 7.3 Arquitectura Hexagonal y DDD

## Arquitectura Hexagonal (Ports & Adapters)

Propuesta por Alistair Cockburn. El objetivo es que la lógica de negocio
(el "núcleo") sea completamente **independiente** de las tecnologías externas
(base de datos, interfaz web, servicios externos).

```
                    [REST API]  [Consola]  [Tests]
                         |          |         |
                    +----+----------+---------+----+
                    |         ADAPTADORES          |
                    |         (Primarios)           |
                    +------------ PORT IN ----------+
                                    |
                    +-----------------------------+
                    |                             |
                    |      NÚCLEO DEL DOMINIO     |
                    |   (Lógica de negocio pura)  |
                    |                             |
                    +---------- PORT OUT ----------+
                    |         ADAPTADORES          |
                    |        (Secundarios)          |
                    +----+----------+---------+----+
                         |          |         |
                    [MySQL]  [MongoDB]  [Email]  [API Externa]
```

---

### Implementación Hexagonal

```java
// ===== DOMINIO (núcleo) — no importa NADA de infraestructura =====

// Entidad del dominio
public class Estudiante {
    private final int legajo;
    private String nombre;
    private String apellido;
    private double promedio;
    private boolean activo;

    public Estudiante(int legajo, String nombre, String apellido) {
        this.legajo = legajo;
        this.nombre = nombre;
        this.apellido = apellido;
        this.promedio = 0.0;
        this.activo = true;
    }

    // Lógica de negocio en la entidad
    public void actualizarPromedio(double nuevaCalificacion) {
        if (nuevaCalificacion < 0 || nuevaCalificacion > 10)
            throw new IllegalArgumentException("Calificación inválida: " + nuevaCalificacion);
        this.promedio = nuevaCalificacion;
    }

    public boolean estaAprobado() { return promedio >= 6.0; }

    public int getLegajo() { return legajo; }
    public String getNombreCompleto() { return nombre + " " + apellido; }
    public double getPromedio() { return promedio; }
    public boolean isActivo() { return activo; }
}

// Puerto de entrada (Caso de Uso / Use Case)
public interface CasoUsoInscribir {
    void inscribir(int legajoEstudiante, String codigoCurso);
}

public interface CasoUsoBuscarEstudiante {
    Optional<Estudiante> buscar(int legajo);
    List<Estudiante> buscarAprobados();
}

// Puerto de salida (para el repositorio)
public interface PuertoEstudianteRepositorio {
    void guardar(Estudiante e);
    Optional<Estudiante> buscarPorLegajo(int legajo);
    List<Estudiante> listarTodos();
}

// Puerto de salida (para notificaciones)
public interface PuertoNotificacion {
    void enviar(int legajoEstudiante, String mensaje);
}

// Implementación del caso de uso — solo conoce interfaces (puertos)
public class ServicioInscripcion implements CasoUsoInscribir {
    private final PuertoEstudianteRepositorio repositorio;
    private final PuertoCursoRepositorio cursoRepo;
    private final PuertoNotificacion notificacion;

    public ServicioInscripcion(PuertoEstudianteRepositorio repo,
                                PuertoCursoRepositorio cursoRepo,
                                PuertoNotificacion notif) {
        this.repositorio = repo;
        this.cursoRepo = cursoRepo;
        this.notificacion = notif;
    }

    @Override
    public void inscribir(int legajoEstudiante, String codigoCurso) {
        Estudiante e = repositorio.buscarPorLegajo(legajoEstudiante)
            .orElseThrow(() -> new RuntimeException("Estudiante no encontrado: " + legajoEstudiante));

        // Lógica de negocio pura
        if (!e.isActivo()) throw new IllegalStateException("El estudiante está inactivo.");

        // ... más lógica ...

        notificacion.enviar(legajoEstudiante, "Inscripción confirmada en: " + codigoCurso);
    }
}

// ===== ADAPTADORES (infraestructura) =====

// Adaptador de repositorio: implementa el puerto usando HashMap
public class EstudianteRepositorioMemoria implements PuertoEstudianteRepositorio {
    private final Map<Integer, Estudiante> datos = new HashMap<>();

    @Override
    public void guardar(Estudiante e) { datos.put(e.getLegajo(), e); }

    @Override
    public Optional<Estudiante> buscarPorLegajo(int legajo) {
        return Optional.ofNullable(datos.get(legajo));
    }

    @Override
    public List<Estudiante> listarTodos() { return new ArrayList<>(datos.values()); }
}

// Adaptador de notificación: implementa el puerto enviando email
public class AdaptadorEmailNotificacion implements PuertoNotificacion {
    @Override
    public void enviar(int legajo, String mensaje) {
        System.out.println("[EMAIL] Legajo " + legajo + ": " + mensaje);
        // En producción: usar JavaMail o SendGrid, etc.
    }
}

// Adaptador REST (entrada primaria)
public class EstudianteRestAdapter {
    private final CasoUsoBuscarEstudiante buscarUseCase;
    private final CasoUsoInscribir inscribirUseCase;

    public EstudianteRestAdapter(CasoUsoBuscarEstudiante buscar, CasoUsoInscribir inscribir) {
        this.buscarUseCase = buscar;
        this.inscribirUseCase = inscribir;
    }

    // GET /estudiantes/{legajo}
    public String obtener(int legajo) {
        return buscarUseCase.buscar(legajo)
            .map(e -> "{\"legajo\":" + e.getLegajo() + ",\"nombre\":\"" + e.getNombreCompleto() + "\"}")
            .orElse("{\"error\":\"No encontrado\"}");
    }

    // POST /inscripciones
    public String inscribir(int legajo, String curso) {
        try {
            inscribirUseCase.inscribir(legajo, curso);
            return "{\"status\":\"OK\"}";
        } catch (Exception ex) {
            return "{\"error\":\"" + ex.getMessage() + "\"}";
        }
    }
}
```

---

## DDD — Domain-Driven Design (Conceptos Básicos)

DDD es una forma de pensar el software centrada en el **dominio del negocio**.

### Conceptos clave de DDD

| Concepto | Descripción | Ejemplo |
|----------|-------------|---------|
| **Entity** | Tiene identidad única | `Estudiante` (identificado por legajo) |
| **Value Object** | Sin identidad propia, definido por sus atributos | `Email`, `Nota`, `Periodo` |
| **Aggregate** | Grupo de entidades con una raíz | `Curso` (contiene `Inscripcion[]`) |
| **Repository** | Abstracción de persistencia por agregado | `CursoRepositorio` |
| **Service** | Lógica que no pertenece a ninguna entidad | `ServicioInscripcion` |
| **Domain Event** | Algo importante que ocurrió | `EstudianteInscripto`, `NotaRegistrada` |

```java
// Value Object — sin setters, comparado por valor
public final class Email {
    private final String valor;

    public Email(String valor) {
        if (valor == null || !valor.matches("^[\\w.-]+@[\\w.-]+\\.[a-z]{2,}$"))
            throw new IllegalArgumentException("Email inválido: " + valor);
        this.valor = valor.toLowerCase();
    }

    public String getValor() { return valor; }

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (!(o instanceof Email)) return false;
        return valor.equals(((Email) o).valor);
    }

    @Override
    public int hashCode() { return valor.hashCode(); }

    @Override
    public String toString() { return valor; }
}

// Domain Event
public record EstudianteInscripto(
    int legajoEstudiante,
    String codigoCurso,
    java.time.LocalDateTime ocurridoEn
) {
    public EstudianteInscripto(int legajo, String curso) {
        this(legajo, curso, java.time.LocalDateTime.now());
    }
}
```

---

## Resumen

| Arquitectura | Fortaleza | Cuándo usar |
|-------------|-----------|-------------|
| Capas | Simple, bien conocida | Aplicaciones CRUD estándar |
| Hexagonal | Testabilidad, independencia tecnológica | Dominio complejo, múltiples adaptadores |
| DDD | Modela dominio complejo fielmente | Proyectos grandes con reglas complejas |

**Siguiente:** [7.4 Microservicios](04-microservicios.md)
