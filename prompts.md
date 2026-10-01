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

**Modelo:** Opus 5.5 Extra
**Herramienta:** Claude Code

```
analiza todo el contenido del repositorio y analiza desde el punto de vista funcional de la aplicación que contiene  qué capabilities hay ya construidas y cómo es el modelo de datos actual. Debe servir para qué sepas que está ya construido
No realices ningún cambio.
```

**Qué salió:** claude ya tiene contexto del proyecto previo a la solicitud que le haré de levantar la spec

## Prompt 2

**Modelo:** Opus 5.5 Extra
**Herramienta:** Claude Code

```
Ahora, debes escribir la spec de lo que el sistema hace hoy y déjala en un archivo versionado del proyecto ruta "docs/spec-viva/mgs.md", no en el chat.
Debes enfocarte exclusivamente en las dos capas (Api y web), y el vertical de cuentas y acceso. Nada más que eso. 
Formato de la especificación (no es negociable):

- Arriba, un ## Purpose de una o dos frases: para qué existe esta capability.
- Debajo, ## Requirements, y colgando de él ### Requirement: en los que el sistema SHALL hacer algo.
- Bajo cada requisito, al menos un #### Scenario: de cuatro almohadillas, con dos viñetas: **WHEN** y **THEN**. No hay casilla para el GIVEN: la precondición se mete dentro del WHEN.
- En castellano, salvo las mayúsculas de la RFC.
- Y tres reglas duras, que son las que separan este ejercicio de escribir documentación:
- Nada de ADDED, MODIFIED ni REMOVED. Eso es el vocabulario de un delta, y esto no es un delta: es la verdad actual del sistema. Si tu archivo tiene una de esas secciones, has escrito otra cosa.
- Solo comportamiento observable desde fuera. Ni un nombre de clase, ni un nombre de archivo, ni una ruta de código. En la API, observable es la petición y la respuesta. En la pantalla, observable es lo que una persona ve y puede hacer.
- No toques el código. 
```

**Qué salió:** generó el documento con 22 requisitos y 65 escenarios siguiendo el formato solicitado

## Prompt 3

**Modelo:** Opus 5.5 Extra
**Herramienta:** Claude Code

```
genera un tabla resumen donde muestres para cada requisito identificado las ubicaciones clave del código fuente donde puedo encontrar la evidencia de la implementación de cada uno de ellos.
```

**Qué salió:** tabla resumen de los requisitos y evidencia de código fuente que facilitó la revisión y detección de inconsistencias
