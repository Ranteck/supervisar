# Spec: supervisar (desde intent.md 2026-11-12)
Estado: aceptado

## Requisitos
- REQ-1 MUST: Un orquestador con supervisor se da de alta con un solo mensaje: intent, proyecto,
  rama y su dirección. Origen: Resultado esperado, "le van diciendo todo".
- REQ-2 MUST: Cada orquestador tiene una dirección propia que lo despierta al recibir un mensaje
  y que no cambia con sus traspasos. Origen: Resultado esperado, "Puedo responder desde el
  supervisor o directamente en la pestaña".
- REQ-3 MUST: Una pregunta que necesita decisión del usuario se hace en la pestaña y se manda al
  supervisor. Se responde en cualquiera de los dos: la respuesta llega al orquestador, y la dada
  en la pestaña llega al supervisor citando la pregunta. Origen: Resultado esperado,
  "en los dos casos la decisión le llega al supervisor".
- REQ-4 MUST: Los orquestadores mandan solo alta, pregunta, decisión y cierre (terminó o
  abandonó); no reportan progreso. Origen: Restricciones, "alcanza con mensajes, checkpoints y
  resúmenes".
- REQ-5 MUST: A pedido, el supervisor resume: orquestadores vivos y su intent, preguntas
  abiertas, bloqueos y duplicados (mismo intent, o mismo proyecto y rama). Antes de dar una
  pregunta por abierta, verifica que no se haya respondido en la pestaña. Origen: Resultado
  esperado, "me da un resumen".
- REQ-6 MUST: Ante una idea nueva, el supervisor dice si va a un intent existente o a uno nuevo y
  propone el paso siguiente; no lanza, cierra ni redirige a nadie sin aprobación. Origen:
  Resultado esperado, "propone acciones y pide aprobación".
- REQ-7 MUST: El supervisor no lleva estado propio: lo reconstruye de los mensajes, `intent/` y
  las sesiones vivas, y al hacer traspaso conserva su dirección. Origen: Resultado esperado,
  "conserva la dirección".
- REQ-8 MUST: Tras el traspaso de un orquestador, la pestaña vieja se cierra cuando la nueva
  confirma que tomó el trabajo, y el supervisor no recibe mensajes por eso. Origen: Resultado
  esperado, "la pestaña anterior no queda viva"; Restricciones, "Hago handoff cerca del 30 % en el supervisor y del 40 % en un orquestador".
- REQ-9 MUST: Sin supervisor activo, el orquestador trabaja como hoy y no crea uno. Hay uno solo
  por máquina; si encuentra más de uno, pregunta al usuario. Origen: Resultado esperado, "no es
  el único canal"; Preguntas abiertas.
- REQ-10 MUST: El traspaso actual sigue igual sin supervisor y solo suma su dirección cuando hay
  uno. Origen: Restricciones, "sin texto repetido entre skills".
- REQ-11 SHOULD: Al pasar su umbral de contexto (supervisor 30 %, orquestador 40 %), el agente
  propone el traspaso y espera aprobación. Origen: Restricciones, "Hago handoff cerca del 30 %
  en el supervisor y del 40 % en un orquestador".

## Capacidades y escenarios
### Alta y traspaso (REQ-1, REQ-2, REQ-8)
- GIVEN un supervisor activo
- WHEN arranca un orquestador y, más tarde, cerca del 40 % de contexto, hace traspaso
- THEN el supervisor recibe un único alta; la pestaña nueva toma la misma dirección y la vieja se
  cierra

### Pregunta respondida en la pestaña (REQ-3, REQ-5)
- GIVEN una pregunta mandada al supervisor
- WHEN el usuario la responde en la pestaña del orquestador
- THEN el supervisor recibe la decisión y no la muestra como abierta

### Idea nueva (REQ-6)
- GIVEN un intent abierto sobre el mismo tema
- WHEN el usuario le cuenta la idea al supervisor
- THEN el supervisor propone sumarla a ese intent y espera aprobación

### Supervisor sin contexto (REQ-7)
- GIVEN un supervisor haciendo traspaso
- WHEN un orquestador le escribe durante el cambio
- THEN el supervisor nuevo recibe el mensaje

### Umbral de contexto (REQ-11)
- GIVEN un supervisor al 31 % de contexto
- WHEN termina de atender un mensaje
- THEN propone el traspaso y no lo hace hasta que el usuario acepte

### Sin supervisor (REQ-9, REQ-10)
- GIVEN ningún supervisor activo
- WHEN un orquestador hace traspaso
- THEN todo es igual que hoy

## Concerns
- Sin probar: que la pestaña nueva tome la dirección de un orquestador vivo sin forzar; se
  verifica en el plan.
- Probado el 2026-11-12: `ask` solo funciona en workers; las preguntas van como mensaje con
  respuesta.
- Cada mensaje cuesta un turno del supervisor, que es el que más rápido se llena: de ahí REQ-4,
  REQ-7 y REQ-8.
- Una decisión dada en la pestaña y no avisada: REQ-5 lo mitiga, no lo garantiza.
- Un orquestador que muere sin cierre se detecta cruzando altas con sesiones vivas.
- Abierto: cómo cargan la skill Codex y Antigravity.
- Sin verificar: cuánto guarda Orca los mensajes.
