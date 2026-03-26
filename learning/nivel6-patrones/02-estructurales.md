# 6.2 Patrones Estructurales

Los patrones estructurales explican cómo **componer objetos y clases** para
formar estructuras más grandes manteniendo la flexibilidad y eficiencia.

---

## Adapter (Adaptador)

**Propósito:** Permite que interfaces incompatibles trabajen juntas.
Actúa como un "traductor" entre dos interfaces.

```java
// Sistema externo de notas (interfaz que no podemos cambiar)
public class SistemaNotasExterno {
    public double[] obtenerCalificaciones(String dniEstudiante) {
        // Retorna array de doubles
        return new double[]{8.0, 7.5, 9.0};
    }
    public String obtenerNombreEstudiante(String dni) {
        return "Ana García"; // datos del sistema externo
    }
}

// Nuestra interfaz interna
public interface RepositorioNotas {
    List<Double> obtenerNotas(String dniEstudiante);
    Optional<String> obtenerNombre(String dniEstudiante);
}

// Adapter — adapta el sistema externo a nuestra interfaz
public class AdaptadorSistemaExterno implements RepositorioNotas {
    private final SistemaNotasExterno sistemaExterno;

    public AdaptadorSistemaExterno(SistemaNotasExterno sistemaExterno) {
        this.sistemaExterno = sistemaExterno;
    }

    @Override
    public List<Double> obtenerNotas(String dniEstudiante) {
        double[] notas = sistemaExterno.obtenerCalificaciones(dniEstudiante);
        List<Double> lista = new ArrayList<>();
        for (double n : notas) lista.add(n);
        return lista;
    }

    @Override
    public Optional<String> obtenerNombre(String dniEstudiante) {
        String nombre = sistemaExterno.obtenerNombreEstudiante(dniEstudiante);
        return Optional.ofNullable(nombre);
    }
}

// Uso — nuestro código solo conoce RepositorioNotas
RepositorioNotas repo = new AdaptadorSistemaExterno(new SistemaNotasExterno());
List<Double> notas = repo.obtenerNotas("12345678");
System.out.println(notas); // [8.0, 7.5, 9.0]
```

---

## Decorator (Decorador)

**Propósito:** Añade responsabilidades a un objeto **dinámicamente**,
sin alterar su clase ni crear subclases para cada combinación.

```java
// Componente base
public interface ServicioNotificacion {
    void enviar(String destinatario, String mensaje);
}

// Implementación base
public class NotificacionEmailBasico implements ServicioNotificacion {
    @Override
    public void enviar(String destinatario, String mensaje) {
        System.out.println("Email a " + destinatario + ": " + mensaje);
    }
}

// Decorador abstracto
public abstract class NotificacionDecorador implements ServicioNotificacion {
    protected final ServicioNotificacion componente;

    public NotificacionDecorador(ServicioNotificacion componente) {
        this.componente = componente;
    }

    @Override
    public void enviar(String destinatario, String mensaje) {
        componente.enviar(destinatario, mensaje);
    }
}

// Decoradores concretos
public class NotificacionConLog extends NotificacionDecorador {
    public NotificacionConLog(ServicioNotificacion componente) {
        super(componente);
    }

    @Override
    public void enviar(String destinatario, String mensaje) {
        System.out.println("[LOG " + java.time.LocalDateTime.now() + "] Enviando a: " + destinatario);
        componente.enviar(destinatario, mensaje);
        System.out.println("[LOG] Envío completado.");
    }
}

public class NotificacionConReintento extends NotificacionDecorador {
    private final int maxReintentos;

    public NotificacionConReintento(ServicioNotificacion componente, int maxReintentos) {
        super(componente);
        this.maxReintentos = maxReintentos;
    }

    @Override
    public void enviar(String destinatario, String mensaje) {
        for (int intento = 1; intento <= maxReintentos; intento++) {
            try {
                componente.enviar(destinatario, mensaje);
                return; // éxito
            } catch (Exception e) {
                System.out.println("Intento " + intento + " fallido. Reintentando...");
            }
        }
        System.out.println("Falló después de " + maxReintentos + " intentos.");
    }
}

// Composición dinámica
ServicioNotificacion servicio = new NotificacionConLog(
    new NotificacionConReintento(
        new NotificacionEmailBasico(),
        3
    )
);
servicio.enviar("ana@mail.com", "Tu inscripción fue confirmada.");
```

---

## Facade (Fachada)

**Propósito:** Provee una **interfaz simplificada** a un sistema complejo de clases.

```java
// Sistema complejo con muchos subsistemas
public class ValidadorEstudiante {
    public boolean validar(Estudiante e) { /* ... */ return true; }
}
public class ServicioBD {
    public void guardar(Estudiante e) { System.out.println("Guardado en BD"); }
    public void asignarLegajo(Estudiante e) { System.out.println("Legajo asignado"); }
}
public class ServicioEmail {
    public void enviarConfirmacion(Estudiante e) { System.out.println("Email enviado"); }
}
public class ServicioCarnet {
    public void generarCarnet(Estudiante e) { System.out.println("Carnet generado"); }
}
public class AuditoriaServicio {
    public void registrar(String evento, Object data) {
        System.out.println("[AUDITORIA] " + evento + ": " + data);
    }
}

// Facade — simplifica el proceso de registro
public class FacadeRegistroEstudiante {
    private final ValidadorEstudiante validador = new ValidadorEstudiante();
    private final ServicioBD bd = new ServicioBD();
    private final ServicioEmail email = new ServicioEmail();
    private final ServicioCarnet carnet = new ServicioCarnet();
    private final AuditoriaServicio auditoria = new AuditoriaServicio();

    public void registrar(Estudiante nuevoEstudiante) {
        System.out.println("Iniciando registro de: " + nuevoEstudiante.getNombreCompleto());

        if (!validador.validar(nuevoEstudiante)) {
            throw new IllegalStateException("Datos de estudiante inválidos.");
        }

        bd.asignarLegajo(nuevoEstudiante);
        bd.guardar(nuevoEstudiante);
        email.enviarConfirmacion(nuevoEstudiante);
        carnet.generarCarnet(nuevoEstudiante);
        auditoria.registrar("REGISTRO_ESTUDIANTE", nuevoEstudiante.getLegajo());

        System.out.println("Registro completado exitosamente.");
    }
}

// El cliente solo habla con la Facade
FacadeRegistroEstudiante facade = new FacadeRegistroEstudiante();
facade.registrar(nuevoEstudiante); // Una sola llamada simple
```

---

## Proxy

**Propósito:** Provee un sustituto de otro objeto para controlar el acceso a él.

```java
// Interfaz
public interface EstudianteRepositorio {
    Optional<Estudiante> buscarPorLegajo(int legajo);
    void guardar(Estudiante e);
}

// Implementación real (costosa — accede a BD)
public class EstudianteRepositorioImpl implements EstudianteRepositorio {
    @Override
    public Optional<Estudiante> buscarPorLegajo(int legajo) {
        System.out.println("Consultando base de datos para legajo: " + legajo);
        // Simula consulta costosa
        return Optional.of(new Estudiante("Ana", "García", "1", "a@b.com", legajo, "Sistemas"));
    }

    @Override
    public void guardar(Estudiante e) {
        System.out.println("Guardando en BD: " + e.getNombreCompleto());
    }
}

// Proxy con caché
public class EstudianteRepositorioProxy implements EstudianteRepositorio {
    private final EstudianteRepositorioImpl repositorioReal;
    private final Map<Integer, Estudiante> cache = new HashMap<>();

    public EstudianteRepositorioProxy() {
        this.repositorioReal = new EstudianteRepositorioImpl();
    }

    @Override
    public Optional<Estudiante> buscarPorLegajo(int legajo) {
        if (cache.containsKey(legajo)) {
            System.out.println("Cache hit para legajo: " + legajo);
            return Optional.of(cache.get(legajo));
        }

        Optional<Estudiante> resultado = repositorioReal.buscarPorLegajo(legajo);
        resultado.ifPresent(e -> cache.put(legajo, e));
        return resultado;
    }

    @Override
    public void guardar(Estudiante e) {
        repositorioReal.guardar(e);
        cache.put(e.getLegajo(), e); // actualiza caché también
    }
}

// Uso transparente
EstudianteRepositorio repo = new EstudianteRepositorioProxy();
repo.buscarPorLegajo(1001); // Consulta BD
repo.buscarPorLegajo(1001); // Cache hit — más rápido
```

---

## Composite

**Propósito:** Compone objetos en estructuras de árbol para representar
jerarquías todo-parte. Trata objetos individuales y composiciones de forma uniforme.

```java
// Componente
public interface ComponenteOrganizacional {
    String getNombre();
    void mostrar(String indentacion);
    int getCantidadPersonas();
}

// Hoja (elemento sin hijos)
public class Empleado implements ComponenteOrganizacional {
    private final String nombre;
    private final String cargo;

    public Empleado(String nombre, String cargo) {
        this.nombre = nombre;
        this.cargo = cargo;
    }

    @Override
    public String getNombre() { return nombre; }

    @Override
    public void mostrar(String indentacion) {
        System.out.println(indentacion + "- " + nombre + " (" + cargo + ")");
    }

    @Override
    public int getCantidadPersonas() { return 1; }
}

// Compuesto (puede tener hijos)
public class DepartamentoEscolar implements ComponenteOrganizacional {
    private final String nombre;
    private final List<ComponenteOrganizacional> miembros = new ArrayList<>();

    public DepartamentoEscolar(String nombre) {
        this.nombre = nombre;
    }

    public void agregar(ComponenteOrganizacional c) { miembros.add(c); }
    public void quitar(ComponenteOrganizacional c) { miembros.remove(c); }

    @Override
    public String getNombre() { return nombre; }

    @Override
    public void mostrar(String indentacion) {
        System.out.println(indentacion + "+ " + nombre + " (" + getCantidadPersonas() + " personas)");
        for (ComponenteOrganizacional m : miembros) {
            m.mostrar(indentacion + "  ");
        }
    }

    @Override
    public int getCantidadPersonas() {
        return miembros.stream().mapToInt(ComponenteOrganizacional::getCantidadPersonas).sum();
    }
}

// Construir jerarquía
DepartamentoEscolar escuela = new DepartamentoEscolar("Escuela Técnica N°1");

DepartamentoEscolar dpto1 = new DepartamentoEscolar("Dpto. Sistemas");
dpto1.agregar(new Empleado("María González", "Profesora"));
dpto1.agregar(new Empleado("Roberto Sánchez", "Profesor"));

DepartamentoEscolar dpto2 = new DepartamentoEscolar("Dpto. Ciencias");
dpto2.agregar(new Empleado("Laura Torres", "Profesora"));

escuela.agregar(new Empleado("Directora García", "Directora"));
escuela.agregar(dpto1);
escuela.agregar(dpto2);

escuela.mostrar("");
System.out.println("Total: " + escuela.getCantidadPersonas() + " personas");
```

---

## Resumen de Patrones Estructurales

| Patrón | Propósito | Cuándo usar |
|--------|-----------|-------------|
| **Adapter** | Traducir entre interfaces incompatibles | Integrar sistemas externos |
| **Decorator** | Añadir responsabilidades dinámicamente | Funcionalidades opcionales combinables |
| **Facade** | Simplificar interfaz compleja | Ocultar complejidad de subsistemas |
| **Proxy** | Controlar acceso a un objeto | Caché, seguridad, lazy loading |
| **Composite** | Jerarquías todo-parte | Estructuras de árbol |

**Siguiente:** [6.3 Patrones de Comportamiento](03-comportamiento.md)
