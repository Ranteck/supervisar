# Con la skill (GREEN)

2026-11-12 · subagentes Haiku (`orquestar:explorador`) que leyeron `skills/supervisar/` · sin
los criterios.

| Escenario | Resultado | Nota |
| --- | --- | --- |
| E1 alta | ✅ | crea su run; un alta con los cuatro datos y sin `--run`; sin avances |
| E2 pregunta | ✅ | `send` sin `--run`; pregunta también en la pestaña; `decisión: msg_q1` textual |
| E3 resumen | ✅ | `msg_q1` cerrada; conflicto de `msg_q2` sin elegir; duplicado A/C; C sin sesiones; sin notas |
| E4 idea nueva | ✅ | clasifica, propone y espera |
| E5 supervisores | ✅ | (a) como siempre; (b) alta a `run_s2`; (c) sigue sin supervisor y avisa |
| E7 concilio | ✅ parcial | sin `run-create` nuevos; la nueva retoma `run_orqA`; el cierre de la vieja es de la Tarea 3 |
| E8 umbral | ✅ | 31 % propone y espera; 25 % no dice nada |

## REFACTOR
- E4 entendió que el supervisor nunca le escribe a un orquestador: se sumó que, con OK, la
  instrucción va con `send --to run:<su run>`. Verificado.
- E8 usó para su propio traspaso la línea de los orquestadores: se sumó que su run es la
  orquestación en curso (verificado) y que el prompt dice "Sos el supervisor" (se verifica en la
  Tarea 5).

## Tarea 3
| Escenario | Resultado | Nota |
| --- | --- | --- |
| E6 traspaso | ✅ | línea del supervisor; `run-use`; cierre como última acción tras `run-current` null; nada al supervisor |
| E7 concilio | ✅ | el concilio sigue en `run_orqA` y lo retoma la nueva; la vieja cierra con la prueba |
| test de `contexto` | ✅ | RED sin script; GREEN con fixture que incluye subagente y línea rota |

## Concilio (Tarea 6)
2026-11-13 · revisor único Opus vía Antigravity (Codex, Flash y Argon sin cuota) · GREEN con
subagente Haiku sin los criterios.

| Escenario | Resultado | Nota |
| --- | --- | --- |
| E9 `Re:` del anterior | ✅ | lee el inbox del run del alta |
| E10 pestaña vs `reply` | ✅ | no aplica ninguna y pregunta cuál vale |
| E11 memoria | ✅ | crea su run y manda el alta igual |
| E12 pregunta abierta | ✅ | el prompt lleva `msg_q2`; nada al supervisor; cierra al final con prueba |
| E7, E8 (regresión) | ✅ | tras los recortes |

Sin aplicar, a decisión de Denis: dos supervisores vivos (5), vivo por pestaña y no por rama
(9), mover el 40 % a handoff-clear.

## E13–E14
2026-11-13 · subagente Haiku sin los criterios.

| Escenario | Resultado | Nota |
| --- | --- | --- |
| E13 otro supervisor vivo | ✅ | sin traspaso pregunta; con traspaso `run-use` directo; guarda su handle |
| E14 vivo por pestaña | ✅ | A "posible traspaso", B vivo |
| E8 (regresión) | ✅ | |
Tras el concilio bajo (Opus vía Antigravity: traspaso sin definir, orden del arranque, precedencia
de `cierre:`): E13a/b/c y E14 con `cierre:` ✅.

## Supervisor por proyecto (E5, E13, E15)
2026-11-13 · subagente Haiku sin los criterios.

| Escenario | Resultado | Nota |
| --- | --- | --- |
| E5 | ✅ | busca en `$(git rev-parse --git-common-dir)/supervisar/run` |
| E13 | ✅ | archivos del `.git` de su repo; pregunta si `term_A` vive |
| E15 | ✅ | A-feat reporta a `run_supA`; B trabaja sin supervisor; la idea de B se propone llevar allá |
| E8 (regresión) | ✅ | se hizo explícito que el prompt del traspaso lleva `run-use --id` |
