---
name: bloques-de-trabajo
description: Propone bloques semanales de concentración y los reserva en Google Calendar tras aprobación explícita.
user-invocable: true
---

# Bloques de trabajo

## Cuándo usarla
Para distribuir una semana de trabajo o estudio entre tareas priorizadas, y para reprogramar bloques aprobados.

## Entrada
Tareas pendientes y prioridades, plazos, duración estimada o real, disponibilidad y restricciones horarias. Consultar Google Tasks y Google Calendar mediante Zapier; zona `Europe/Madrid`.

## Procedimiento
1. Obtener los compromisos y eventos actuales del **calendario elegido**. Si hay varios calendarios y no está claro cuál usar, preguntar. No extraer detalles de calendarios compartidos que no sean necesarios.
2. Proponer bloques con: referencia de tarea/objetivo, inicio, final, duración, calendario y prioridad; incluir pausas razonables cuando correspondan.
3. Si falta una duración, sugerir una estimación señalada como estimada. Señalar conflictos y sobrecarga; no presentar una propuesta imposible como confirmada.
4. Mostrar la propuesta por Telegram y **esperar aprobación explícita** de los bloques concretos antes de crearlos o mover otros eventos.
5. Crear eventos `Trabajo · [tarea]` con referencia al objetivo/tarea en descripción si se admite. No añadir asistentes por defecto ni alterar reuniones existentes.
6. Leer los eventos por ID y verificar fechas, horas, zona, calendario y ausencia de solapamientos no autorizados. Para replanificaciones actualizar eventos existentes por ID, no crear copias.

## Resultado
Antes de la aprobación: `estado=propuesta`, lista de bloques, conflictos y pregunta clara. Tras ella: `estado, eventos_creados/actualizados, ids, enlaces, conflictos, pendientes`. La coordinadora envía una única confirmación final por Telegram; si la skill se invoca por separado, ella comunica el resultado verificado.

## Límites
No ejecutar reservas por interpretar un «quizá» como consentimiento. Ante fallos parciales, no reintentar creando eventos nuevos si ya hay un ID registrado.
