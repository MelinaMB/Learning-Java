# Ejercicios — Nivel 4: Java Avanzado

## Ejercicio 1 — Concurrencia

**Enunciado:** Implementa un `ProcesadorMasivo` que use un pool de 4 hilos para
procesar 100 estudiantes en paralelo. Cada "procesamiento" consiste en calcular
el promedio y guardarlo. Usa `ExecutorService` y asegúrate de manejar las
interrupciones correctamente.

---

## Ejercicio 2 — IO/NIO

**Enunciado:** Crea un `ImportadorEstudiantes` que:
1. Lea un CSV con la estructura: `legajo,nombre,apellido,carrera,promedio`
2. Convierta cada fila en un objeto `Estudiante`
3. Genere un reporte en un archivo de texto con estadísticas (total, promedio, aprobados)
4. Use `Files.lines()` con Stream API para el procesamiento

---

## Ejercicio 3 — Anotaciones y Reflexión

**Enunciado:** Crea la anotación `@CampoRequerido(String etiqueta)` y un
`ValidadorObjeto` que use reflexión para verificar que todos los campos
anotados con `@CampoRequerido` no sean null ni vacíos en cualquier objeto que reciba.

---

## Checklist de Nivel 4

- [ ] ¿Entiendes la diferencia entre `Thread`, `Runnable` y `Callable`?
- [ ] ¿Sabes por qué usar `ExecutorService` en lugar de crear hilos directamente?
- [ ] ¿Puedes leer y escribir archivos CSV con NIO.2?
- [ ] ¿Entiendes qué es la reflexión y cuándo (no) usarla?
- [ ] ¿Puedes crear y procesar anotaciones personalizadas?

Avanza al [Nivel 5 — Buenas Prácticas](../nivel5-buenas-practicas/README.md).
