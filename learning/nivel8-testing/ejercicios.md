# Ejercicios — Nivel 8: Testing

## Ejercicio 1 — JUnit 5 básico

**Enunciado:** Escribe tests unitarios para la clase `Estudiante` verificando:
- `getNombreCompleto()` retorna nombre + espacio + apellido
- `estaAprobado()` retorna true para promedio >= 6.0
- `setPromedio()` lanza `IllegalArgumentException` para valores fuera de [0, 10]
- `toString()` contiene el legajo y el nombre

---

## Ejercicio 2 — Tests parametrizados

**Enunciado:** Crea tests parametrizados para una clase `ClasificadorNota`
con el método `String clasificar(double nota)` que retorna
"Sobresaliente", "Notable", "Bien", "Suficiente", "Insuficiente".
Prueba al menos 10 valores incluyendo límites de cada rango.

---

## Ejercicio 3 — Mockito

**Enunciado:** Escribe tests para `EstudianteServicioImpl` usando mocks para:
1. `registrar()` — verifica que se llama `guardar()` en el repositorio
2. `buscar()` — verifica que retorna Optional.empty() cuando el repo no encuentra
3. `darDeBaja()` — verifica que lanza excepción cuando no existe y que no notifica

---

## Ejercicio 4 — TDD

**Enunciado:** Desarrolla la clase `CalculadorBecas` usando TDD.
Reglas:
- Promedio >= 9.0: beca del 100%
- Promedio >= 8.0: beca del 50%
- Promedio >= 7.0: beca del 25%
- Promedio < 7.0: sin beca

Escribe los tests primero, luego la implementación, luego refactoriza.

---

## Checklist de Nivel 8

- [ ] ¿Sabes qué hace `@BeforeEach` y para qué sirve?
- [ ] ¿Entiendes la diferencia entre `assertEquals` y `assertSame`?
- [ ] ¿Puedes crear mocks con Mockito y configurar su comportamiento?
- [ ] ¿Entiendes qué verifica `verify(mock).metodo(...)`?
- [ ] ¿Puedes explicar el ciclo Red-Green-Refactor con tus palabras?
- [ ] ¿Sabes cómo capturar argumentos con `ArgumentCaptor`?

Avanza al [Nivel 9 — Herramientas y Ecosistema](../nivel9-ecosistema/README.md).
