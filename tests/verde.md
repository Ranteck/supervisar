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
