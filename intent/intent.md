# Intención: supervisar
Autor: Denis Hugo Perafan · Estado: aceptado · Fecha: 2026-11-12

## Problema
Por cada intent o idea termino con 4 o 5 chats tipo orquestador, en parte porque cada handoff
abre otra pestaña y la anterior queda viva. Cada uno tiene su contexto, sus tareas, sus preguntas
y sus decisiones pendientes, y "me termino volviendo loco" saltando entre chat y chat.

## Resultado esperado
Hay un observador (supervisor) y los orquestadores le van diciendo todo, para que pueda coordinar.
Tengo una sesión principal desde la cual gestiono el trabajo:
- me da un resumen del estado general;
- centraliza las preguntas que necesitan una decisión mía;
- detecta tareas duplicadas, dependencias, conflictos y bloqueos;
- clasifica una idea nueva: corresponde a un intent existente o necesita uno nuevo;
- propone acciones y pide aprobación, en lugar de intervenir solo.

Puedo responder desde el supervisor o directamente en la pestaña del orquestador; en los dos casos
la decisión le llega al supervisor. El supervisor no es un cuello de botella ni el único canal.
Después de un handoff, la pestaña anterior no queda viva. El supervisor también se queda sin
contexto: cuando hace su propio traspaso, conserva la dirección a la que le escriben los
orquestadores y guarda resúmenes, no el historial completo.

## Usuarios y sistemas afectados
Yo, como único usuario. Los orquestadores (Claude Code, Codex, Antigravity) en Orca. Las skills
`handoff-clear` y `ai-native-sdlc`, que la nueva skill reutiliza.

## Restricciones
- Es una skill, no una CLI ni un bus de eventos propio: usa lo que Orca ya trae (runs, mensajes,
  resumen de worktrees, búsqueda en sesiones).
- No se necesita compartir todo el historial: alcanza con mensajes, checkpoints y resúmenes, y
  recuperar más contexto cuando haga falta.
- El estado durable sigue en `intent/` de cada proyecto.
- Mínima y genérica, sin texto repetido entre skills; una skill por tarea.
- Preferentemente, el supervisor corre en un modelo de alta capacidad.
- Las sesiones usan ventana de 1M. Hago handoff cerca del 30 % en el supervisor y del 50 % en un
  orquestador. El que más rápido se llena es el supervisor; un orquestador
  con varios workers, no tanto.

## Fuera de alcance
- Cambiar la skill `orquestar`.
- Que el supervisor actúe solo (acciones autorizadas, automatización): queda para más adelante.
- Detectar conflictos de archivos entre worktrees distintos.

## Preguntas abiertas
- ¿Un solo supervisor global o uno por proyecto?
- ¿Cómo carga la skill un orquestador Codex o Antigravity?

## Cambios
2026-11-12: se suma la frecuencia de handoff (30–50 % de contexto) y que el supervisor es el que
más rápido se llena, porque define cuánto cuesta cada mensaje que recibe. Aprobado por Denis.
2026-11-12: los umbrales pasan de 30–50 % a ~150k (supervisor) y ~250k (orquestador) porque las
ventanas son de 1M y cada turno del supervisor relee todo su contexto. Aprobado por Denis.
2026-11-12: los umbrales vuelven a mi rango (supervisor ~30 %, orquestador ~50 %); con ~15 % y
~25 % viviría cambiando de sesión. Aprobado por Denis.
