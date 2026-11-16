# SKILLS_DESIGN.md — Diseño de skills de Sebastian

**Estado:** diseño previo a implementación  
**Fecha:** 2026-11-16  
**Objetivo:** gestionar por Telegram (texto o voz) la planificación mensual/semanal, las tareas, el calendario, el seguimiento diario, las reuniones y los borradores de correo, dejando un historial consultable de lo realizado y sus motivos.

Este documento responde, para cada skill, a las **tres preguntas exigidas por 4Geeks**: qué hace, qué input necesita y cómo se verifica un buen output. No implementa ninguna skill ni automatización.

## Arquitectura compartida

Las skills utilizan el contexto existente en `AGENTS.md`, `SOUL.md`, `IDENTITY.md`, `USER.md` y, cuando corresponda, `MEMORY.md`. **No duplican** sus normas generales de seguridad, personalidad, memoria ni autorización.

- **Entrada e interacción:** Telegram privado; texto o voz transcrita (transcripción pendiente de comprobar).
- **Integraciones:** Zapier MCP con Google Docs, Drive, Tasks, Calendar y Gmail. Notion está conectado, pero queda fuera del alcance inicial.
- **Fuentes de referencia:** **Docs** = objetivos, planes, motivos, actas y diario; **Tasks** = tareas y estados; **Calendar** = fechas y bloques de tiempo; **Drive** = organización y enlaces; **Telegram** = solicitudes, preguntas y confirmaciones.
- **Consistencia:** reutilizar identificadores/enlaces de objetivos, tareas, documentos y eventos; localizar antes de crear; registrar resultados parciales para poder reintentar sin duplicar.
- **Seguridad:** todas las skills se someten a `AGENTS.md`, incluidos los límites ante contenido no confiable y los requisitos de aprobación. Una skill no concede permisos nuevos.

## Confirmación obligatoria de resultados por Telegram

**Criterio transversal de aceptación:** ninguna ejecución se da por terminada sin comprobar sus resultados y emitir, al chat privado autorizado, **un mensaje final breve y verificable**.

1. **Verificar:** tras cada escritura, comprobar el resultado mediante el recurso recuperado (ID, estado, ubicación y/o lectura posterior, según permita la integración). Un resultado devuelto sin comprobación suficiente se marca *pendiente de verificar*, nunca *completado*.
2. **Informar:** al finalizar, enviar por Telegram qué se creó, actualizó o descartó, con cifras y **enlaces relevantes** (Doc, tarea, evento o borrador) cuando estén disponibles.
3. **Distinguir estados:** *Completado y verificado* / *Parcial* / *No completado*. Si algo falla, indicar qué sí se hizo y qué queda pendiente, sin afirmar una sincronización inexistente.
4. **Evitar ruido:** cuando varias skills colaboren en un mismo flujo, enviar **una única confirmación consolidada** desde el flujo coordinador; las skills auxiliares devuelven resultados estructurados al coordinador, no mensajes independientes. Si se invoca una skill por separado, ella confirma su propio resultado.
5. **No confundir preguntas con cierres:** el aviso de las 17:30 y la pregunta posterior a una reunión inician un flujo; **no sustituyen** la confirmación de cierre tras procesar la respuesta. Si no hay respuesta, no inventar avances ni enviar un falso mensaje de finalización.
6. **Telegram no disponible:** registrar que la notificación no se pudo entregar y permitir reintento controlado; no afirmar que el usuario recibió el resultado.

**Ejemplos de cierre:**
- «Plan semanal creado y verificado: 6 tareas en Tasks. [Ver plan]. Pendiente: aprobar 3 bloques de calendario».
- «Revisión registrada: 2 tareas completadas, 1 aplazada y diario actualizado. [Ver diario]».
- «Resultado parcial: borrador guardado en Gmail, pero no he podido verificar el enlace. No se ha enviado».

## Skill 01 — `planificacion-periodica`

**1. ¿Qué hace esta skill?**  
Convierte objetivos y compromisos enviados por Telegram en un plan mensual o semanal priorizado con resultados esperados y acciones vinculables.

**2. ¿Qué input necesita el agente?**  
Texto o transcripción con periodo, objetivos, prioridades, plazos y restricciones; pueden estar incompletos. Consulta planes existentes, contexto del usuario y, si procede, disponibilidad y tareas. Distingue ideas exploratorias de compromisos; pregunta solo por datos decisivos.

**3. ¿Cómo es un buen output y cómo sabrás que funcionó?**  
Google Doc `AAAA-MM - Plan mensual` o `AAAA-SNN - Plan semanal`, en `01_Planificación`, con prioridades, objetivos, criterios de éxito, fechas y enlaces a acciones. **Verificar** que el documento existe, está en la carpeta correcta y refleja fielmente el periodo y los objetivos sin duplicarlos. **Telegram:** confirmar creación/actualización, número de objetivos, enlace y próximos pasos o pendientes.

**Relaciones:** 09 localiza/crea el Doc; 02 gestiona las acciones; 03 propone bloques; 04 y 06 pueden solicitar revisiones.

## Skill 02 — `gestion-tareas`

**1. ¿Qué hace esta skill?**  
Crea y actualiza tareas de Google Tasks a partir de planes, instrucciones y revisiones, manteniendo estados y referencias coherentes.

**2. ¿Qué input necesita el agente?**  
Acciones con descripción, objetivo de origen, plazo, prioridad y estado cuando existan; IDs de tareas ya registradas y cambios confirmados por el usuario. Consulta la lista de Tasks acordada y tareas existentes antes de escribir.

**3. ¿Cómo es un buen output y cómo sabrás que funcionó?**  
Tareas creadas, modificadas o completadas en la lista correcta, sin duplicados y con plazos/estados fieles al input. **Verificar** por ID o consulta posterior cada cambio y conservar la referencia al objetivo. **Telegram:** «3 tareas creadas, 2 actualizadas; 1 cambio pendiente», con enlaces si la integración los facilita. No dar por completado lo que el usuario no confirmó.

**Relaciones:** recibe 01, 04 y 06; aporta tareas a 03; comparte referencias con 09.

## Skill 03 — `bloques-de-trabajo`

**1. ¿Qué hace esta skill?**  
Propone una agenda semanal para trabajar en las tareas prioritarias y, tras aprobación, reserva bloques en Google Calendar.

**2. ¿Qué input necesita el agente?**  
Tareas pendientes, prioridades, fechas, duraciones estimadas, preferencias horarias y disponibilidad real de Calendar; zona `Europe/Madrid`. Si faltan duraciones, propone estimaciones identificadas como tales.

**3. ¿Cómo es un buen output y cómo sabrás que funcionó?**  
Propuesta de bloques por Telegram con tarea, día, hora y duración; **solo tras aprobarla**, eventos enlazados a sus tareas. **Verificar** IDs, horarios, calendario correcto y ausencia de solapamientos no deseados. **Telegram:** confirmar bloques efectivamente creados y enlazarlos; señalar conflictos o bloques sin crear.

**Relaciones:** 01/02 aportan prioridades y tareas; 07 comparte lógica de Calendar; 04/06 pueden desencadenar replanificación.

## Skill 04 — `revision-diaria`

**1. ¿Qué hace esta skill?**  
A las **17:30, Europe/Madrid**, pregunta por Telegram qué se hizo, qué avanzó y qué debe añadirse, modificarse o retirarse de la planificación.

**2. ¿Qué input necesita el agente?**  
Disparador programado de OpenClaw; planes y tareas vigentes; respuesta del usuario por texto o voz transcrita. Distingue actividades realizadas, intenciones y peticiones de cambio. Si no hay respuesta, deja la revisión pendiente sin insistir automáticamente.

**3. ¿Cómo es un buen output y cómo sabrás que funcionó?**  
Una pregunta diaria entregada una sola vez; tras la respuesta, registro en 05 y actualizaciones autorizadas mediante 02/01, con posible propuesta de 03. **Verificar** diario y estados de tareas antes de cerrar. **Telegram:** una confirmación final breve con avances registrados, cambios realizados y pendientes. El envío a las 17:30 requiere una automatización aparte, no solo esta skill.

**Relaciones:** coordina 05 y, según respuestas, 02, 01 y 03.

## Skill 05 — `diario-actividad`

**1. ¿Qué hace esta skill?**  
Mantiene un historial diario consultable de lo realizado, los avances, las decisiones y sus motivos en Google Docs.

**2. ¿Qué input necesita el agente?**  
Resumen de revisión diaria, información confirmada de reuniones y notas del usuario: fecha, acciones, motivos, dificultades y próximos pasos. Consulta la entrada del día antes de ampliarla; distingue hechos declarados de inferencias.

**3. ¿Cómo es un buen output y cómo sabrás que funcionó?**  
Una entrada por fecha en `AAAA-MM - Diario de actividad`, dentro de `02_Diario/AAAA`, con **Hecho / Avances / Decisiones y motivos / Pendientes / Referencias**. **Verificar** la persistencia, fecha, ausencia de duplicados y enlaces a planes/reuniones; probar recuperación por fecha o decisión. **Telegram:** confirmar entrada actualizada y facilitar enlace; si actúa como parte de 04, la confirmación se integra en el cierre de 04.

**Relaciones:** recibe 04 y 06; utiliza 09 para localizar el Doc; 01 consulta antecedentes cuando corresponda.

## Skill 06 — `seguimiento-reuniones`

**1. ¿Qué hace esta skill?**  
Al finalizar una reunión relevante, pregunta si surgieron decisiones o cambios y registra los confirmados en documentos y tareas.

**2. ¿Qué input necesita el agente?**  
Evento identificado como reunión, disparador de finalización y respuesta posterior por texto o voz con decisiones, motivos, responsables, fechas y dudas. Excluir bloques personales y de concentración; no deducir acuerdos del mero título del evento.

**3. ¿Cómo es un buen output y cómo sabrás que funcionó?**  
Una pregunta breve por Telegram; si no hubo cambios, cierre sin crear acta. Si los hubo, acta en `03_Reuniones/AAAA` con decisiones, razones, tareas y responsables, más ajustes autorizados en 01/02/03 y contexto para 05. **Verificar** acta y cada actualización. **Telegram:** un cierre consolidado con cambios reales, enlaces y pendientes; el disparador puede tener retraso.

**Relaciones:** 09 archiva; 02/01/03 actualizan; 05 conserva el porqué.

## Skill 07 — `calendario-inteligente`

**1. ¿Qué hace esta skill?**  
Interpreta solicitudes de agenda en lenguaje natural para crear o modificar eventos de Google Calendar.

**2. ¿Qué input necesita el agente?**  
Texto o voz con asunto, fecha/franja, duración, lugar, participantes y recordatorios cuando se indiquen; consulta calendario y disponibilidad, usando `Europe/Madrid`. Pregunta ante ambigüedad relevante.

**3. ¿Cómo es un buen output y cómo sabrás que funcionó?**  
Evento en el calendario correcto con horario, duración, recordatorio y asistentes autorizados. **Verificar** ID, datos y ausencia de duplicados/conflictos involuntarios. **Telegram:** confirmar fecha, hora y enlace del evento, o explicar qué dato impidió crearlo.

**Relaciones:** comparte Calendar con 03 y 06; puede consultar 02 como contexto.

## Skill 08 — `borradores-correo`

**1. ¿Qué hace esta skill?**  
Transforma instrucciones escritas o dictadas en borradores de Gmail ajustados a la intención y al estilo del usuario, sin enviarlos.

**2. ¿Qué input necesita el agente?**  
Texto o transcripción con propósito, destinatario si existe, datos y tono; opcionalmente conversaciones previas y documentos autorizados. No inventa direcciones, hechos, adjuntos ni firma.

**3. ¿Cómo es un buen output y cómo sabrás que funcionó?**  
Borrador en Gmail con asunto y cuerpo revisados, destinatario conocido y firma sin duplicados. **Verificar** que existe y su estado es **borrador, no enviado**. **Telegram:** confirmar borrador creado, asunto y enlace o identificador disponible; indicar datos pendientes si los hay.

**Relaciones:** consulta 05/06 y documentos mediante 09 cuando aporten contexto; no envía ni modifica otros servicios por defecto.

## Skill 09 — `organizacion-documental`

**1. ¿Qué hace esta skill?**  
Organiza en Google Drive carpetas, Google Docs y referencias compartidas para que todo el sistema sea localizable y no genere duplicados.

**2. ¿Qué input necesita el agente?**  
Tipo de documento, periodo, proyecto si procede, título, contenido y enlaces/IDs recibidos de otras skills; consulta previamente la estructura existente y la ubicación solicitada.

**3. ¿Cómo es un buen output y cómo sabrás que funcionó?**  
Carpeta/documento localizado o creado, con nombre consistente, ID y enlace registrado en el índice. Estructura inicial propuesta:

```text
Sebastian/
├── 00_Sistema/         (índice de objetivos y referencias)
├── 01_Planificación/   (planes por año/mes/semana)
├── 02_Diario/          (un Doc por mes, por año)
├── 03_Reuniones/       (actas por año)
├── 04_Proyectos/       (documentación por proyecto)
└── 99_Archivo/         (histórico cerrado)
```

**Verificar** ubicación, acceso, enlaces y capacidad de reutilizar el documento en ejecuciones sucesivas. **Telegram:** si se invoca sola, confirmar estructura o documento creado; si es auxiliar, devolver resultado a la skill coordinadora para incluirlo en su único aviso. No reorganizar documentos existentes importantes sin autorización.

**Relaciones:** 01, 05 y 06 la utilizan para documentos; 02, 03 y 07 para IDs y enlaces.

## Flujos conjuntos (no son skills adicionales)

1. **Planificar:** Telegram → 01 → 09 (Doc) → 02 (Tasks) → 03 (propuesta de agenda) → Calendar tras aprobación → **Telegram: un cierre verificado**.
2. **Revisión diaria:** automatización 17:30 → Telegram (pregunta) → respuesta → 04 → 05 + 02/01 y, si procede, 03 → **Telegram: un cierre verificado**.
3. **Fin de reunión:** disparador Calendar → 06 (pregunta Telegram) → respuesta → 09 (acta) + 05 (motivos) + 02/01/03 (ajustes) → **Telegram: un cierre verificado**.
4. **Evento puntual:** Telegram → 07 → Calendar → comprobación → **Telegram: confirmación y enlace**.
5. **Correo dictado:** Telegram voz/texto → 08 → borrador Gmail → comprobación → **Telegram: confirmación; no se envía**.

## Dependencias y decisiones de implementación

- **Conectado:** Zapier MCP con Gmail, Google Docs, Google Drive, Google Calendar y **Google Tasks**; comprobar acciones de lectura/escritura y su respuesta de verificación. Notion no es necesario en esta primera versión.
- **Por comprobar:** notas de voz y transcripción en Telegram; envío al chat privado desde flujos programados; selección inequívoca de lista Tasks y calendario; actualización y lectura posterior de Docs/Tasks/Calendar/Gmail; detección de reuniones terminadas.
- **Por configurar:** automatización diaria a las **17:30 Europe/Madrid** y disparador posterior a reuniones. Una skill no crea por sí sola un programador.
- **Por concretar:** carpeta raíz definitiva, lista de Tasks y criterios para aprobar replanificaciones. La creación o cambio significativo de bloques de agenda requiere aprobación.
- **Verificación y fallos:** cada skill debe especificar las operaciones de comprobación y los datos mínimos que devuelve al coordinador (estado, acciones, IDs, enlaces, errores, pendientes). Diseñar reintentos idempotentes y un único aviso de Telegram por flujo.
- **Seguridad:** aplicar `AGENTS.md` y configurar permisos reales de las integraciones. Probar intentos de prompt injection y fallos parciales antes de utilizar datos importantes.

## Orden de construcción y pruebas

**Fase 1 — Planificación:** 09 → 01 → 02 → 03. Prueba: un mensaje de objetivos produce Doc y tareas comprobados, propuesta horaria, bloques tras aprobación y **un resumen de cierre en Telegram**.

**Fase 2 — Seguimiento:** 05 → 04 → 06. Prueba: revisión por voz a las 17:30, diario y Tasks actualizados y verificables, consulta histórica y aviso de cierre; tras una reunión, pregunta pertinente y acta si hubo cambios.

**Fase 3 — Productividad:** 07 → 08. Prueba: evento natural y borrador dictado creados, verificados y confirmados por Telegram, sin envíos ni invitados no autorizados.

**Criterio global de aceptación:** las nueve skills coordinan texto/voz, registran **qué, cuándo y por qué**, enlazan datos sin duplicados y comunican **solo resultados efectivamente verificados** por Telegram mediante cierres breves, indicando fallos y pendientes sin ocultarlos.
