# Parte A: La Auditoría

## Expectativa

Regla y cumple:

1. Rama por unidad de trabajo: No
2. Commit por petición: Si
3. Cambio que toque rutas, controladores, etc se cierra mismo commit con docs: No
4. Creación PR con adversarial reviewer: No
5. Al abrir PR no repetir resumen en consola: Si

## Auditoría de proceso: últimos 15 commits de `s5/start`

| # | Qué pide la regla | Qué se revisó para comprobarla | Veredicto | Evidencia | Qué harías con ella |
|---|---|---|---|---|---|
| 1 | Antes de tocar código, mirar en qué rama se está y, si ya es una rama de trabajo, seguir en ella. | `git reflog`; `git branch -a` | NO SE PUEDE COMPROBAR | Git no registra qué rama se consultó antes de editar: el reflog local solo muestra el clon y dos checkouts, y harían falta las transcripciones de las sesiones. | Convertir en comprobación automática |
| 2 | Las ramas de trabajo nuevas se crean solo desde `main` o una `sN/*` y se llaman `feat/<slug>`. | `git branch -a`; `git log -n 15 --merges` | NO SE PUEDE COMPROBAR | No existe ninguna rama `feat/*` en local, en `origin` ni en `upstream`, ni hay merges que la nombren; si se usó y luego se borró, no queda rastro. | Convertir en comprobación automática |
| 3 | Nunca commitear directamente en `main` ni en una rama `sN/*`. | `git log -n 15 --merges`; `git branch -a --contains <hash>`; `git log --format='%h %P %an %cn'` | NO SE CUMPLE | Los 15 commits forman una cadena lineal sin merges y solo están en `s4/end` y `s5/start`; quedaron directamente sobre ramas `sN/*`, incluidos los 12 de trabajo (`4427afc` a `cbe9617`). | Convertir en comprobación automática |
| 4 | Hacer un commit por cada petición, al cerrarla. | `git log -n 15 --format='%h %ad %s' --date=iso` | NO SE PUEDE COMPROBAR | El historial no guarda qué petición originó cada commit; para cruzarlos harían falta las transcripciones de las sesiones. | Borrarla |
| 5 | Crear cada commit con la skill `/commit`. | `.claude/skills/commit/SKILL.md`; `git log -n 15 --format='%h %s'` | NO SE PUEDE COMPROBAR | Git no registra la herramienta con que se hizo un commit; 14 de 15 asuntos siguen el formato `tipo(scope): descripción` de la skill, pero `eff1686` no tiene scope. | Dejarla |
| 6 | Si un commit cambia rutas, controladores, validadores o transformers de una capability, en ese mismo commit se actualizan su documento OpenAPI y su README. | `git log -- backend/start/routes.ts backend/app/{controllers,validators,transformers}`; `git show --stat 4e03d1f cbe9617`; `git ls-tree 4e03d1f^ docs/capabilities` | NO SE CUMPLE | `4e03d1f` cambia `task_transformer.ts` sin README ni OpenAPI de `tasks`; aún no existían, y la regla se añadió después, en `160fbc7`. `cbe9617` sí la cumple. | Convertir en comprobación automática |
| 7 | Commitear el diff regenerado de `backend/.adonisjs/` cuando un cambio lo modifica. | `git show --stat ea5db1f 4e03d1f cbe9617`; `backend/.adonisjs/client/registry/schema.d.ts` | CUMPLE | `ea5db1f` añade rutas e incluye el diff de `.adonisjs/`; `4e03d1f` y `cbe9617` no alteran el registro, porque este solo referencia los tipos de controladores y validadores mediante `import(...)`. | Dejarla |
| 8 | Abrir un único PR, con la descripción completa, al terminar la unidad de trabajo. | `git ls-remote <origin> 'refs/pull/*'`; `git fetch upstream 'refs/pull/*/head'` y comparar los heads de los PR con los 15 hashes | NO SE CUMPLE | `origin` no tiene ningún PR; en `upstream` ningún PR tiene como head uno de los 15 commits, y los que los contienen (#85 a #91) son PRs posteriores abiertos sobre `eff1686`. | Convertir en comprobación automática |
| 9 | Pasar el subagente `adversarial-reviewer` sobre ese PR como último paso de la unidad. | `.claude/agents/adversarial-reviewer.md`; `refs/pull/*` de `origin` y `upstream` | NO SE PUEDE COMPROBAR | El subagente es de solo lectura y no deja rastro en git; sin un PR de estos commits no hay comentarios que revisar, y harían falta las transcripciones de las sesiones. | Convertir en comprobación automática |
| 10 | Al abrir el PR, responder en el chat solo con su URL, sin repetir el resumen. | Ninguna fuente disponible: no hay transcripciones de las sesiones | NO SE PUEDE COMPROBAR | Las respuestas de Claude en el chat no quedan en el repo; harían falta los registros de las sesiones de Claude Code. | Dejarla |

## Parte B: las tres líneas

**La regla en la que más te equivocaste:**

Compara tu expectativa inicial con el veredicto final y elige la que más se alejó de lo que pensabas. Si acertaste todas, indícalo y explica en una frase en qué te apoyaste para acertar. Si solo era una impresión, también conviene dejarlo por escrito.

**Una regla que no se puede comprobar con la información disponible:**

El pase del adversarial-reviewer, ya que para comprobarlo  harían falta las transcripciones de las sesiones de Claude Code. Se podría corregir aplicandolo de forma obligatoria en un workflow CI.

**La regla que eliminarías y por qué:**

Nunca commitear directamente en main ni en una rama sN/* ya que se puede automatizar con branch protection de github.
