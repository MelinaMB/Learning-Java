# Ejercicios — Nivel 2: POO

## Ejercicio 1 — Clase básica con constructor

**Enunciado:** Crea la clase `Materia` con los atributos: `id (int)`, `nombre (String)`,
`codigo (String)`, `horasSemanales (int)`, `area (String)`.
Incluye: constructor completo, getters, `toString()`, y un método `esIntensiva()`
que retorne `true` si tiene más de 5 horas semanales.

<details>
<summary>Ver solución</summary>

```java
public class Materia {
    private int id;
    private String nombre;
    private String codigo;
    private int horasSemanales;
    private String area;

    public Materia(int id, String nombre, String codigo, int horasSemanales, String area) {
        this.id = id;
        this.nombre = nombre;
        this.codigo = codigo;
        this.horasSemanales = horasSemanales;
        this.area = area;
    }

    public int getId() { return id; }
    public String getNombre() { return nombre; }
    public String getCodigo() { return codigo; }
    public int getHorasSemanales() { return horasSemanales; }
    public String getArea() { return area; }

    public boolean esIntensiva() {
        return horasSemanales > 5;
    }

    @Override
    public String toString() {
        return String.format("Materia{codigo='%s', nombre='%s', horas=%d, area='%s'}",
            codigo, nombre, horasSemanales, area);
    }
}
```
</details>

---

## Ejercicio 2 — Encapsulamiento con validaciones

**Enunciado:** Crea la clase `Calificacion` que sea **inmutable** (usa `final`):
- `legajoEstudiante (int)`
- `codigoMateria (String)`
- `nota (double)` — debe estar entre 0 y 10
- `fecha (String)`
- `tipo (String)` — "PARCIAL", "FINAL", "TRABAJO_PRACTICO"

Métodos: `estaAprobada()`, `esPromocion()` (nota >= 8), `toString()`.

<details>
<summary>Ver solución</summary>

```java
public final class Calificacion {
    private final int legajoEstudiante;
    private final String codigoMateria;
    private final double nota;
    private final String fecha;
    private final String tipo;

    public Calificacion(int legajoEstudiante, String codigoMateria,
                        double nota, String fecha, String tipo) {
        if (nota < 0 || nota > 10) throw new IllegalArgumentException("Nota inválida: " + nota);
        if (!tipo.equals("PARCIAL") && !tipo.equals("FINAL") && !tipo.equals("TRABAJO_PRACTICO"))
            throw new IllegalArgumentException("Tipo inválido: " + tipo);
        this.legajoEstudiante = legajoEstudiante;
        this.codigoMateria = codigoMateria;
        this.nota = nota;
        this.fecha = fecha;
        this.tipo = tipo;
    }

    public int getLegajoEstudiante() { return legajoEstudiante; }
    public String getCodigoMateria() { return codigoMateria; }
    public double getNota() { return nota; }
    public String getFecha() { return fecha; }
    public String getTipo() { return tipo; }

    public boolean estaAprobada() { return nota >= 6.0; }
    public boolean esPromocion() { return nota >= 8.0; }

    @Override
    public String toString() {
        return String.format("Calificacion{materia='%s', tipo='%s', nota=%.1f, aprobada=%b}",
            codigoMateria, tipo, nota, estaAprobada());
    }
}
```
</details>

---

## Ejercicio 3 — Herencia

**Enunciado:** Diseña la siguiente jerarquía:

```
Aula (abstracta)
├── AulaTeoria  (capacidad, tiene proyector)
└── AulaLaboratorio (capacidad, cantidad de computadoras, sistema operativo)
```

Atributos comunes en `Aula`: `numero (int)`, `piso (int)`, `disponible (boolean)`.
Método abstracto en `Aula`: `String getDescripcion()`.
Método concreto en `Aula`: `void reservar()` (cambia disponible a false).

<details>
<summary>Ver solución</summary>

```java
public abstract class Aula {
    private int numero;
    private int piso;
    private boolean disponible;

    public Aula(int numero, int piso) {
        this.numero = numero;
        this.piso = piso;
        this.disponible = true;
    }

    public int getNumero() { return numero; }
    public int getPiso() { return piso; }
    public boolean isDisponible() { return disponible; }

    public void reservar() {
        if (!disponible) {
            System.out.println("Aula " + numero + " ya está reservada.");
            return;
        }
        this.disponible = false;
        System.out.println("Aula " + numero + " reservada exitosamente.");
    }

    public void liberar() { this.disponible = true; }

    public abstract String getDescripcion();

    @Override
    public String toString() {
        return String.format("Aula %d (Piso %d) — %s — %s",
            numero, piso, getDescripcion(), disponible ? "Libre" : "Ocupada");
    }
}

public class AulaTeoria extends Aula {
    private int capacidad;
    private boolean tieneProyector;

    public AulaTeoria(int numero, int piso, int capacidad, boolean tieneProyector) {
        super(numero, piso);
        this.capacidad = capacidad;
        this.tieneProyector = tieneProyector;
    }

    @Override
    public String getDescripcion() {
        return String.format("Teoría | Cap: %d | Proyector: %s",
            capacidad, tieneProyector ? "Sí" : "No");
    }
}

public class AulaLaboratorio extends Aula {
    private int capacidad;
    private int cantidadComputadoras;
    private String sistemaOperativo;

    public AulaLaboratorio(int numero, int piso, int capacidad,
                           int cantidadComputadoras, String sistemaOperativo) {
        super(numero, piso);
        this.capacidad = capacidad;
        this.cantidadComputadoras = cantidadComputadoras;
        this.sistemaOperativo = sistemaOperativo;
    }

    @Override
    public String getDescripcion() {
        return String.format("Laboratorio | Cap: %d | %d PCs | SO: %s",
            capacidad, cantidadComputadoras, sistemaOperativo);
    }
}
```
</details>

---

## Ejercicio 4 — Interfaces

**Enunciado:** Define la interfaz `Reportable` con el método `String generarReporte()`.
Implementa esta interfaz en `Estudiante`, `Curso` y `Profesor`.
Luego escribe un método estático `imprimirTodos(Reportable... items)` que imprima
el reporte de cada item.

---

## Ejercicio 5 — Proyecto Integrador Nivel 2

**Enunciado:** Crea un pequeño sistema con las siguientes clases:

- `Persona` (abstracta): nombre, apellido, dni
- `Estudiante extends Persona`: legajo, lista de notas (array), métodos de evaluación
- `Profesor extends Persona`: especialidad, materia que dicta
- `Curso`: nombre, profesor asignado, array de estudiantes (máx 30)
  - Método `inscribir(Estudiante e)` — agrega al array si hay lugar
  - Método `obtenerPromedioCurso()` — promedio de todos los promedios
  - Método `listarAprobados()` — imprime los estudiantes con promedio >= 6
- `Main`: crea un curso, un profesor, 5 estudiantes con notas y muestra estadísticas

---

## Checklist de Nivel 2

- [ ] ¿Puedes explicar la diferencia entre clase e instancia?
- [ ] ¿Sabes cuándo usar `private`, `protected` y `public`?
- [ ] ¿Entiendes por qué no debes hacer los atributos `public`?
- [ ] ¿Sabes la diferencia entre sobreescribir (`@Override`) y sobrecargar?
- [ ] ¿Puedes explicar el polimorfismo con un ejemplo concreto?
- [ ] ¿Sabes cuándo usar clase abstracta y cuándo usar interfaz?
- [ ] ¿Puedes crear una clase inmutable?

Si puedes responderlas, estás listo para el [Nivel 3 — Java Intermedio](../nivel3-intermedio/README.md).
