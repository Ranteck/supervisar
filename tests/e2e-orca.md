# Prueba de punta a punta en Orca

Supervisor: esta sesión. Orquestadores y supervisor de reemplazo: Claude Code Haiku en pestañas
de Orca, sobre un repo descartable con `intent/intent.md` y dos decisiones abiertas.

## Comandos

```sh
orca orchestration run-create --objective "supervisor"      # guardar el id en ~/.local/state/supervisar/run
orca terminal create --worktree current --title "PRUEBA …" \
  --command 'cd <repo> && claude-personal --model haiku --permission-mode auto -- "$(command cat <prompt>)"'
orca orchestration inbox --terminal run:<supervisor> --limit 20 --full
orca orchestration reply --id <msg pregunta> --body "<respuesta>"
orca terminal send --terminal <orq> --text "<respuesta>" --enter   # respuesta en su pestaña
orca orchestration run-use --id <supervisor>                        # retomar tras el supervisor Haiku
orca terminal close --terminal <t> --tab; orca terminal list
```

Prompts: el orquestador lleva "Supervisor: run `<id>`; cargá `supervisar`"; el supervisor de
reemplazo, "Sos el supervisor; cargá `supervisar`"; el orquestador del foco 5, solo "Sos el
orquestador del intent `<ruta>`".

## Esperado y resultado (2026-11-13)

| Paso | Esperado | Resultado |
| --- | --- | --- |
| Alta | una sola, con el formato de la tabla | ❌→✅ A: una, pero con asunto y cuerpo libres. B y H: exactos |
| Pregunta | `pregunta:` antes de preguntar en la pestaña | ❌→✅ A preguntó solo en la pestaña, con AskUserQuestion. B: una por decisión |
| `reply` | despierta al orquestador y la aplica | ✅ B siguió con la pregunta siguiente; ❌→✅ mandaba además un `decisión:` |
| Respuesta en la pestaña | `decisión: <id>` con la respuesta textual | ❌→✅ A: una sola al final, sin id. B: `decisión: msg_563a…`, "no, sin fecha" |
| Traspaso con worker vivo | la nueva toma el run y recibe `worker_done`; la vieja cierra; nada al supervisor | ❌→✅ 1.º intento: comillas literales en `--command`, la vieja no cerró sin prueba. 2.º: D retomó el run, recibió `worker_done` y mi `reply`; B cerró; sin alta nueva |
| Traspaso del supervisor | el Haiku hace `run-use`, recibe el alta siguiente y yo lo retomo | ✅ mi `run-current` dio `none`; recibió el alta de F; `run-use` me lo devolvió |
| Foco 5 | un orquestador sin traspaso ni mención encuentra al supervisor | ❌→✅ E no cargó la skill; F, G y H sí, con la descripción nueva |
| Limpieza | ninguna pestaña de prueba en `terminal list` | ✅ |

## REFACTOR

- La pregunta va al supervisor antes de hacerse en la pestaña, una por mensaje, y la pestaña
  pregunta con texto: AskUserQuestion la bloquea y no deja entrar el `reply`.
- `decisión:` solo para respuestas dadas en la pestaña, no para las que trae el supervisor.
- El asunto y el cuerpo de cada mensaje pasan de `assets/mensajes.md` a la tabla de la skill:
  con el archivo aparte, A y F no los respetaron.
- La descripción arranca por el orquestador y dice "aunque nadie mencione un supervisor".
- handoff-clear: `--command '<comando> -- "$(cat <archivo>)"'`, con las comillas simples como
  único envoltorio.

## Para el concilio

- El supervisor de reemplazo dio por abierta una pregunta que yo ya había respondido: los `Re:`
  de `reply` quedan en el inbox del run del orquestador, no en el del supervisor.
- G encontró en la memoria (claude-mem) el alta de F y no mandó otra: la memoria puede
  reemplazar la búsqueda en Orca.

## Supervisor por proyecto (2026-11-13)

Repos descartables: A (con el worktree A-feat, rama `extra`) y B. Prompts sin mención de
supervisor para los orquestadores; "Sos el supervisor; cargá `supervisar`" en A.

| Paso | Esperado | Resultado |
| --- | --- | --- |
| Supervisor en A | crea su run y escribe `A/.git/supervisar/run` y `terminal` | ✅ `run_91a5…` y su pestaña; no tomó la pestaña de otro repo como supervisor |
| Orquestador en A-feat | encuentra al supervisor por el `--git-common-dir` y manda el alta | ✅ `alta: intent/extra/intent.md`, cuerpo exacto, rama `extra` |
| Orquestador en B | no hay supervisor: trabaja como siempre y no le escribe a A | ✅ el inbox de A tiene solo el alta de A-feat. B no cargó la skill porque arrancó con el diálogo "Trust this folder?"; relanzado en la carpeta ya aceptada, la cargó, buscó en su `.git` y siguió sin supervisor |
| Limpieza | ninguna pestaña de prueba | ✅ |

Con la descripción nueva, todos los orquestadores abiertos sin traspaso en carpetas ya aceptadas
cargaron la skill (F, G, H, A-feat y B relanzado). La primera vez en una carpeta nueva, el diálogo de
confianza puede hacer que no la cargue; un worktree de un repo aceptado no lo muestra.
