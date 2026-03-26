# 8.2 Mockito

Mockito es la librería de mocking más popular para Java.
Permite crear **objetos simulados (mocks)** que imitan el comportamiento
de dependencias reales, permitiendo testear en aislamiento.

## Dependencia Maven

```xml
<dependency>
    <groupId>org.mockito</groupId>
    <artifactId>mockito-junit-jupiter</artifactId>
    <version>5.5.0</version>
    <scope>test</scope>
</dependency>
```

---

## Conceptos

| Concepto | Descripción |
|----------|-------------|
| **Mock** | Objeto simulado que controlas completamente |
| **Stub** | Configurar qué retorna un mock ante una llamada |
| **Verify** | Verificar que un método fue llamado |
| **ArgumentCaptor** | Capturar el argumento pasado a un mock |
| **Spy** | Objeto real con algunos métodos reemplazados |

---

## Crear Mocks

```java
import org.junit.jupiter.api.extension.ExtendWith;
import org.mockito.Mock;
import org.mockito.InjectMocks;
import org.mockito.junit.jupiter.MockitoExtension;
import static org.mockito.Mockito.*;
import static org.junit.jupiter.api.Assertions.*;

@ExtendWith(MockitoExtension.class)
class EstudianteServicioTest {

    @Mock
    private EstudianteRepositorio repositorio;   // Mock de la dependencia

    @Mock
    private NotificacionServicio notifServicio;  // Mock de otra dependencia

    @InjectMocks
    private EstudianteServicioImpl servicio;     // La clase que queremos testear
    // Mockito inyecta automáticamente los mocks en el constructor
}
```

---

## Stubbing — Configurar comportamiento

```java
@Test
void buscar_EstudianteExistente_RetornaEstudiante() {
    // Arrange — configurar el mock
    Estudiante esperado = new Estudiante("Ana", "García", "1", "a@b.com", 1001, "Sistemas");
    when(repositorio.buscarPorLegajo(1001)).thenReturn(Optional.of(esperado));

    // Act
    Optional<Estudiante> resultado = servicio.buscar(1001);

    // Assert
    assertTrue(resultado.isPresent());
    assertEquals("Ana García", resultado.get().getNombreCompleto());
}

@Test
void buscar_EstudianteInexistente_RetornaVacio() {
    when(repositorio.buscarPorLegajo(anyInt())).thenReturn(Optional.empty());

    Optional<Estudiante> resultado = servicio.buscar(9999);

    assertFalse(resultado.isPresent());
}

@Test
void registrar_FallaEnBD_PropagaExcepcion() {
    doThrow(new RuntimeException("Error de conexión"))
        .when(repositorio).guardar(any(Estudiante.class));

    assertThrows(RuntimeException.class,
        () -> servicio.registrar("Ana", "García", "ana@mail.com", "Sistemas"));
}
```

### Matchers

```java
// any() — cualquier valor del tipo
when(repo.buscarPorLegajo(anyInt())).thenReturn(Optional.empty());
when(repo.guardar(any(Estudiante.class))).thenReturn(null);
when(repo.buscarPorCarrera(anyString())).thenReturn(List.of());

// Valores específicos
when(repo.buscarPorLegajo(1001)).thenReturn(Optional.of(estudiante));
when(repo.buscarPorLegajo(eq(1001))).thenReturn(Optional.of(estudiante)); // explícito

// Predicados
when(repo.buscarPorLegajo(intThat(l -> l > 1000))).thenReturn(Optional.of(estudiante));
```

---

## Verify — Verificar llamadas

```java
@Test
void registrar_DatosValidos_LlamaGuardarYNotificacion() {
    // Arrange
    when(repositorio.buscarPorLegajo(anyInt())).thenReturn(Optional.empty());

    // Act
    servicio.registrar("Ana", "García", "ana@mail.com", "Sistemas");

    // Assert — verificar que se llamaron los métodos correctos
    verify(repositorio).guardar(any(Estudiante.class));       // llamado 1 vez
    verify(notifServicio).enviarBienvenida(any(Estudiante.class)); // llamado 1 vez
    verify(repositorio, times(1)).guardar(any());              // exactamente 1 vez
    verify(repositorio, never()).eliminar(anyInt());           // nunca llamado
    verify(repositorio, atLeastOnce()).guardar(any());         // al menos 1 vez
    verify(repositorio, atMost(2)).guardar(any());            // máximo 2 veces

    // Verificar que no hubo más interacciones
    verifyNoMoreInteractions(repositorio);
}

@Test
void darDeBaja_EstudianteInexistente_NoLlamaNotificacion() {
    when(repositorio.eliminar(anyInt())).thenReturn(false);

    assertThrows(RuntimeException.class, () -> servicio.darDeBaja(9999));

    verify(notifServicio, never()).enviar(anyInt(), anyString());
}
```

---

## ArgumentCaptor — Capturar argumentos

```java
@Test
void registrar_NuevoEstudiante_GuardaConDatosCorrectos() {
    // Arrange
    ArgumentCaptor<Estudiante> captor = ArgumentCaptor.forClass(Estudiante.class);

    // Act
    servicio.registrar("Ana", "García", "ana@mail.com", "Sistemas");

    // Capturar el argumento pasado a guardar()
    verify(repositorio).guardar(captor.capture());
    Estudiante estudianteGuardado = captor.getValue();

    // Verificar los datos del objeto guardado
    assertEquals("Ana", estudianteGuardado.getNombre());
    assertEquals("García", estudianteGuardado.getApellido());
    assertEquals("ana@mail.com", estudianteGuardado.getEmail());
    assertEquals("Sistemas", estudianteGuardado.getCarrera());
    assertTrue(estudianteGuardado.isActivo());
}
```

---

## Spy — Objeto real con algunos métodos sobreescritos

```java
@Test
void spy_UsaImplementacionRealPeroPermiteMockearAlgunos() {
    // Spy sobre una implementación real
    EstudianteRepositorioEnMemoria repoReal = new EstudianteRepositorioEnMemoria();
    EstudianteRepositorioEnMemoria repoSpy = spy(repoReal);

    // La mayoría de métodos usan la implementación real
    // Solo sobreescribimos el que queremos controlar
    doReturn(Optional.empty())
        .when(repoSpy).buscarPorLegajo(9999);

    // Operación real funciona
    Estudiante e = new Estudiante("Ana", "García", "1", "a@b.com", 1001, "Sistemas");
    repoSpy.guardar(e);
    assertTrue(repoSpy.buscarPorLegajo(1001).isPresent()); // usa la implementación real

    // El método sobreescrito retorna lo que configuramos
    assertFalse(repoSpy.buscarPorLegajo(9999).isPresent());
}
```

---

## Ejemplo completo de Test con Mockito

```java
@ExtendWith(MockitoExtension.class)
@DisplayName("ServicioInscripcion Tests")
class ServicioInscripcionTest {

    @Mock private EstudianteRepositorio estudianteRepo;
    @Mock private CursoRepositorio cursoRepo;
    @Mock private NotificacionServicio notifServicio;

    @InjectMocks
    private ServicioInscripcion servicio;

    private Estudiante estudianteActivo;
    private Curso cursoConLugar;

    @BeforeEach
    void setUp() {
        estudianteActivo = new Estudiante("Ana", "García", "1", "a@b.com", 1001, "Sistemas");
        cursoConLugar = new Curso(1, "Matemática I", "MAT-001", 1, 1);
    }

    @Test
    @DisplayName("Inscripción exitosa envía notificación")
    void inscribir_TodoCorrecto_EnviaNotificacion() {
        when(estudianteRepo.buscarPorLegajo(1001)).thenReturn(Optional.of(estudianteActivo));
        when(cursoRepo.buscarPorCodigo("MAT-001")).thenReturn(Optional.of(cursoConLugar));

        servicio.inscribir(1001, "MAT-001");

        ArgumentCaptor<String> mensajeCaptor = ArgumentCaptor.forClass(String.class);
        verify(notifServicio).enviar(eq(1001), mensajeCaptor.capture());
        assertTrue(mensajeCaptor.getValue().contains("MAT-001"));
    }

    @Test
    @DisplayName("Inscripción falla si estudiante no existe")
    void inscribir_EstudianteNoExiste_LanzaExcepcion() {
        when(estudianteRepo.buscarPorLegajo(anyInt())).thenReturn(Optional.empty());

        assertThrows(RuntimeException.class, () -> servicio.inscribir(9999, "MAT-001"));

        verify(notifServicio, never()).enviar(anyInt(), anyString());
        verify(cursoRepo, never()).buscarPorCodigo(anyString());
    }
}
```

---

## Resumen Mockito

| Concepto | Código |
|---------|--------|
| Crear mock | `@Mock`, `mock(Clase.class)` |
| Inyectar mocks | `@InjectMocks` |
| Configurar retorno | `when(mock.metodo()).thenReturn(valor)` |
| Configurar excepción | `doThrow(ex).when(mock).metodo()` |
| Verificar llamada | `verify(mock).metodo(...)` |
| Capturar argumento | `ArgumentCaptor.forClass(Clase.class)` |

**Siguiente:** [8.3 TDD](03-tdd.md)
