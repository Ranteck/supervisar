# supervisar

Skill de Claude Code para coordinar desde una sola sesión, el supervisor, los orquestadores que
trabajan en paralelo en un mismo proyecto. Funciona sobre [Orca](https://onorca.dev).

## Para qué sirve

Por cada intent o idea es fácil terminar con varios chats de orquestador abiertos, cada uno con
sus tareas, sus preguntas y sus decisiones pendientes. El supervisor:

- da un resumen del estado: qué orquestador está vivo, qué preguntas siguen abiertas, qué está
  duplicado o bloqueado;
- junta las preguntas que necesitan una decisión tuya, y le devuelve tu respuesta al orquestador;
- dice si una idea nueva corresponde a un intent existente o pide uno nuevo;
- propone acciones y pide tu OK; nunca lanza, cierra ni redirige nada por su cuenta.

Podés responder desde el supervisor o desde la pestaña del orquestador; en los dos casos la
decisión le llega al otro.

## Cómo funciona

Hay un supervisor por proyecto (un repo con todos sus worktrees). Su dirección es un run de Orca
cuyo id vive en `.git/supervisar/run`, que todos los worktrees comparten y que git no versiona.

Un orquestador que arranca en el repo lo encuentra solo y le manda nada más que cuatro mensajes:
`alta` al empezar, `pregunta` cuando necesita tu decisión, `decisión` cuando respondiste en su
pestaña y `cierre` al terminar. El avance no se reporta: el supervisor lo lee de Orca y de
`intent/` cuando lo necesita, así su contexto se llena más despacio.

## Por qué encaja con Orca

La orquestación de Orca ya trae lo que un supervisor necesita, sin bus de eventos propio:

- **Un run es una dirección estable, no una pestaña.** Cualquier sesión puede tomarlo con
  `run-use`, y los mensajes a `run:<id>` le llegan a quien lo coordine en ese momento. Por eso un
  traspaso no pierde nada: la pestaña nueva toma el run y recibe lo pendiente, incluidos los
  `worker_done` de los workers que siguen corriendo.
- **Los mensajes despiertan a quien los recibe.** Orca avisa en la sesión de Claude Code, así que
  nadie hace polling: tu respuesta despierta al orquestador, y una pregunta despierta al
  supervisor.
- **El inbox persiste.** Si el supervisor está entre sesiones, los mensajes esperan; las
  respuestas (`reply`) quedan en el run del orquestador y se pueden releer.
- **Cada mensaje trae la pestaña que lo mandó.** Con `orca terminal list` el supervisor sabe si un
  orquestador sigue vivo, y `orca worktree ps` y `orca search` completan el estado.

## Con handoff-clear y ai-native-sdlc

- [handoff-clear](https://github.com/Ranteck/handoff-clear) hace los traspasos: abre la pestaña
  nueva con la línea "Supervisor: run `<id>`", cierra la vieja cuando la nueva tomó el run y
  mide el contexto. Supervisar propone el traspaso al 30 % (supervisor) o al 40 % (orquestador).
- [ai-native-sdlc](https://github.com/Ranteck/ai-native-sdlc) define `intent/`, el estado
  durable de cada proyecto: el supervisor lo lee para el resumen y para clasificar ideas nuevas.

## Instalación

Requiere Claude Code, Orca, handoff-clear instalado y proyectos en repos git.

```sh
git clone https://github.com/Ranteck/supervisar.git ~/Proyectos/skills/supervisar
ln -s ~/Proyectos/skills/supervisar/skills/supervisar ~/.claude/skills/supervisar
```

Para abrir un supervisor, en una pestaña de Orca dentro del proyecto: "Sos el supervisor; cargá
`supervisar`". Los orquestadores la cargan solos al arrancar un intent.

## Licencia

MIT
