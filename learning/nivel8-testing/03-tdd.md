# 8.3 TDD — Test Driven Development

TDD (Desarrollo Guiado por Tests) es una práctica donde los tests se escriben
**antes** que el código de producción.

## El Ciclo Red-Green-Refactor

```
    +----------+
    |   RED    |  1. Escribe un test que FALLA
    | (falla)  |     (no hay código todavía)
    +----+-----+
         |
         v
    +----------+
    |  GREEN   |  2. Escribe el MÍNIMO código para que pase
    | (pasa)   |     (no importa qué tan feo sea)
    +----+-----+
         |
         v
    +----------+
    | REFACTOR |  3. Mejora el código sin cambiar el comportamiento
    |  (limpia) |     (los tests siguen pasando)
    +----+-----+
         |
         v
    (repetir)
```

---

## Ejemplo Completo — Desarrollar `CalculadorPromedio` con TDD

### Iteración 1 — Promedio simple

**RED — Escribir el test primero:**

```java
class CalculadorPromedioTest {

    @Test
    void calcularPromedio_DosNotas_RetornaMedia() {
        CalculadorPromedio calc = new CalculadorPromedio();
        double resultado = calc.calcular(8.0, 6.0);
        assertEquals(7.0, resultado, 0.001);
    }
}
// CalculadorPromedio NO EXISTE aún — el test no compila (RED)
```

**GREEN — Mínimo código para pasar:**

```java
public class CalculadorPromedio {
    public double calcular(double n1, double n2) {
        return (n1 + n2) / 2.0;
    }
}
// Test pasa (GREEN)
```

**REFACTOR — ¿Hay algo que mejorar? Aquí no mucho.**

---

### Iteración 2 — Manejar lista vacía

**RED:**
```java
@Test
void calcularPromedio_ListaVacia_RetornaCero() {
    CalculadorPromedio calc = new CalculadorPromedio();
    double resultado = calc.calcular(List.of());
    assertEquals(0.0, resultado, 0.001);
}
// Falla: el método no acepta List<Double>
```

**GREEN:**
```java
public class CalculadorPromedio {
    public double calcular(double n1, double n2) {
        return (n1 + n2) / 2.0;
    }

    public double calcular(List<Double> notas) {
        if (notas.isEmpty()) return 0.0;
        double suma = 0;
        for (double n : notas) suma += n;
        return suma / notas.size();
    }
}
```

---

### Iteración 3 — Validar nota inválida

**RED:**
```java
@Test
void calcular_NotaFueraDeRango_LanzaExcepcion() {
    CalculadorPromedio calc = new CalculadorPromedio();
    assertThrows(IllegalArgumentException.class,
        () -> calc.calcular(List.of(8.0, 11.0, 7.0)));
}
```

**GREEN:**
```java
public double calcular(List<Double> notas) {
    if (notas.isEmpty()) return 0.0;
    for (double nota : notas) {
        if (nota < 0 || nota > 10)
            throw new IllegalArgumentException("Nota fuera de rango: " + nota);
    }
    double suma = 0;
    for (double n : notas) suma += n;
    return suma / notas.size();
}
```

**REFACTOR — Usar Stream:**
```java
public double calcular(List<Double> notas) {
    if (notas.isEmpty()) return 0.0;
    notas.forEach(nota -> {
        if (nota < 0 || nota > 10)
            throw new IllegalArgumentException("Nota fuera de rango: " + nota);
    });
    return notas.stream().mapToDouble(Double::doubleValue).average().orElse(0);
}
// Tests siguen pasando — REFACTOR exitoso
```

---

## Ejemplo Realista — Desarrollar `ServicioInscripcion` con TDD

```java
// Primero los tests
@ExtendWith(MockitoExtension.class)
class ServicioInscripcionTDDTest {

    @Mock EstudianteRepositorio estudianteRepo;
    @Mock CursoRepositorio cursoRepo;
    @InjectMocks ServicioInscripcion servicio;

    // Test 1: caso feliz
    @Test
    void inscribir_EstudianteYCursoExistentes_Exitoso() {
        when(estudianteRepo.buscarPorLegajo(1001))
            .thenReturn(Optional.of(crearEstudiante(1001)));
        when(cursoRepo.buscarPorCodigo("MAT-001"))
            .thenReturn(Optional.of(crearCurso("MAT-001", false)));

        assertDoesNotThrow(() -> servicio.inscribir(1001, "MAT-001"));
        verify(cursoRepo).actualizar(any(Curso.class));
    }

    // Test 2: estudiante no existe
    @Test
    void inscribir_EstudianteNoExiste_LanzaExcepcion() {
        when(estudianteRepo.buscarPorLegajo(anyInt())).thenReturn(Optional.empty());

        assertThrows(EstudianteNoEncontradoException.class,
            () -> servicio.inscribir(9999, "MAT-001"));
    }

    // Test 3: curso lleno
    @Test
    void inscribir_CursoLleno_LanzaCursoLlenoException() {
        when(estudianteRepo.buscarPorLegajo(1001))
            .thenReturn(Optional.of(crearEstudiante(1001)));
        when(cursoRepo.buscarPorCodigo("MAT-001"))
            .thenReturn(Optional.of(crearCurso("MAT-001", true))); // curso lleno

        assertThrows(CursoLlenoException.class,
            () -> servicio.inscribir(1001, "MAT-001"));
    }

    private Estudiante crearEstudiante(int legajo) {
        return new Estudiante("Test", "Test", "1", "t@t.com", legajo, "Sistemas");
    }

    private Curso crearCurso(String codigo, boolean lleno) {
        Curso c = new Curso(1, "Test", codigo, 1, 1);
        if (lleno) {
            // Llenar el curso hasta la capacidad máxima
            for (int i = 0; i < c.getCapacidadMaxima(); i++) {
                c.inscribir(new Estudiante("X", "X", String.valueOf(i), "x@x.com", i, "X"));
            }
        }
        return c;
    }
}
```

---

## Beneficios del TDD

| Beneficio | Descripción |
|-----------|-------------|
| **Diseño guiado** | Pensar en el uso antes que en la implementación mejora el diseño |
| **Cobertura alta** | Todo el código tiene tests desde el inicio |
| **Confianza en refactoring** | Los tests detectan regresiones inmediatamente |
| **Documentación viva** | Los tests describen qué debe hacer el código |
| **Debugging más fácil** | Los tests aíslan exactamente dónde está el error |

---

## Resumen TDD

| Paso | Acción |
|------|--------|
| RED | Escribe un test que falla (porque el código no existe) |
| GREEN | Escribe el mínimo código para hacer pasar el test |
| REFACTOR | Mejora el código manteniendo los tests en verde |
| Repetir | Para cada nueva funcionalidad |

**Siguiente:** [Ejercicios del Nivel 8](ejercicios.md)
