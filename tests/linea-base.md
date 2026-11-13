# Línea de base (RED): sin la skill

2026-11-12 · subagentes Haiku (`orquestar:explorador`) · cada uno recibió solo el escenario, sin
los criterios · `handoff-clear` en su versión actual (1ca4fc7).

| Escenario | Resultado | Falla |
| --- | --- | --- |
| E1 alta | ❌ | no crea su run; en vez de un alta, le manda una consulta al supervisor; planea reportar avances por hito |
| E2 pregunta | ⚠️ | pregunta en la pestaña, usa `send` y cita `msg_q1`, pero agrega `--run run_orqA` a `send --to run:run_sup123`, combinación que falla en Orca |
| E3 resumen | ❌ | ante respuestas contradictorias elige OAuth en vez de mostrar el conflicto; el resto aprueba |
| E4 idea nueva | ✅ | lo cubre la guarda contra el desvío de `ai-native-sdlc` |
| E5 supervisores | ✅* | aprueba las tres variantes, pero la pregunta ya lo empuja a buscar; lo espontáneo se mide en la Tarea 5 |
| E6 traspaso | ❌ | el prompt no menciona al supervisor; la pestaña vieja no se cierra; le avisa del traspaso al supervisor (el `run-use` sí aprueba) |
| E7 concilio y traspaso | ❌ | lanza el concilio en `run_orqA` y la nueva lo retoma, pero la vieja no se cierra |
| E8 umbral de contexto | ❌ | con 31 % no propone nada: considera baja la ocupación y no conoce ningún umbral |

\* La primera versión de E5 filtraba la convención `supervisor:` y qué terminales estaban vivas;
se reescribió y se volvió a correr.

## Qué tiene que corregir la skill
- Alta única con intent, proyecto, rama y run propio, creando el run antes (E1).
- Prohibir el progreso: solo alta, pregunta, decisión y cierre (E1).
- `send` al supervisor sin `--run` (E2).
- Mostrar los conflictos sin elegir (E3).
- Proponer el traspaso al pasar el umbral: 30 % el supervisor, 40 % el orquestador (E8).
- En `handoff-clear`, sumar la línea del supervisor, no avisar del traspaso y cerrar la pestaña
  vieja con `run-current` en null (E6, E7).

## Concilio (E9–E12), con la skill de 06e22f8
2026-11-13 · subagente Haiku (`orquestar:explorador`) · sin los criterios.

| Escenario | Resultado | Nota |
| --- | --- | --- |
| E9 `Re:` del anterior | ❌ | lee solo el inbox del supervisor; no ve el `Re:` en `run_orqA` |
| E10 pestaña vs `reply` | ⚠️ | pregunta cuál vale, pero improvisando: la skill dice "aplicala" |
| E11 memoria | ❌ | no manda el alta si la memoria dice que hubo otra |
| E12 pregunta abierta | ❌ | el prompt no lleva `msg_q2` |
