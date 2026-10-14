# Prompts

Aquí van **todos los prompts que lanzaste** para hacer el ejercicio, en el orden en que los
lanzaste, con el modelo y la herramienta de cada uno.

Esto no es papeleo. Lo que se revisa es **cómo pediste las cosas**, no solo lo que salió: un
resultado flojo con un prompt bueno y un resultado flojo con un prompt vago necesitan feedback
distinto, y sin este archivo no se distinguen.

## Cómo rellenarlo

- Un apartado `## Prompt N` por cada prompt.
- **Pega el prompt tal cual lo lanzaste**, dentro del bloque de código, aunque ocupe diez líneas
  y aunque tenga faltas. No lo reescribas para que quede bien: el que arreglaste mentalmente
  después no es el que lanzaste.
- Incluye también los que **no funcionaron**. Suelen ser los más útiles de leer.
- `Modelo` y `Herramienta` en todos. Si cambiaste de una a otra a mitad, se nota aquí.

Borra el ejemplo de abajo cuando escribas el primero.

---

## Prompt 1

**Modelo:** Opus 5.5 Medium
**Herramienta:** Claude Code

```text

resume los últimos commits asociados al branch actual

```

**Qué salió:** 14 commits con sus descripciones, agrupados por tema y este resumen: En resumen, la rama parte del trabajo de la sesión 4 (API documentada, ADRs, el arreglo del email expuesto y reglas de proceso) y encima solo añade el enunciado del ejercicio de la sesión 5. Todavía no hay código de esta sesión.

## Prompt 2

**Modelo:** Opus 5.5 Medium
**Herramienta:** Claude Code

```text

lista los últimos 15 commits en una tabla con su descripción

```

**Qué salió:** tabla con los 15 commits con su hash, fecha, autor, tipo y descripción.

## Prompt 3

**Modelo:** Opus 5.5 Medium
**Herramienta:** Claude Code

```text

Ahora, eres un auditor de código fuente y metodología de desarrollo. Revisarás los 15 commits consultados y los vas a contrastar contra las reglas de proceso descritas en Claude.md
El formato de la auditoría será en una tabla contemplando las siguientes columnas por cada fila:
- Qué pide la regla: Resúmela en una frase y con tus propias palabras. no mezcles más de una regla por fila.
- Qué se revisó para comprobarla: Indica el comando ejecutado o el archivo consultado. Es indispensable para justificar el veredicto.
- Veredicto: Solo puede ser se CUMPLE, NO SE CUMPLE o NO SE PUEDE COMPROBAR con la información disponible.
- Evidencia: Escríbela en una línea. Si la regla no se cumple, indica dónde se incumple. Si no se puede comprobar, explica qué información falta.
- Qué harías con ella: deja esta columna en blanco, yo la completaré

Esto es una auditoría, no una tarea de corrección. No modifiques ningún archivo. 
Guarda el resultado en docs/auditoria/MGS.md, no en el chat.

```

**Qué salió:** documento generado con la información solicitada y un resumen de los hallazgos en consola
