# Guía Java — De 0 a Experto

Bienvenido a la guía más completa de Java en español. Esta guía está diseñada para llevarte
desde los primeros pasos con el lenguaje hasta conceptos avanzados de arquitectura de software.

## ¿Cómo usar esta guía?

1. **Sigue el orden de los niveles** — cada nivel construye sobre el anterior.
2. **Escribe el código tú mismo** — no copies y pegues; escribir el código a mano acelera el aprendizaje.
3. **Completa los ejercicios** — cada nivel tiene ejercicios prácticos al final.
4. **Relaciona los conceptos** — todos los ejemplos usan un sistema escolar (Estudiantes, Cursos, Profesores).

---

## Ruta de Aprendizaje

```
NIVEL 1            NIVEL 2            NIVEL 3            NIVEL 4
Fundamentos  -->   POO          -->   Intermedio   -->   Avanzado
(2-3 sem)          (3-4 sem)          (4-5 sem)          (4-5 sem)
     |
     v
NIVEL 5            NIVEL 6            NIVEL 7            NIVEL 8 & 9
Buenas       -->   Patrones     -->   Arquitectura -->   Testing &
Prácticas          de Diseño          (3-4 sem)          Ecosistema
(2-3 sem)          (3-4 sem)                             (3-4 sem)
```

---

## Tabla de Niveles

| Nivel | Tema | Tiempo Estimado | Prerrequisitos |
|-------|------|-----------------|----------------|
| 1 | Fundamentos de Java | 2-3 semanas | Ninguno |
| 2 | Programación Orientada a Objetos | 3-4 semanas | Nivel 1 |
| 3 | Java Intermedio | 4-5 semanas | Nivel 2 |
| 4 | Java Avanzado | 4-5 semanas | Nivel 3 |
| 5 | Buenas Prácticas | 2-3 semanas | Nivel 2+ |
| 6 | Patrones de Diseño | 3-4 semanas | Nivel 2, Nivel 5 |
| 7 | Arquitectura | 3-4 semanas | Nivel 6 |
| 8 | Testing | 3-4 semanas | Nivel 3 |
| 9 | Herramientas y Ecosistema | 2-3 semanas | Nivel 3+ |

**Tiempo total estimado:** 6 a 10 meses de estudio consistente (1-2 horas diarias).

---

## Proyecto Hilo Conductor

A lo largo de toda la guía construiremos juntos un **Sistema de Gestión Escolar**:

- `Estudiante` — datos personales, inscripciones, calificaciones
- `Profesor` — materias que dicta, carga horaria
- `Curso` — estudiantes inscritos, horarios, materias
- `Calificacion` — notas, promedios, aprobación
- `Inscripcion` — relación entre estudiante y curso

Esto te permitirá ver cómo los conceptos se aplican en un contexto coherente
y creciente a medida que avanzas de nivel.

---

## Recursos de apoyo

- [Glosario Java](glosario.md) — términos básicos y del ecosistema.
- [Errores Comunes](errores-comunes.md) — cosas que conviene evitar.
- [Guía de Estilo](guia-de-estilo.md) — cómo escribir ejemplos claros.
- [Depuración](depuracion.md) — cómo leer y resolver errores.
- [Guía de VS Code](vscode-setup.md) — IDE recomendado y extensiones.
- [Mapa de Aprendizaje](mapa-de-aprendizaje.md) — orden sugerido de estudio.
- [Autoevaluaciones](autoevaluaciones.md) — preguntas para medir progreso.
- [Repaso Rápido](repaso-rapido.md) — ruta corta de repaso.
- [Buenas Prácticas de Estudio](buenas-practicas-estudio.md) — cómo estudiar mejor.
- [Repaso en 7 Días](repaso-7-dias.md) — plan corto para retomar la guía.
- [Git para Java](git-para-java.md) — uso básico de Git en el learning.
- [Proyectos Finales](proyectos-finales.md) — mini metas por nivel y cierre general.
- [Soluciones Guiadas](soluciones-guiadas.md) — apoyo para comparar tu enfoque.

---

## Licencia

Este material se publica bajo licencia `MIT`. Puedes reutilizarlo y compartirlo siempre que conserves la atribución al repositorio.

---

## Requisitos Previos

- **Java JDK 17 o superior** instalado ([Descargar](https://adoptium.net/))
- **Un IDE** — preferimos VS Code con las extensiones recomendadas en la guía
- **Ganas de aprender** — la constancia es más importante que el talento

---

## Convenciones de esta Guía

> Los bloques con este formato son **consejos importantes** o aclaraciones.

```java
// Los bloques de código tienen sintaxis resaltada
// y están listos para ejecutarse
public class Ejemplo {
    public static void main(String[] args) {
        System.out.println("¡Hola, Java!");
    }
}
```

> **Error común:** Los bloques con esta etiqueta indican errores frecuentes a evitar.

---

## Comenzar

Listo para empezar? Ve al [Nivel 1 — Fundamentos](nivel1-fundamentos/README.md) y da tu primer paso.
