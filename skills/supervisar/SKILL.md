---
name: supervisar
description: Usar cuando el usuario pide un supervisor u observador central ("supervisá", "sos el supervisor", "coordiná los orquestadores"), cuando un prompt trae "Supervisor: run", o cuando una sesión de Claude Code en Orca arranca o retoma un intent como orquestador.
---

# Supervisar

Un supervisor central recibe de los orquestadores solo altas, preguntas, decisiones y cierres; el
resto lo lee de Orca y de `intent/` cuando hace falta. Su dirección es un run de Orca permanente,
cuyo id está en `~/.local/state/supervisar/run`.

## Si sos el supervisor

1. **Al arrancar:** si el archivo existe, `orca orchestration run-use --id <id>`. Si no existe, o
   `run-use` da `run_not_found`, `orca orchestration run-create --objective "supervisor"` y guardá
   el id en el archivo.
2. **Altas y cierres:** no respondas; quedan en el inbox.
3. **Preguntas:** mostráselas al usuario y mandá su respuesta con
   `orca orchestration reply --id <msg_id> --body "<respuesta>"`.
4. **Resumen, a pedido:** con `assets/resumen.md`, desde
   `orca orchestration inbox --terminal run:<id> --limit 100`, `orca worktree ps` (un orquestador
   está vivo si su proyecto y rama tienen sesiones vivas) y el `intent/` de cada proyecto. Antes de
   listar una pregunta como abierta, buscala con `orca search "<texto>"`:
   - si el usuario la respondió en la pestaña del orquestador, no está abierta;
   - si hay dos respuestas distintas, mostrá las dos y preguntá cuál vale.
5. **Ideas nuevas:** decí a qué intent van o si piden uno nuevo, y proponé el paso siguiente. Para
   abrir un orquestador aprobado, usá `handoff-clear` con la línea
   "Supervisor: run `<id>`; cargá `supervisar`".
6. Todo se reconstruye de esas fuentes: no guardes notas propias. Nada se lanza, cierra ni
   redirige sin aprobación del usuario; con su OK, una instrucción a un orquestador va con
   `orca orchestration send --to run:<su run> --subject "…" --body "…"`.
7. Al terminar de atender cada mensaje, corré
   `python3 ~/.claude/skills/handoff-clear/scripts/contexto`. Si da 30 % o más, proponé el
   traspaso con `handoff-clear`: tu run es la orquestación en curso que la pestaña nueva retoma, y
   el prompt dice "Sos el supervisor; cargá `supervisar`" en vez de la línea "Supervisor: run".

## Si reportás a uno

Hay supervisor si el prompt trae "Supervisor: run `<id>`" o si existe el archivo. Si no, trabajá
como siempre y no crees uno.

1. **Tu run:** si `orca orchestration run-current` da `none`,
   `orca orchestration run-create --objective "<intent>"`. Es tu único run: los workers y concilios
   van en él, sin otro `run-create`, porque el traspaso se lleva un solo run.
2. **Mensajes:** solo estos cuatro, con
   `orca orchestration send --to run:<supervisor> --subject "<tipo>: …" --body "…"`, sin `--run`,
   y con el formato de `assets/mensajes.md`.

   | Tipo | Cuándo |
   | --- | --- |
   | `alta` | al arrancar, o al retomar sin traspaso |
   | `pregunta` | necesitás una decisión del usuario; también se la hacés en tu pestaña |
   | `decisión` | el usuario respondió en tu pestaña |
   | `cierre` | terminaste o abandonaste el intent |

   El supervisor lee el avance de Orca y de `intent/` cuando lo necesita.
3. Si el alta da `run_not_found`, seguí sin supervisor y avisale al usuario.
4. La respuesta del supervisor te llega como mensaje y te despierta: aplicala.
5. **Traspaso:** con `handoff-clear`, y el prompt lleva "Supervisor: run `<id>`; cargá
   `supervisar`". La pestaña nueva retoma tu run y el supervisor no recibe nada.
6. Al cerrar cada tarea, corré `python3 ~/.claude/skills/handoff-clear/scripts/contexto`. Si da
   40 % o más, proponé el traspaso.
