---
name: seguimiento-onboarding
description: "Genera los borradores de los mails de avance de onboarding (semanas 2, 4 y 6) para el onboarder que ejecuta el comando, a partir de sus tareas 'Mandar mail de avance a las N semanas' abiertas en HubSpot. Descarta a los clientes con contacto reciente o sin tareas pendientes y deja nota en el ticket. Pensada para correr de forma desatendida de lunes a viernes a las 10:00 (hora de Chile), o manualmente con /seguimiento-onboarding. Requiere la skill 'onboarding-mails' en el mismo proyecto, además de HubSpot MCP, Metabase Clay, Gmail MCP y Slack MCP."
---

# Rutina: seguimiento de onboarding (semanas 2, 4 y 6)

Se invoca como `/seguimiento-onboarding [email_onboarder]`.

Esta rutina implementa el **Paso 3 del SOP "Seguimiento de Adopción en
Onboarding"** (reportes de seguimiento en semanas 2, 4 y 6). No calcula
fechas por su cuenta: sigue las tareas que el workflow de HubSpot
**"Creacion de tareas OB"** crea cuando el ticket entra a *Bienvenida y
Configuración*. Así el equipo ve en HubSpot exactamente el mismo calendario
que usa la rutina.

Esta rutina NO reemplaza a la skill `onboarding-mails`: la usa. Acá solo se
resuelve "¿a quién le toca hoy?" y se dispara la skill una vez por cliente.
Si `onboarding-mails` no está disponible en este proyecto, avisa y detente.

## Cómo encaja con el SOP y HubSpot

| Momento del SOP | Quién lo dispara | Qué hace esta rutina |
|---|---|---|
| Paso 2 — Mail 1 (minuta de bienvenida) | Onboarder, tras la reunión | Nada (fuera de alcance) |
| Paso 3 — Mails semanas 2, 4 y 6 | Tareas "Mandar mail de avance a las 2/4/6 semanas" | Arma el borrador o deja nota de por qué no aplica |
| Paso 4 — Reporte al cierre | Onboarder | Nada (fuera de alcance) |

Reparto de responsabilidades en el Paso 3:

- **La rutina:** revisa las tareas que vencen, filtra, deja el borrador en
  Gmail o una nota en el ticket, y avisa en `#team-onboarding`.
- **El onboarder:** revisa y completa el borrador, lo envía con copia a
  `ob@clay.cl`, registra la nota del envío en el ticket y **marca la tarea
  como completada**. La rutina nunca envía mails ni cierra tareas.

## 0. Identificar al onboarder

- Si se pasó `$ARGUMENTS` (un email o nombre), usa eso.
- Si no, resuelve el onboarder desde el usuario de HubSpot/Gmail conectado.
- Si no puedes determinarlo con confianza, pregúntalo. No adivines ni
  proceses las tareas de otro onboarder.

Resuelve su `owner_id` de HubSpot (`search_owners`) antes de seguir.

## 1. Traer las tareas de avance que vencen

Busca en HubSpot (`search_crm_objects`, objectType `TASK`) las tareas que
cumplan todo esto:

- `hubspot_owner_id` = owner del paso 0.
- `hs_task_status` distinto de `COMPLETED`.
- `hs_task_subject` empieza con **"Mandar mail de avance a las"**
  (`2 semanas`, `4 semanas` o `6 semanas`).
- `hs_timestamp` (fecha de vencimiento) **hoy o antes**, en zona horaria
  America/Santiago. Incluye las vencidas: si la rutina no corrió un día, la
  tarea sigue abierta y se toma al día siguiente.

El número de semana `{{n}}` sale del asunto de la tarea (2, 4 o 6). No lo
recalcules desde `createdate` ni desde `fecha_inicio_ob`.

Para cada tarea, trae el **ticket de onboarding asociado** y, desde el
ticket, la empresa y el contacto principal. Del ticket lee también
`hs_lastcontacted` (paso 2) y `hs_pipeline_stage`.

Descarta y reporta como `Error` en el resumen:

- Tareas sin ticket asociado (`Error: tarea sin ticket asociado`).
- Tareas cuyo ticket ya está en *Cierre* o *No completado*
  (`Error: ticket cerrado con tarea abierta`), para que el onboarder cierre
  la tarea.

Si hay más de una tarea abierta de avance para el mismo ticket (por ejemplo,
la de 2 semanas quedó sin cerrar y ya venció la de 4), trabaja **solo la de
semana más alta** y menciona la otra en pendientes: el mail nuevo reemplaza
al anterior.

## 2. Filtro 1: ¿el cliente ya tuvo contacto esta semana?

El mail de avance existe para retomar el contacto con clientes que no han
sabido de nosotros. Si el onboarder ya habló con el cliente, no corresponde
mandarle otro mail.

- Usa `hs_lastcontacted` del ticket (*Last contacted date*: llamada,
  reunión, mail, chat o WhatsApp registrado). **No uses
  `hs_lastactivitydate`**: esa propiedad también cuenta tareas, incluidas
  las que crean el workflow y DIIO, y marcaría casi todo como "con
  actividad".
- Si `hs_lastcontacted` cae dentro de los **últimos 7 días** (hoy incluido,
  America/Santiago), **no generes el borrador**:
  - Crea una **nota en el ticket** con este texto:
    `Seguimiento semana {{n}}: no aplica. Cliente registra actividad los últimos 7 días (último contacto: {{dd-mm-aaaa}}).`
  - **No cierres la tarea**: la cierra el onboarder.
  - En el resumen: `No aplica seguimiento: cliente registra actividad los últimos 7 días ({{fecha}})`.
- Si `hs_lastcontacted` está vacía o tiene más de 7 días, pasa al filtro 2.

## 3. Filtro 2: ¿tiene tareas pendientes?

El mail de avance es, ante todo, un recordatorio de lo que el cliente tiene
pendiente. Revisa `tareas_pendientes` de la empresa en la card **6206** del
dashboard 607 (`execute_card`, filtrando localmente por `nombre_empresa`
sin distinguir mayúsculas, igual que en `onboarding-mails`):

- **Una o más tareas pendientes:** pasa al paso 4 y se arma el mail
  completo.
- **Cero tareas pendientes** (checklist al 100%): no generes el borrador.
  - Nota en el ticket:
    `Seguimiento semana {{n}}: no aplica. Cliente sin tareas pendientes en el checklist de onboarding.`
  - No cierres la tarea.
  - En el resumen: `No aplica seguimiento: sin tareas pendientes`.
- **La empresa no aparece en el dashboard 607:** no la descartes en
  silencio. Repórtala como `Error: empresa no encontrada en dashboard 607`.

Si el usuario pide explícitamente generar el mail de un cliente puntual
("genera el de {{cliente}} igual"), sáltate los filtros 1 y 2 solo para ese
caso.

## 4. Evitar duplicados

Antes de generar un borrador, revisa los drafts de Gmail con asunto
`¿Cómo va {{nombre_empresa}} en Clay? — Semana {{n}}`. Si ya existe, no
crees otro: repórtalo como `Ya existía` en el resumen.

## 5. Generar el borrador

Para cada cliente que pasó los dos filtros, sigue el flujo de **Mails de
seguimiento** de la skill `onboarding-mails`
(`references/mails_seguimiento.md`), sin saltarte pasos: dashboard 607,
estructura de grupo, tabla de avance, Cassius, próximos pasos y
sugerencias (con su fallback). El número de semana `{{n}}` ya viene del
paso 1.

**Comentario libre del onboarder:**

- Corrida **interactiva**: pídele su comentario para cada cliente antes de
  cerrar el borrador.
- Corrida **desatendida**: deja el marcador
  `[[Agregar tu comentario acá antes de enviar]]` y súmalo a pendientes.

El resultado siempre es un **borrador de Gmail**, nunca un envío.

## 6. Resumen final

Al terminar, entrega una tabla con todas las tareas revisadas, incluidas
las descartadas, para que el onboarder vea qué pasó con cada una:

| Cliente | Semana | Resultado |
|---|---|---|
| {{nombre_empresa}} | 2 / 4 / 6 | Borrador creado · Ya existía · No aplica seguimiento: cliente registra actividad los últimos 7 días ({{fecha}}) · No aplica seguimiento: sin tareas pendientes · Error: {{detalle}} |

Y una lista de **pendientes para el onboarder**:

- Borradores por revisar (comentario libre, celdas vacías a propósito,
  datos no disponibles).
- Tareas que debe cerrar en HubSpot: las de clientes con borrador (una vez
  enviado el mail) y las marcadas "No aplica" (ya tienen nota en el ticket).
- Tareas de semanas anteriores que quedaron abiertas.

## 7. Aviso en Slack

Envía el resumen a **`#team-onboarding`** en cada corrida, haya o no
clientes. Slack no muestra tablas en Markdown: usa una línea por cliente.

```
:memo: *Seguimiento de onboarding — {{onboarder}} — {{fecha}}*

• *{{nombre_empresa}}* · Semana {{n}} · {{resultado}}
• ...

*Pendientes:*
• {{item_pendiente}}
(o "Sin pendientes 🎉")
```

Si no había tareas de avance para hoy, manda igual:
`Hoy no había mails de avance por generar para {{onboarder}}.`

Si falla el envío a Slack, no reviertas borradores ni notas: reporta el
fallo en el resumen que devuelves.

## Cuándo corre: días hábiles a las 10:00

La rutina corre **de lunes a viernes a las 10:00 (hora de Chile,
America/Santiago)**, una vez al día. Cron de referencia:

```
CRON_TZ=America/Santiago 0 10 * * 1-5
```

- **Fin de semana:** no corre. Las tareas que vencen sábado o domingo
  quedan abiertas y se toman el lunes, porque el paso 1 incluye las
  vencidas.
- **Feriados:** al partir, revisa si hoy es feriado nacional en Chile. Si lo
  es, no proceses tareas ni crees notas o borradores: solo manda a
  `#team-onboarding` el aviso
  `Hoy es feriado: el seguimiento de onboarding se procesa el próximo día hábil.`
  y termina. Las tareas se toman el siguiente día hábil.
- **Filtro de 7 días:** se cuenta en días corridos, no hábiles (incluye
  fines de semana).
- **Ejecución manual:** se puede correr a mano cualquier día y a cualquier
  hora (por ejemplo, para recuperar un día en que no corrió). Como evita
  duplicados, no genera mails repetidos.

## Notas para correrla de forma desatendida

- Fija el email del onboarder como argumento (una ejecución por onboarder).
- Respeta el horario de arriba: una sola corrida por día hábil.
- Herramientas permitidas: HubSpot (lectura + crear notas), Metabase Clay,
  Gmail (solo borradores) y Slack (mensajes).
