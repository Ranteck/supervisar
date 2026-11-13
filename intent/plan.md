# supervisar: plan de implementación
Estado: aceptado

> **Para agentes:** SUB-SKILL REQUERIDA: superpowers:executing-plans (método Native). Los pasos
> usan checkboxes (`- [ ]`).

**Objetivo:** una skill `supervisar` con dos roles (supervisor y orquestador que le reporta), y
`handoff-clear` ajustada para pasarle la dirección y cerrar la pestaña vieja.

**Arquitectura:** sin código. La skill es Markdown y usa lo que ya trae Orca: runs como
direcciones, `send` y `reply`, `run-use`, `worktree ps`, `search`. El estado durable sigue en
`intent/` de cada proyecto.

**Stack:** Markdown, Orca CLI 1.4.253 y Claude Code; Codex y Antigravity solo como workers o
concilio.

**Spec:** `intent/spec.md` (aceptado 2026-11-12), desde `intent/intent.md`.

## Restricciones globales
- Solo cuatro mensajes: alta, pregunta, decisión y cierre. Sin progreso (REQ-4).
- El supervisor no guarda estado propio y nunca actúa sin aprobación (REQ-6, REQ-7).
- Un solo supervisor por máquina (REQ-9).
- Mínima y genérica, sin repetir texto de `handoff-clear` ni de la guía de Orca; en español.
- Hechos verificados el 2026-11-12, que la skill da por ciertos:
  - `ask` solo funciona en workers;
  - solo un agente con run propio se despierta al recibir un mensaje;
  - `run-use` pasa el run a la pestaña nueva y la vieja queda con `run-current` en null;
  - `send --to run:<x>` falla si se le agrega `--run` de otro run.

## Foco de revisión
1. El archivo de dirección apunta a un run que ya no existe (Orca reseteado): el supervisor crea
   otro y reescribe el archivo; el orquestador que recibe `run_not_found` sigue sin supervisor y
   avisa.
2. Un orquestador sin run propio queda sordo: el alta crea el run antes de mandarse.
3. Si el usuario responde en las dos pestañas, el supervisor muestra el conflicto; no elige.
4. Si el supervisor se queda sin contexto en medio de un mensaje, no tiene que perderse nada:
   la vieja cierra recién con `run-current` en null.
5. Un orquestador que arrancó sin traspaso y sin que le hablen del supervisor no lo busca: la
   descripción de la skill tiene que dispararse igual.

---

### Tarea 1: Escenarios y línea de base (RED)

**Archivos:** crear `tests/escenarios.md`.

- [ ] Escribir seis escenarios, cada uno con prompt, contexto simulado y criterio de aprobado:
  - **E1, alta:** "hay un supervisor en run X; arrancá el intent Y". Aprueba si crea su run y
    manda un solo alta con intent, proyecto, rama y dirección.
  - **E2, pregunta:** necesita una decisión. Aprueba si pregunta en su pestaña, manda
    `pregunta:` por `send` (no `ask`) y, cuando el usuario responde ahí, manda `decisión:`
    citando el id.
  - **E3, resumen:** inbox con una pregunta ya respondida en la pestaña (`search` lo muestra) y
    respuestas contradictorias en otra. Aprueba si no la da por abierta y muestra el conflicto.
  - **E4, idea nueva:** la idea se solapa con un intent abierto. Aprueba si propone sumarla y no
    lanza nada.
  - **E5, supervisores:** ninguno, uno vivo más uno viejo, y dos vivos. Aprueba si con ninguno
    trabaja como hoy, con el viejo lo descarta y con dos vivos pregunta.
  - **E6, traspaso:** aprueba si el prompt lleva la línea del supervisor, la nueva hace
    `run-use`, la vieja cierra solo con `run-current` en null y el supervisor no recibe nada.
- [ ] Correr cada escenario con un subagente Haiku **sin** la skill y anotar cómo falla.
- [ ] Commit: `test: add supervisar pressure scenarios and baseline`.

### Tarea 2: La skill (GREEN)

**Archivos:** crear `skills/supervisar/SKILL.md`, `skills/supervisar/assets/mensajes.md` y
`skills/supervisar/assets/resumen.md`.

**Interfaces que producen:**
- Asunto de cada mensaje: `alta:`, `pregunta:`, `decisión:` o `cierre:`, más un texto corto.
  `decisión:` cita el id de la pregunta en `--thread-id`.
- El supervisor tiene un solo run permanente, creado con `run-create --objective "supervisor"`.
  Su id vive en `~/.local/state/supervisar/run`, y cada supervisor nuevo lo retoma con `run-use`.
  Orca no expone qué terminal tiene un run y `run-list` está paginado; verificado el 2026-11-12.
- La dirección del supervisor se pasa como `run:<id>`.

- [ ] Frontmatter: `name: supervisar`. La descripción dispara con "supervisá", "sos el
  supervisor" o un prompt que traiga "Supervisor: run".
- [ ] Sección **Si sos el supervisor**:
  - crear o retomar el run;
  - resumir a pedido desde `inbox`, `worktree ps`, `intent/` y `search`;
  - clasificar las ideas nuevas;
  - proponer y esperar aprobación; para lanzar un orquestador aprobado, usar `handoff-clear`
    con la línea del supervisor;
  - no contestar altas ni cierres.
- [ ] Sección **Si reportás a uno**:
  - ubicar al supervisor: primero el prompt o el usuario; si no, el archivo de dirección;
  - crear tu run si no tenés, y que sea el único: los workers y concilios van en ese run, sin
    `run-create` nuevos, porque `run-use` se queda solo con el run que nombra;
  - los cuatro mensajes;
  - preguntar en tu pestaña y también por `send`.
- [ ] `assets/`: plantilla de cada mensaje y del resumen. El resumen lista orquestadores, su
  intent, preguntas abiertas, conflictos, bloqueos y duplicados.
- [ ] En el resumen, un orquestador está vivo si su proyecto y rama tienen sesiones vivas en
  `worktree ps`.
- [ ] Repetir E1–E5, E7 y E8 **con** la skill: todos tienen que aprobar. Si alguno falla, ajustar la
  skill y no el escenario.
- [ ] Commit: `feat: add supervisar skill`.

### Tarea 3: handoff-clear

**Archivos:** modificar `~/Proyectos/handoff-clear/skills/handoff-clear/SKILL.md`, en los
pasos 2 y 3. Es otro repo.

- [ ] Paso 2, **Estado**: si hay supervisor, sumar "Supervisor: run `<id>`; cargá
  `supervisar`". Si no hay, nada.
- [ ] Paso 3, cierre:
  - con orquestación en curso: esperar a que la pestaña nueva esté `tui-idle` y cerrar la
    propia recién cuando `run-current` dé null;
  - sin orquestación: cerrar cuando la nueva muestre sus tres líneas;
  - en las dos variantes, el cierre es la última acción.
- [ ] Script `skills/handoff-clear/scripts/contexto`: imprime el porcentaje de la sesión
  actual, sumando input y caché del último mensaje del modelo que no sea de un subagente, en
  `$CLAUDE_CONFIG_DIR/projects/*/$CLAUDE_CODE_SESSION_ID.jsonl`, sobre 1M. Lleva un comentario
  `shortcut:` porque el formato del JSONL no está documentado. Test con un JSONL de ejemplo que
  incluye un mensaje de subagente que hay que ignorar.
- [ ] En `supervisar`: el supervisor corre `contexto` al terminar de atender cada mensaje, y el
  orquestador al cerrar cada tarea; si pasan el 30 % o el 40 %, proponen el traspaso (REQ-11).
- [ ] Correr E6 sin el cambio (RED) y con el cambio (GREEN).
- [ ] Commit en ese repo: `feat: report to the supervisor and close the old tab on handoff`.

### Tarea 4: Instalación

- [ ] Crear `~/.claude/skills/supervisar` → `../../Proyectos/skills/supervisar/skills/supervisar`.
  Ese directorio lo comparten los dos perfiles.
- [ ] Verificar: `test -f ~/.claude/skills/supervisar/SKILL.md`. Codex y Antigravity no la
  necesitan, porque le reportan al orquestador.

### Tarea 5: Prueba de punta a punta en Orca

**Archivos:** crear `tests/e2e-orca.md` con los comandos y lo esperado.

- [ ] Esta sesión hace de supervisor, con un run `supervisor:` nuevo.
- [ ] Orquestador Haiku con la skill:
  - el alta llega una sola vez;
  - la `pregunta:` llega y mi `reply` lo despierta;
  - una respuesta dada en su pestaña llega como `decisión:`.
- [ ] Traspaso con `handoff-clear`, con un worker del mismo run todavía corriendo: la nueva toma
  el run y recibe su `worker_done`, la vieja se cierra y al supervisor no le llega nada.
- [ ] Traspaso del supervisor: un Haiku hace `run-use` del run supervisor y recibe el alta
  siguiente; después lo retomo yo con `run-use` (REQ-7).
- [ ] Un orquestador nuevo, abierto sin traspaso y sin mencionar al supervisor, lo encuentra y se
  da de alta (foco 5).
- [ ] Cerrar todas las pestañas de prueba y verificar con `terminal list`.
- [ ] Commit: `test: add Orca end-to-end runbook`.

### Tarea 6: Concilio (6.4)

- [ ] Codex y Antigravity en nivel medio y solo lectura, con spec, skill, diff de
  `handoff-clear` y escenarios. Que intenten romperlo: wake storms, supervisores viejos,
  decisiones perdidas, traspasos a mitad de un mensaje. Que revisen contra los criterios
  mínima/genérica y sin redundancia.
- [ ] Cada falla confirmada se convierte en un escenario nuevo antes de corregirla.
- [ ] Commit con las correcciones.

### Tarea 7: Cierre
- [ ] Pedir OK para crear el remoto en GitHub y pushear los dos repos.

## Tests a escribir
| Test | REQ |
| --- | --- |
| E1 alta | REQ-1, REQ-2, REQ-4 |
| E2 pregunta y decisión | REQ-3, REQ-4 |
| E3 resumen | REQ-5, REQ-7 |
| E4 idea nueva | REQ-6 |
| E5 supervisores | REQ-9 |
| E6 traspaso | REQ-8, REQ-10 |
| E7 concilio durante el traspaso | REQ-2, REQ-8 |
| E8 umbral de contexto | REQ-11 |
| test de `contexto` | REQ-11 |
| E2E Orca | REQ-1, REQ-2, REQ-3, REQ-7, REQ-8 |

## Riesgos
- Cerrar la propia pestaña corta la sesión: tiene que ser la última acción.
- Los subagentes de los escenarios no corren Orca de verdad; la Tarea 5 es la prueba real.
- Interpretación mía: REQ-8 cierra la pestaña vieja también sin supervisor, porque el intent lo
  pide sin condición. Con eso, REQ-10 queda para lo demás.
