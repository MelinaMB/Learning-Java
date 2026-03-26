# Ejercicios — Nivel 3: Java Intermedio

## Ejercicio 1 — Colecciones

**Enunciado:** Implementa `GestorCurso` con:
- `Map<Integer, Estudiante>` para almacenar estudiantes por legajo
- `Set<String>` para las materias disponibles
- Métodos: `inscribir`, `darDeBaja`, `buscar`, `listarAprobados()`, `promedioCurso()`

---

## Ejercicio 2 — Streams

**Enunciado:** Con la lista de estudiantes dada, usa Stream API para:
1. Filtrar los que tienen promedio >= 7 y carrera "Sistemas"
2. Obtener una lista de nombres en mayúsculas, ordenada alfabéticamente
3. Calcular el promedio del curso
4. Agrupar estudiantes por carrera y mostrar cuántos hay en cada una
5. Encontrar al estudiante con la nota más alta

---

## Ejercicio 3 — Excepciones Personalizadas

**Enunciado:** Crea:
- `InscripcionDuplicadaException` (checked): cuando se intenta inscribir a alguien ya inscripto
- `NotaFueraDeRangoException` (unchecked): nota < 0 o > 10
- `CursoInexistenteException` (checked): cuando el curso no existe

Úsalas en un servicio de inscripciones con validaciones completas.

---

## Ejercicio 4 — Optional

**Enunciado:** Refactoriza un `EstudianteRepositorio` que retorna `null` para
que use `Optional`. Actualiza el servicio para manejar los Optional con
`orElse`, `map`, `filter` e `ifPresent`.

---

## Ejercicio 5 — Fechas

**Enunciado:** Crea una clase `PeriodoAcademico` con:
- `fechaInicio` y `fechaFin` (LocalDate)
- `estaActivo()` — si hoy está dentro del período
- `getDiasRestantes()` — días hasta el fin
- `getDuracionEnSemanas()` — duración total en semanas
- `getFechaExamenFinal()` — 7 días antes de fechaFin
- Formato legible: "15/03/2024 al 30/11/2024"

---

## Checklist de Nivel 3

- [ ] ¿Sabes cuándo usar List vs Set vs Map?
- [ ] ¿Entiendes la diferencia entre checked y unchecked exceptions?
- [ ] ¿Puedes escribir un pipeline de Stream con filter, map y collect?
- [ ] ¿Entiendes qué es Optional y cuándo usarlo?
- [ ] ¿Sabes formatear y parsear fechas con DateTimeFormatter?
- [ ] ¿Puedes usar Collectors.groupingBy()?

Si puedes responderlas, avanza al [Nivel 4 — Java Avanzado](../nivel4-avanzado/README.md).
