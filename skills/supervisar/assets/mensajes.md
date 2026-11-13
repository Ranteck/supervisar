# Mensajes al supervisor

Cada uno se manda con `orca orchestration send --to run:<supervisor> --subject … --body …`.

| Tipo | `--subject` | `--body` |
| --- | --- | --- |
| alta | `alta: <ruta del intent>` | `proyecto: <ruta> · rama: <rama> · run: <tu run>` |
| pregunta | `pregunta: <la pregunta, en una línea>` | `opciones: <a, b, …> · contexto: <archivo donde se decide>` |
| decisión | `decisión: <id del mensaje de la pregunta>` | `<respuesta del usuario, textual>` |
| cierre | `cierre: <ruta del intent>` | `resultado: terminado o abandonado · <commit o motivo>` |
