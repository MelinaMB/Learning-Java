# 5.1 Clean Code

"Clean Code" (Código Limpio) es el arte de escribir código que sea fácil de
leer, entender y mantener por otros desarrolladores (y por ti mismo en el futuro).

> "El código se lee diez veces más de lo que se escribe." — Robert C. Martin

---

## Nombres Significativos

El nombre de una variable, método o clase debe revelar su intención.

```java
// MAL — ¿qué significa d, n, e?
int d = 30;
String n = "Ana García";
List<Object> e = new ArrayList<>();

// BIEN — nombres que expresan intención
int diasHastaCierreInscripcion = 30;
String nombreCompletoEstudiante = "Ana García";
List<Estudiante> estudiantesAprobados = new ArrayList<>();
```

### Nombres para clases, métodos y variables

```java
// CLASES — sustantivos que describen QUÉ son
class Estudiante { }          // BIEN
class GestorEstudiante { }    // BIEN — gestora algo
class EStudiante { }          // MAL — abreviatura sin sentido
class Datos { }               // MAL — demasiado genérico

// MÉTODOS — verbos que describen QUÉ hacen
public void inscribirEstudiante(Estudiante e) { }   // BIEN
public List<Estudiante> obtenerAprobados() { }      // BIEN
public boolean estaAprobado() { }                   // BIEN — booleanos con is/has/esta
public void proc(Estudiante e) { }                  // MAL — ¿qué procesa?
public void hacerCosa() { }                         // MAL — no dice nada

// VARIABLES BOOLEANAS — como preguntas de sí/no
boolean estaActivo = true;         // BIEN
boolean tieneDeuda = false;        // BIEN
boolean activo = true;             // Aceptable
boolean flag = true;               // MAL — ¿qué representa?
boolean b = false;                 // MAL — completamente opaco

// NÚMEROS — evitar "números mágicos"
// MAL
if (nota >= 6.0) { }
if (faltas > 5) { }

// BIEN — usa constantes con nombre
final double NOTA_MINIMA_APROBACION = 6.0;
final int MAX_FALTAS_PERMITIDAS = 5;

if (nota >= NOTA_MINIMA_APROBACION) { }
if (faltas > MAX_FALTAS_PERMITIDAS) { }
```

---

## Funciones/Métodos Pequeños

Un método debe hacer **una sola cosa** y hacerla bien.

```java
// MAL — método que hace demasiado
public void procesarEstudiante(Estudiante e, double nota, List<String> emails) {
    // Validar nota
    if (nota < 0 || nota > 10) throw new IllegalArgumentException("Nota inválida");

    // Actualizar promedio
    List<Double> notas = repositorio.obtenerNotas(e.getLegajo());
    notas.add(nota);
    double suma = 0;
    for (double n : notas) suma += n;
    double promedio = suma / notas.length;
    e.setPromedio(promedio);
    repositorio.actualizar(e);

    // Enviar email
    if (promedio >= 6.0) {
        for (String email : emails) {
            enviarEmail(email, "Aprobado", "Felicidades, " + e.getNombre() + "...");
        }
    } else {
        for (String email : emails) {
            enviarEmail(email, "Reprobado", "Lamentablemente...");
        }
    }

    // Loguear
    System.out.println("[" + LocalDateTime.now() + "] Estudiante " + e.getLegajo() + " procesado.");
}

// BIEN — cada método tiene responsabilidad única
public void registrarCalificacion(int legajoEstudiante, double nota) {
    validarNota(nota);
    double nuevoPromedio = calcularNuevoPromedio(legajoEstudiante, nota);
    actualizarPromedio(legajoEstudiante, nuevoPromedio);
    notificarResultado(legajoEstudiante, nuevoPromedio);
    registrarAuditoria(legajoEstudiante, nota);
}

private void validarNota(double nota) {
    if (nota < 0 || nota > 10)
        throw new NotaInvalidaException(nota);
}

private double calcularNuevoPromedio(int legajo, double nuevaNota) {
    List<Double> notas = repositorio.obtenerNotas(legajo);
    notas.add(nuevaNota);
    return notas.stream().mapToDouble(Double::doubleValue).average().orElse(0);
}

private void actualizarPromedio(int legajo, double promedio) {
    repositorio.actualizarPromedio(legajo, promedio);
}

private void notificarResultado(int legajo, double promedio) {
    Estudiante e = repositorio.buscar(legajo).orElseThrow();
    notificacionServicio.enviar(e, promedio >= NOTA_MINIMA ? "Aprobado" : "Reprobado");
}
```

---

## Comentarios

Los comentarios deberían ser la excepción, no la regla.
El código bien escrito es autoexplicativo.

```java
// MAL — comentario que repite lo que hace el código
// Incrementa el contador en 1
contador++;

// MAL — comentario de código comentado (¡bórralo!)
// estudiante.setPromedio(7.5);
// System.out.println("debug: " + estudiante);

// BIEN — comentario que explica el PORQUÉ (no el qué)
// La nota mínima es 6.0 por reglamento académico vigente desde 2020
// Ver acta de resolución 245/2020
private static final double NOTA_MINIMA = 6.0;

// BIEN — advertencia o complejidad no obvia
// Este cálculo usa la media ponderada según los créditos de cada materia.
// No reemplazar por media simple sin revisar con el área académica.
private double calcularPromedioAcademico(List<Calificacion> calificaciones) { ... }
```

---

## Formateo y Estructura

```java
// BIEN — espacio vertical para separar conceptos
public class ServicioInscripcion {

    private final EstudianteRepositorio estudianteRepo;
    private final CursoRepositorio cursoRepo;
    private final NotificacionServicio notifServicio;

    public ServicioInscripcion(EstudianteRepositorio estudianteRepo,
                                CursoRepositorio cursoRepo,
                                NotificacionServicio notifServicio) {
        this.estudianteRepo = estudianteRepo;
        this.cursoRepo = cursoRepo;
        this.notifServicio = notifServicio;
    }

    public void inscribir(int legajoEstudiante, String codigoCurso) {
        Estudiante estudiante = obtenerEstudiante(legajoEstudiante);
        Curso curso = obtenerCurso(codigoCurso);

        validarInscripcion(estudiante, curso);
        realizarInscripcion(estudiante, curso);
        notificarInscripcion(estudiante, curso);
    }

    private Estudiante obtenerEstudiante(int legajo) {
        return estudianteRepo.buscarPorLegajo(legajo)
            .orElseThrow(() -> new EstudianteNoEncontradoException("Legajo: " + legajo, legajo));
    }

    private Curso obtenerCurso(String codigo) {
        return cursoRepo.buscarPorCodigo(codigo)
            .orElseThrow(() -> new CursoInexistenteException("Código: " + codigo));
    }

    private void validarInscripcion(Estudiante e, Curso c) {
        if (c.estaLleno())
            throw new CursoLlenoException(c.getNombre(), c.getCapacidadMaxima());
        if (c.estaInscripto(e.getLegajo()))
            throw new InscripcionDuplicadaException(e.getLegajo(), c.getCodigo());
    }

    private void realizarInscripcion(Estudiante e, Curso c) {
        c.inscribir(e);
        cursoRepo.actualizar(c);
    }

    private void notificarInscripcion(Estudiante e, Curso c) {
        notifServicio.enviar(e, "Inscripción confirmada en: " + c.getNombre());
    }
}
```

---

## Resumen de Reglas de Clean Code

| Regla | Descripción |
|-------|-------------|
| Nombres intencionales | El nombre revela propósito |
| Sin números mágicos | Usa constantes con nombre |
| Métodos pequeños | Una responsabilidad por método |
| Sin comentarios obvios | El código debe ser autoexplicativo |
| Sin código comentado | Si no se usa, bórralo |
| Separación vertical | Líneas en blanco entre conceptos |
| Nivel de abstracción | Un método no mezcla alto y bajo nivel |

**Siguiente:** [5.2 Principios SOLID](02-solid.md)
