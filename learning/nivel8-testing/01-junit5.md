# 8.1 JUnit 5

JUnit 5 es el framework de testing más popular para Java.
Un test unitario verifica que una pequeña unidad de código (método, clase)
funciona correctamente de forma aislada.

## Dependencia Maven

```xml
<dependency>
    <groupId>org.junit.jupiter</groupId>
    <artifactId>junit-jupiter</artifactId>
    <version>5.10.0</version>
    <scope>test</scope>
</dependency>
```

---

## Anotaciones Principales

| Anotación | Descripción |
|-----------|-------------|
| `@Test` | Marca un método como test |
| `@BeforeEach` | Se ejecuta antes de cada test |
| `@AfterEach` | Se ejecuta después de cada test |
| `@BeforeAll` | Se ejecuta una vez antes de todos los tests (static) |
| `@AfterAll` | Se ejecuta una vez después de todos (static) |
| `@Disabled` | Deshabilita un test temporalmente |
| `@DisplayName` | Nombre descriptivo para el test |
| `@Nested` | Agrupa tests relacionados |
| `@ParameterizedTest` | Test con múltiples entradas |

---

## Tests Básicos

```java
import org.junit.jupiter.api.*;
import static org.junit.jupiter.api.Assertions.*;

class EstudianteTest {

    private Estudiante estudiante;

    @BeforeEach
    void setUp() {
        // Se ejecuta antes de CADA test — objeto fresco para cada test
        estudiante = new Estudiante("Ana", "García", "12345678", "ana@mail.com", 1001, "Sistemas");
    }

    @AfterEach
    void tearDown() {
        // Limpieza si fuera necesario
        estudiante = null;
    }

    @Test
    @DisplayName("Debe retornar nombre completo correctamente")
    void debeRetornarNombreCompleto() {
        // AAA: Arrange - Act - Assert
        // Arrange: ya hecho en setUp

        // Act
        String resultado = estudiante.getNombreCompleto();

        // Assert
        assertEquals("Ana García", resultado);
    }

    @Test
    @DisplayName("Debe aprobar cuando el promedio es mayor o igual a 6")
    void debeAprobarConPromedioAlto() {
        estudiante.setPromedio(8.5);
        assertTrue(estudiante.estaAprobado());
    }

    @Test
    @DisplayName("No debe aprobar con promedio menor a 6")
    void noDebeAprobarConPromedioBajo() {
        estudiante.setPromedio(5.9);
        assertFalse(estudiante.estaAprobado());
    }

    @Test
    @DisplayName("Debe lanzar excepción con nota inválida")
    void debeLanzarExcepcionConNotaInvalida() {
        assertThrows(IllegalArgumentException.class,
            () -> estudiante.setPromedio(11.0));

        assertThrows(IllegalArgumentException.class,
            () -> estudiante.setPromedio(-1.0));
    }

    @Test
    @DisplayName("El legajo no debe cambiar después de la creación")
    void elLegajoNoDebeModificarse() {
        int legajoOriginal = estudiante.getLegajo();
        // ... intentar modificar (si hay setLegajo, no debería existir)
        assertEquals(legajoOriginal, estudiante.getLegajo());
    }
}
```

---

## Assertions (Afirmaciones)

```java
// Igualdad
assertEquals(esperado, actual);
assertEquals(8.5, estudiante.getPromedio(), 0.001); // con delta para doubles
assertNotEquals("Ana", resultado);

// Booleanos
assertTrue(condicion);
assertFalse(condicion);

// Nulos
assertNull(resultado);
assertNotNull(resultado);

// Referencias
assertSame(obj1, obj2);   // mismo objeto en memoria
assertNotSame(obj1, obj2);

// Colecciones
assertIterableEquals(List.of("a","b"), lista);

// Excepciones
Exception ex = assertThrows(IllegalArgumentException.class, () -> metodoQueDebeFallar());
assertEquals("Mensaje esperado", ex.getMessage());

// No lanza excepción
assertDoesNotThrow(() -> metodoQueDebeEjecutarseSinError());

// Múltiples assertions — todas se evalúan aunque alguna falle
assertAll("verificar estudiante",
    () -> assertEquals("Ana", estudiante.getNombre()),
    () -> assertEquals("García", estudiante.getApellido()),
    () -> assertEquals(1001, estudiante.getLegajo())
);

// Tiempo
assertTimeout(Duration.ofMillis(500), () -> {
    // Este código debe ejecutarse en menos de 500ms
    operacionRapida();
});
```

---

## Tests Parametrizados

```java
import org.junit.jupiter.params.ParameterizedTest;
import org.junit.jupiter.params.provider.*;

class CalculadoraNotasTest {

    @ParameterizedTest
    @ValueSource(doubles = {6.0, 7.5, 8.0, 9.0, 10.0})
    @DisplayName("Promedio aprobatorio debe retornar aprobado")
    void promedioAprobatorio(double promedio) {
        Estudiante e = crearEstudianteConPromedio(promedio);
        assertTrue(e.estaAprobado(),
            "Promedio " + promedio + " debería ser aprobatorio");
    }

    @ParameterizedTest
    @ValueSource(doubles = {0.0, 1.0, 3.5, 5.9})
    void promedioReprobatorio(double promedio) {
        Estudiante e = crearEstudianteConPromedio(promedio);
        assertFalse(e.estaAprobado());
    }

    @ParameterizedTest
    @CsvSource({
        "10.0, Sobresaliente",
        "9.0, Sobresaliente",
        "8.0, Notable",
        "7.0, Bien",
        "6.0, Suficiente",
        "5.0, Insuficiente"
    })
    void clasificacionCorrecta(double nota, String estadoEsperado) {
        assertEquals(estadoEsperado, ClasificadorNota.clasificar(nota));
    }

    @ParameterizedTest
    @MethodSource("proveedorDatosEstudiante")
    void registroEstudianteValido(String nombre, String apellido, int legajo) {
        Estudiante e = new Estudiante(nombre, apellido, "dni", "x@x.com", legajo, "Sistemas");
        assertNotNull(e);
        assertEquals(nombre, e.getNombre());
    }

    static Stream<Arguments> proveedorDatosEstudiante() {
        return Stream.of(
            Arguments.of("Ana", "García", 1001),
            Arguments.of("Luis", "Pérez", 1002),
            Arguments.of("Carlos", "Ruiz", 1003)
        );
    }
}
```

---

## Tests Anidados con @Nested

```java
@DisplayName("ServicioInscripcion Tests")
class ServicioInscripcionTest {

    private ServicioInscripcion servicio;
    private EstudianteRepositorioEnMemoria repo;

    @BeforeEach
    void setUp() {
        repo = new EstudianteRepositorioEnMemoria();
        servicio = new ServicioInscripcion(repo);
    }

    @Nested
    @DisplayName("Cuando el estudiante existe y está activo")
    class CuandoExiste {

        private Estudiante estudianteActivo;

        @BeforeEach
        void crearEstudiante() {
            estudianteActivo = new Estudiante("Ana", "García", "1", "a@b.com", 1001, "Sistemas");
            repo.guardar(estudianteActivo);
        }

        @Test
        @DisplayName("Debe inscribir exitosamente")
        void debeInscribirExitosamente() {
            assertDoesNotThrow(() -> servicio.inscribir(1001, "MAT-001"));
        }
    }

    @Nested
    @DisplayName("Cuando el estudiante NO existe")
    class CuandoNoExiste {

        @Test
        @DisplayName("Debe lanzar excepción")
        void debeLanzarExcepcion() {
            assertThrows(RuntimeException.class,
                () -> servicio.inscribir(9999, "MAT-001"));
        }
    }
}
```

---

## Buenas prácticas en Tests

```java
// BIEN — nombre descriptivo: método_escenario_resultado
@Test
void calcularPromedio_ConListaVacia_RetornaCero() { }

@Test
void inscribir_CursoLleno_LanzaCursoLlenoException() { }

@Test
void getNombreCompleto_ConNombreYApellido_RetornaConcatenacion() { }

// BIEN — patrón AAA (Arrange-Act-Assert)
@Test
void registrarNota_NotaValida_ActualizaPromedio() {
    // Arrange
    Estudiante estudiante = new Estudiante("Ana", "García", "1", "a@b.com", 1001, "Sistemas");

    // Act
    estudiante.setPromedio(8.5);

    // Assert
    assertEquals(8.5, estudiante.getPromedio(), 0.001);
    assertTrue(estudiante.estaAprobado());
}
```

---

## Resumen JUnit 5

| Elemento | Descripción |
|---------|-------------|
| `@Test` | Método de test |
| `@BeforeEach` | Setup antes de cada test |
| `assertEquals` | Verificar igualdad |
| `assertThrows` | Verificar que se lanza excepción |
| `@ParameterizedTest` | Ejecutar con múltiples inputs |
| `@Nested` | Agrupar tests relacionados |

**Siguiente:** [8.2 Mockito](02-mockito.md)
