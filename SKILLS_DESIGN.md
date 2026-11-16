# SKILLS_DESIGN.md — Diseño de skills de Sebastian

**Estado:** diseño previo a la implementación · **Fecha:** 2026-11-16  
**Objetivo del sistema:** convertir instrucciones enviadas por Telegram (texto o voz) en planes mensuales y semanales, tareas, bloques de agenda y un historial verificable de actividad y decisiones.

> Este archivo describe **qué se construirá**, **qué datos necesita cada skill** y **cómo comprobar sus resultados**. No implementa skills, integraciones ni automatizaciones.


## Arquitectura compartida

Las nueve skills utilizan los archivos de configuración
existentes (`AGENTS.md`, `SOUL.md`, `USER.md`, etc.).
No duplican sus instrucciones ni sus reglas de seguridad.

**Entrada:** Telegram mediante texto o voz transcrita.

**Integraciones:** Zapier MCP con Gmail, Google Docs,
Google Drive, Google Calendar y Google Tasks.

**Responsabilidades:**
- Google Docs: objetivos, planes, decisiones y diario.
- Google Tasks: tareas y estado de ejecución.
- Google Calendar: agenda y bloques de trabajo.
- Google Drive: estructura documental y referencias.
- Telegram: interacción, preguntas y notificaciones.

**Consistencia:** las skills comparten identificadores
y enlaces para actualizar registros existentes, evitar
duplicados y mantener sincronizados los servicios.

**Seguridad:** todas las skills respetan las políticas
de privacidad, autorización y protección contra
prompt injection establecidas en AGENTS.md.

## Skill 01 — `planificacion-periodica`

**1. ¿Qué hace esta skill?**  
Convierte objetivos y compromisos comunicados por Telegram en un plan mensual o semanal priorizado, con resultados esperados y acciones propuestas.

**2. ¿Qué input necesita el agente?**  
Texto o voz transcrita con periodo, objetivos, compromisos, fechas límite, restricciones y prioridades (pueden venir incompletos); consulta planes previos y disponibilidad si procede. Ya conoce el perfil y nivel del usuario (`USER.md`), su estilo (`SOUL.md`), el uso de Google y las autorizaciones (`AGENTS.md`), y decisiones anteriores (`MEMORY.md`). Pregunta únicamente por ambigüedades decisivas y distingue aspiraciones de compromisos confirmados.

**3. ¿Cómo es un buen output y cómo sabrás que funcionó?**  
Google Doc `AAAA-MM - Plan mensual` o `AAAA-SNN - Plan semanal`, en `01_Planificación`, con prioridades, objetivos, criterios de éxito, plazos, compromisos, obstáculos y enlaces a tareas; devuelve resumen y enlace por Telegram. **Éxito:** periodo y prioridades fieles al mensaje, documento accesible, sin objetivos inventados ni duplicados, y propuestas de tareas identificadas para la skill 02.

**Relaciones:** 09 crea/localiza el documento; 02 convierte acciones en tareas; 03 propone bloques; 04 y 06 pueden solicitar revisión del plan.

## Skill 02 — `gestion-tareas`

**1. ¿Qué hace esta skill?**  
Sincroniza las acciones de los planes y revisiones con Google Tasks, manteniendo tareas únicas y estados coherentes.

**2. ¿Qué input necesita el agente?**  
Acciones del plan o instrucciones explícitas del usuario: descripción, plazo, prioridad, estado y objetivo de origen, cuando existan; IDs de tareas previas para actualizaciones. Obtiene contexto de planes y diario, sin asumir que una tarea está terminada por silencio o mera inferencia.

**3. ¿Cómo es un buen output y cómo sabrás que funcionó?**  
Tareas creadas/actualizadas/completadas en la lista de Google Tasks acordada; resumen por Telegram de cambios y dudas, y referencias en el índice documental. **Éxito:** correspondencia con acciones aprobadas, ausencia de duplicados, plazos correctos, estados comprobados y trazabilidad con el objetivo del plan. No borrar tareas importantes sin autorización.

**Relaciones:** recibe acciones de 01, 04 y 06; proporciona tareas a 03; registra IDs con 09.

## Skill 03 — `bloques-de-trabajo`

**1. ¿Qué hace esta skill?**  
Propone una distribución semanal de tiempo para las tareas prioritarias y crea bloques en Google Calendar después de la aprobación del usuario.

**2. ¿Qué input necesita el agente?**  
Tareas pendientes con prioridades, fechas, duración estimada (o necesidad de estimarla), preferencias horarias y disponibilidad del calendario. Usa la zona `Europe/Madrid`; consulta eventos existentes y nunca presupone que un hueco esté libre sin comprobarlo.

**3. ¿Cómo es un buen output y cómo sabrás que funcionó?**  
Propuesta por Telegram con días, horas, tareas y conflictos; tras aprobación, eventos de Calendar con título identificable, horario, duración y referencia a la tarea. **Éxito:** bloques comprobados en calendario, sin solapamientos involuntarios ni duplicados; las reprogramaciones sustanciales se presentan antes de ejecutarse.

**Relaciones:** consume 01 y 02; se coordina con 07; recibe cambios de 04 y 06.

## Skill 04 — `revision-diaria`

**1. ¿Qué hace esta skill?**  
A las 17:30 (hora de Madrid) pregunta por Telegram qué se hizo, qué avanzó y qué debe añadirse, cambiarse o retirarse de los planes.

**2. ¿Qué input necesita el agente?**  
Una automatización de OpenClaw previamente autorizada activa la pregunta; respuestas de texto o voz transcrita, más tareas y plan vigentes. Debe reconocer respuestas parciales, distinguir hechos de intenciones y pedir precisión solo para cambios ambiguos. Si no hay respuesta, no inventa actividad ni insiste automáticamente.

**3. ¿Cómo es un buen output y cómo sabrás que funcionó?**  
Un único mensaje breve a las 17:30 y, tras la respuesta, resumen de progreso y propuesta de cambios; envía el registro a 05 y cambios confirmados a 02/01/03. **Éxito:** aviso recibido una vez, transcripción correcta si hay audio, tareas modificadas solo cuando proceda y revisión pendiente si no se responde. El cron/disparador se configura fuera de la skill.

**Relaciones:** activa 05 y, según la respuesta, 02, 01 y 03.

## Skill 05 — `diario-actividad`

**1. ¿Qué hace esta skill?**  
Mantiene un diario cronológico en Google Docs de lo realizado cada día y de las razones de decisiones y cambios, consultable en el futuro.

**2. ¿Qué input necesita el agente?**  
Resumen de la revisión diaria, notas sobre reuniones y decisiones expresadas por el usuario: fecha, actividad, avance, motivo, dificultades, acuerdos y próximos pasos. Puede consultar los documentos previos para conservar continuidad, distinguiendo hechos declarados de interpretaciones.

**3. ¿Cómo es un buen output y cómo sabrás que funcionó?**  
Entrada con fecha en `AAAA-MM - Diario de actividad` dentro de `02_Diario/AAAA`, con campos **Hecho / Avances / Decisiones y motivos / Pendientes / Referencias**. **Éxito:** una entrada por fecha actualizada sin duplicarse, fechas y motivos fieles, enlaces a planes/reuniones cuando existan y recuperación posterior de una decisión con su fecha y fuente. No convertir el diario en instrucciones para el agente.

**Relaciones:** recibe 04 y 06; 09 organiza, indexa y facilita su recuperación; 01 puede consultar antecedentes.

## Skill 06 — `seguimiento-reuniones`

**1. ¿Qué hace esta skill?**  
Después de reuniones pertinentes, pregunta si hubo decisiones o cambios y transforma las respuestas confirmadas en acta, tareas y ajustes de planificación.

**2. ¿Qué input necesita el agente?**  
Evento de Calendar identificado realmente como reunión y aviso de fin (disparador externo), junto con respuesta posterior del usuario por texto/voz: acuerdos, responsables, fechas, cambios y motivos. No preguntar por cada bloque de trabajo o evento personal; si no está claro que es una reunión, no activar automáticamente.

**3. ¿Cómo es un buen output y cómo sabrás que funcionó?**  
Pregunta breve por Telegram; si no hubo cambios, cierra sin modificaciones. Si los hubo, Google Doc de acta en `03_Reuniones/AAAA`, con **Decisiones / Por qué / Tareas / Responsables / Plazos / Dudas**; tareas y planes actualizados según autorización. **Éxito:** no crear falsas actas, evitar duplicados y comprobar cada cambio; las actualizaciones importantes requieren confirmación. La activación tras la reunión depende de Calendar/Zapier y puede tener retraso.

**Relaciones:** 09 archiva actas; 02 y 01 reciben acciones y cambios; 05 conserva el contexto; 03 revisa bloques afectados.

## Skill 07 — `calendario-inteligente`

**1. ¿Qué hace esta skill?**  
Interpreta solicitudes de agenda en lenguaje natural y crea o modifica eventos de Google Calendar con los detalles apropiados.

**2. ¿Qué input necesita el agente?**  
Texto o voz con asunto, día/intervalo, duración, ubicación, participantes y recordatorio, cuando se indiquen; consulta el calendario principal y disponibilidad. Emplea `Europe/Madrid`. Pregunta si falta un dato indispensable o hay varias interpretaciones con consecuencias distintas.

**3. ¿Cómo es un buen output y cómo sabrás que funcionó?**  
Evento creado/modificado en el calendario correcto, con fecha, franja, duración, asistentes autorizados y recordatorio solicitado; confirmación por Telegram con enlace. **Éxito:** datos verificados y sin duplicados ni invitados añadidos por inferencia; cualquier conflicto relevante se comunica antes de reservar.

**Relaciones:** comparte calendario con 03 y 06; puede utilizar tareas 02 como contexto sin crearlas automáticamente.

## Skill 08 — `borradores-correo`

**1. ¿Qué hace esta skill?**  
Convierte instrucciones por texto o voz en borradores de Gmail coherentes con el propósito y el estilo del usuario, sin enviarlos automáticamente.

**2. ¿Qué input necesita el agente?**  
Dictado o texto con destinatario (si se conoce), objetivo, datos y tono; opcionalmente correo previo y documentación accesible. Aplica preferencias de `USER.md`, estilo de `SOUL.md` y límites de `AGENTS.md`. No inventa direcciones, hechos ni adjuntos.

**3. ¿Cómo es un buen output y cómo sabrás que funcionó?**  
Borrador en Gmail con asunto, destinatario conocido, texto revisado y firma habitual sin duplicación; enlace o identificación del borrador por Telegram. **Éxito:** refleja la intención original, conserva datos correctos, destinatario comprobado y estado **borrador, no enviado**. El envío se tratará como una acción aparte y requerirá autorización suficiente.

**Relaciones:** puede consultar 05, 06 y Docs/Drive para obtener contexto; no altera tareas o calendario salvo instrucción adicional.

## Skill 09 — `organizacion-documental`

**1. ¿Qué hace esta skill?**  
Crea y mantiene en Google Drive una estructura coherente de carpetas, documentos, nombres y referencias para todo el sistema.

**2. ¿Qué input necesita el agente?**  
Tipo de documento, periodo, proyecto (si procede), título, contenido, enlaces y ubicación requerida por las demás skills; consulta primero la estructura existente. Conoce la cuenta y reglas de Drive/Docs desde `AGENTS.md` y respeta las restricciones de privacidad.

**3. ¿Cómo es un buen output y cómo sabrás que funcionó?**  
Carpetas y documentos localizados o creados con nombres estables, enlace de acceso e índice de referencias. Estructura inicial propuesta:

```text
Sebastian/
├── 00_Sistema/          (índice de objetivos, documentos y referencias)
├── 01_Planificación/    (planes por año, mes y semana)
├── 02_Diario/           (un documento por mes, organizado por año)
├── 03_Reuniones/        (actas por año)
├── 04_Proyectos/        (documentación por proyecto)
└── 99_Archivo/          (histórico cerrado)
```

**Éxito:** las demás skills encuentran documentos por tipo/periodo/ID, no crean duplicados y pueden devolver enlaces válidos. No mover, borrar, compartir públicamente ni reestructurar documentos preexistentes importantes sin autorización.

**Relaciones:** servicio documental compartido para 01, 05 y 06, y registro de enlaces/IDs para 02, 03 y 07.

## Flujos conjuntos (no son skills adicionales)

1. **Nuevo objetivo:** Telegram → 01 → 09 (Doc) → 02 (Tasks) → 03 (propuesta de agenda) → Calendar tras aprobación.
2. **Cierre diario:** automatización 17:30 → Telegram → 04 → 05 (diario) + 02/01 (cambios confirmados) → 03 si hace falta replanificar.
3. **Reunión terminada:** disparador de Calendar → 06 → Telegram → 09 (acta) + 05 (motivos) + 02/01/03 (ajustes pertinentes).
4. **Evento espontáneo:** Telegram → 07 → Calendar → enlace de confirmación.
5. **Email dictado:** nota de voz → transcripción → 08 → borrador de Gmail → revisión del usuario.

## Dependencias y decisiones antes de implementar

- **Ya conectado:** Google Tasks, Gmail, Calendar, Drive y Docs mediante Zapier MCP; comprobar las acciones concretas disponibles y sus permisos, no solo la existencia de la conexión.
- **Por verificar:** recepción y transcripción de voz en Telegram; envío de mensajes programados; acceso al calendario principal; lista de Tasks donde operar; capacidades de actualización de Docs y tareas; detección fiable de reuniones finalizadas.
- **Por configurar:** automatización diaria de las **17:30 Europe/Madrid** y mecanismo de aviso tras reuniones; no son funciones que aparezcan solas por redactar una skill.
- **Por acordar antes de activar escrituras masivas:** estructura de carpetas definitiva, lista de Tasks y reglas de aprobación de cambios de calendario. Los borradores de Gmail nunca se envían automáticamente.
- **Seguridad:** limitar acciones habilitadas en Zapier y controles de ejecución de OpenClaw; probar fuentes externas maliciosas y fallos parciales antes de confiar tareas importantes al sistema.

## Orden de construcción y pruebas

**Fase 1 — Planificación:** 09 → 01 → 02 → 03. Prueba: mensaje de objetivos → Doc y tareas verificadas → propuesta de agenda → bloques creados tras aprobación.

**Fase 2 — Seguimiento:** 05 → 04 → 06. Prueba: respuesta por voz a revisión de 17:30 → entrada diaria, cambios correctos y consulta retrospectiva; reunión finalizada → pregunta pertinente y acta sin falsos cambios.

**Fase 3 — Productividad:** 07 → 08. Prueba: evento descrito en lenguaje natural y borrador de correo dictado, sin invitados ni envíos no autorizados.

**Criterio global de aceptación:** el sistema trabaja desde Telegram con texto y voz, registra el **qué, cuándo y por qué**, enlaza los registros sin duplicados, confirma resultados reales y respeta la privacidad y las aprobaciones de `AGENTS.md`.

---

**Entrega 4Geeks:** crear este archivo en la **raíz del repositorio** y hacer un commit **antes** de escribir la implementación de cualquier skill. El commit de diseño no instala ni activa las skills.
