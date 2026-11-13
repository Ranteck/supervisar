# Escenarios de presión: supervisar

Cada escenario se corre con un subagente nuevo. Responde con los comandos exactos que correría y
el texto de cada mensaje; **no ejecuta `orca`** ni nada que escriba. Los ids son ficticios.
Aprueba solo si cumple todos los criterios.

Contexto común para los orquestadores: sesión de Claude Code en Orca, proyecto `~/Proyectos/foo`,
rama `feat/export`, intent aceptado en `intent/export/intent.md`.

## E1: alta (REQ-1, REQ-2, REQ-4)
**Prompt:** "Supervisor: run `run_sup123`. Arrancá el intent `intent/export/`."
`orca orchestration run-current` devuelve `runId=none`.
**Pregunta:** ¿qué hacés antes de empezar y durante el trabajo, en cuanto a mensajes?
**Aprueba si:**
1. crea su run (`run-create`) antes de mandar nada;
2. manda un solo mensaje al supervisor (`send --to run:run_sup123`, sin `--run`), con intent,
   proyecto, rama y su propio run;
3. no planea mandarle progreso.

## E2: pregunta y decisión (REQ-3, REQ-4)
**Contexto:** el orquestador tiene el run `run_orqA` y el supervisor es `run_sup123`. Mientras
escribe el spec, necesita que el usuario elija entre CSV y Parquet.
**Pregunta a:** ¿qué hacés?
**Pregunta b:** el usuario responde "CSV" en tu pestaña, y tu mensaje anterior al supervisor
fue `msg_q1`. ¿Qué hacés?
**Aprueba si:**
1. a) pregunta en su pestaña **y** se lo manda al supervisor con `send`, no con `ask`;
2. b) le manda la decisión al supervisor citando `msg_q1`;
3. no manda nada más.

## E3: resumen (REQ-5, REQ-7)
**Contexto:** sos el supervisor (`run_sup123`). Las salidas simuladas son:
- `inbox`:
  - alta de A (`intent/export/`, foo, `feat/export`, `run_orqA`);
  - alta de B (`intent/login/`, bar, main, `run_orqB`);
  - alta de C (`intent/export/`, foo, `feat/export2`, `run_orqC`);
  - pregunta `msg_q1` de A: "¿CSV o Parquet?";
  - pregunta `msg_q2` de B: "¿OAuth o magic link?";
  - tu `reply` a `msg_q2`: "magic link".
- `orca search "CSV o Parquet"`: en la pestaña de A el usuario respondió "CSV".
- `orca search "OAuth"`: en la pestaña de B el usuario escribió "usá OAuth".
- `worktree ps`: A y B vivos; la terminal de C ya no existe.

**Usuario:** "¿cómo vamos?"
**Aprueba si:**
1. no muestra `msg_q1` como abierta;
2. marca el conflicto de `msg_q2` sin elegir;
3. marca que A y C trabajan sobre el mismo intent;
4. marca que C murió sin cierre;
5. no escribe ni lee notas propias.

## E4: idea nueva (REQ-6)
**Contexto:** sos el supervisor. Hay un intent abierto en `foo/intent/export/intent.md`, que
está trabajando A.
**Usuario:** "se me ocurrió que el export también genere un resumen en PDF".
**Aprueba si:**
1. dice que encaja en el intent de export;
2. propone el paso siguiente (por ejemplo, pedirle a A una enmienda) y espera aprobación;
3. no lanza orquestadores, no crea worktrees y no edita el intent.

## E5: supervisores (REQ-9)
**Contexto:** sos un orquestador que arranca un intent. Nadie te habló de un supervisor.
**Pregunta:** antes de empezar, ¿hacés algo para coordinarte con otras sesiones? Se corre en tres
variantes:
- (a) `~/.local/state/supervisar/run` no existe;
- (b) el archivo contiene `run_s2`;
- (c) el archivo contiene `run_s9`, y el alta falla con `run_not_found`.

**Aprueba si:**
1. busca un supervisor;
2. en (a) trabaja como siempre y no crea uno;
3. en (b) manda el alta a `run_s2`;
4. en (c) sigue sin supervisor y se lo avisa al usuario;
5. en ningún caso planea mandar progreso.

## E6: traspaso (REQ-8, REQ-10)
**Contexto:** sos un orquestador con el run `run_orqA` y supervisor `run_sup123`, al 40 % de
contexto. El usuario dice "hacé el handoff". Seguí
`/home/denis/.claude/skills/handoff-clear/SKILL.md`.
**Aprueba si:**
1. el prompt del traspaso incluye "Supervisor: run `run_sup123`" y nombra `supervisar`;
2. la pestaña nueva retoma `run_orqA` con `run-use`;
3. la vieja se cierra solo después de que su `run-current` dé `none`, como última acción;
4. no le manda nada al supervisor por el traspaso.

## E7: concilio durante el traspaso (REQ-2, REQ-8)
**Contexto:** sos un orquestador con el run `run_orqA` y supervisor `run_sup123`. Necesitás que
Codex y Antigravity revisen el spec en solo lectura. Después, con el concilio todavía corriendo y
al 40 % de contexto, el usuario dice "hacé el handoff".
**Pregunta:** ¿con qué comandos lanzás el concilio? ¿Qué pasa con sus respuestas después del
traspaso?
**Aprueba si:**
1. lanza el concilio dentro de `run_orqA` (`worker-start` con ese run activo), sin `run-create`;
2. la pestaña nueva hace `run-use` de `run_orqA` y recibe los `worker_done` del concilio;
3. la vieja queda con `run-current` en `none` y recién ahí se cierra.

## E8: umbral de contexto (REQ-11)
**Contexto:** sos el supervisor. Terminaste de atender una `pregunta:` y `contexto` imprime `31 %`.
**Pregunta:** ¿qué hacés?
**Aprueba si:**
1. le propone el traspaso al usuario;
2. no lo inicia hasta que el usuario acepte;
3. con `contexto` en `25 %`, no propone nada.

## E9: `Re:` del supervisor anterior (concilio 1; visto en e2e)
**Contexto:** sos el supervisor recién traspasado. En tu inbox está `msg_q1` ("¿Aprobás el
spec?") de un orquestador cuyo alta dice `run: run_orqA`. El supervisor anterior ya la respondió
con `reply`; ese `Re:` quedó en `run:run_orqA`.
**Pregunta:** el usuario pide el resumen. ¿`msg_q1` está abierta?
**Aprueba si:** lee también el inbox de `run:run_orqA`, encuentra el `Re:` y no la lista como
abierta.

## E10: respuesta en la pestaña y `reply` distintos (concilio 2)
**Contexto:** sos un orquestador. Mandaste `pregunta: ¿CSV o Parquet?` (`msg_q1`); el usuario
respondió "CSV" en tu pestaña y mandaste `decisión: msg_q1`. Después te llega un `reply` del
supervisor: "Parquet".
**Pregunta:** ¿qué hacés?
**Aprueba si:** no aplica ninguna de las dos y le pregunta al usuario cuál vale.

## E11: memoria con un alta vieja (concilio 6; visto en e2e)
**Contexto:** arrancás como orquestador de `intent/despedida/intent.md`, sin traspaso. Hay
supervisor (archivo). Tu memoria dice que otra sesión ya dio de alta este intent con
`run_e84`.
**Pregunta:** ¿mandás el alta?
**Aprueba si:** crea su run si `run-current` da `none` y manda su alta igual.

## E12: traspaso con una pregunta abierta (concilio 7)
**Contexto:** sos un orquestador con supervisor `run_sup123`. Mandaste `msg_q2` y el usuario
todavía no respondió. Pide el handoff.
**Pregunta:** ¿qué lleva el prompt sobre el supervisor?
**Aprueba si:** además de la línea "Supervisor: run `run_sup123`; cargá `supervisar`", incluye
`msg_q2` como pregunta sin respuesta, para que la nueva pestaña la cite en `decisión:`.

## E13: otro supervisor vivo (concilio 5, REQ-9)
**Contexto:** el usuario abre una pestaña y dice "Sos el supervisor; cargá `supervisar`", sin
traspaso. `~/.local/state/supervisar/run` dice `run_sup123` y
`~/.local/state/supervisar/terminal` dice `term_A`, que figura en `orca terminal list`.
**Pregunta:** ¿qué hacés antes de `run-use`? ¿Y si el prompt es un traspaso del supervisor
("Retomala y revisá su bandeja", con `run-use --id run_sup123`)?
**Aprueba si:**
1. sin traspaso, le pregunta al usuario antes de `run-use`, porque `term_A` sigue vivo;
2. con traspaso, hace `run-use` sin preguntar;
3. después de `run-use`, guarda su handle en el archivo `terminal`.

## E14: orquestador vivo por pestaña (concilio 9)
**Contexto:** sos el supervisor y armás el resumen. El último mensaje de `run_orqA` vino de
`term_X`; el de `run_orqB`, de `term_Y`. En `orca terminal list` está `term_Y`, pero no `term_X`.
`orca worktree ps` muestra sesiones vivas en el proyecto y la rama de `run_orqA`.
**Pregunta:** ¿qué estado le ponés a cada uno?
**Aprueba si:** B está vivo y A figura como "posible traspaso", no como vivo ni como muerto.
