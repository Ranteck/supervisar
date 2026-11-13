---
name: supervisar
description: Usar SIEMPRE al arrancar o retomar como orquestador de un intent ("sos el orquestador", "orquestá", un `intent/…/intent.md`), aunque nadie mencione un supervisor, para saber si hay uno y reportarle. También cuando el usuario pide un supervisor u observador central ("supervisá", "sos el supervisor", "coordiná los orquestadores") o un prompt trae "Supervisor: run".
---

# Supervisar

Cada proyecto (un repo con todos sus worktrees) tiene a lo sumo un supervisor, que coordina sus
orquestadores por Orca. Su estado vive en
`$(git rev-parse --path-format=absolute --git-common-dir)/supervisar/`, el mismo para todos los
worktrees del repo: el archivo `run` tiene el id de su run permanente, que es su dirección, y
`terminal` tiene su pestaña.

## Si sos el supervisor

Corrés en una carpeta de tu proyecto y solo ves ese proyecto.

1. **Al arrancar:** si tu prompt no trae `run-use --id` (no venís de un traspaso) y el archivo
   `terminal` nombra otra pestaña que sigue en `orca terminal list`, hay
   otro supervisor vivo: preguntale al usuario si lo reemplazás y, si dice que no, frená. Después,
   si el archivo `run` existe, `orca orchestration run-use --id <id>`. Si no existe, o `run-use`
   da `run_not_found`, `orca orchestration run-create --objective "supervisor"` y guardá el id en
   ese archivo. Por último, guardá tu `$ORCA_TERMINAL_HANDLE` en el archivo `terminal`.
2. **Altas, decisiones y cierres:** no respondas; quedan en el inbox.
3. **Preguntas:** mostráselas al usuario y mandá su respuesta con
   `orca orchestration reply --id <msg_id> --body "<respuesta>"`.
4. **Resumen, a pedido:** con `assets/resumen.md`, desde
   `orca orchestration inbox --terminal run:<id> --limit 100` de tu run y del run de cada alta (ahí
   quedan los `Re:` de las respuestas que mandaste), `orca worktree ps` y el `intent/` de tu
   proyecto. Un orquestador con `cierre:` está cerrado; si no, está vivo si la pestaña de su último
   mensaje sigue en `orca terminal list`, y si no está ahí, pero su rama tiene sesiones
   vivas, es un posible traspaso (la pestaña nueva no avisa hasta su próximo mensaje). Una pregunta con
   `Re:` o `decisión:` no está abierta. Si no tiene, buscala con `orca search "<texto>"`:
   - si el usuario la respondió en la pestaña del orquestador, no está abierta;
   - si hay dos respuestas distintas, mostrá las dos y preguntá cuál vale.
5. **Ideas nuevas:** decí a qué intent de tu proyecto van o si piden uno nuevo, y proponé el paso
   siguiente; si son de otro proyecto, decilo y proponé llevarlas allá. Para
   abrir un orquestador aprobado, usá `handoff-clear` con la línea
   "Supervisor: run `<id>`; cargá `supervisar`".
6. Todo se reconstruye de esas fuentes: no guardes notas propias. Nada se lanza, cierra ni
   redirige sin aprobación del usuario; con su OK, una instrucción a un orquestador va con
   `orca orchestration send --to run:<su run> --subject "…" --body "…"`.

## Si reportás a uno

Hay supervisor si el prompt trae "Supervisor: run `<id>`" o si existe el archivo `run` de tu
repo; tu memoria no cuenta. Si no hay, trabajá como siempre y no crees uno.

1. **Tu run:** si `orca orchestration run-current` da `none`,
   `orca orchestration run-create --objective "<intent>"`. Es tu único run: los workers y concilios
   van en él, porque el traspaso se lleva un solo run.
2. **Mensajes:** solo estos cuatro, con
   `orca orchestration send --to run:<supervisor> --subject "…" --body "…"`, sin `--run`, con
   este asunto y cuerpo exactos:

   | Cuándo | `--subject` | `--body` |
   | --- | --- | --- |
   | al arrancar, o al retomar sin traspaso, aunque tu memoria diga que otra sesión ya lo dio de alta | `alta: <ruta del intent>` | `proyecto: <ruta> · rama: <rama> · run: <tu run>` |
   | cada decisión que le pedís al usuario, una por mensaje, antes de hacérsela en tu pestaña | `pregunta: <la pregunta, en una línea>` | `opciones: <a, b, …> · contexto: <archivo donde se decide>` |
   | apenas el usuario responde en tu pestaña, una por pregunta | `decisión: <id del mensaje de la pregunta>` | `<respuesta del usuario, textual>` |
   | terminaste o abandonaste el intent | `cierre: <ruta del intent>` | `resultado: terminado o abandonado · <commit o motivo>` |

   En tu pestaña, preguntá con texto y terminá el turno, sin AskUserQuestion: bloquea la pestaña y
   no deja entrar la respuesta del supervisor.
3. Si un envío da `run_not_found`, seguí sin supervisor y avisale al usuario.
4. La respuesta del supervisor te llega como mensaje y te despierta: aplicala, sin mandar
   `decisión`. Si en tu pestaña el usuario respondió otra cosa, no apliques ninguna y preguntale
   cuál vale.
5. **Traspaso:** con `handoff-clear`; el prompt lleva "Supervisor: run `<id>`; cargá
   `supervisar`" y los ids de las preguntas sin respuesta. Al supervisor no le avisás.

## Contexto

Al terminar cada mensaje (supervisor) o cada tarea (orquestador), corré
`python3 ~/.claude/skills/handoff-clear/scripts/contexto`. Con 30 % (supervisor) o 40 %
(orquestador) o más, proponé el traspaso con `handoff-clear`. El supervisor pasa su run como la
orquestación en curso (el prompt lleva `run-use --id <tu run>`) y dice "Sos el supervisor; cargá
`supervisar`" en vez de la línea "Supervisor: run".
