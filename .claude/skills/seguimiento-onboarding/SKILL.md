---
name: seguimiento-onboarding
description: "Genera los borradores de seguimiento de onboarding (semana 2, 4 y 6) para TODOS los tickets del onboarder que ejecuta este comando, calculando automáticamente a quién le toca hoy. Pensada para correr manualmente (/seguimiento-onboarding) o de forma desatendida vía cron. Requiere la skill 'onboarding-mails' instalada en el mismo proyecto, además de HubSpot MCP, Clay MCP, clay-dw:product-health y Gmail MCP."
---

# Rutina: seguimiento de onboarding (semana 2/4/6)

Se invoca como `/seguimiento-onboarding [email_onboarder]`. El argumento con
el email es opcional si Claude ya puede identificar al onboarder por el
usuario de HubSpot/Gmail conectado en la sesión; si no puede, lo pide (ver
paso 0).

Esta rutina NO reemplaza a la skill `onboarding-mails` — la usa. Aquí solo se
resuelve la parte de "¿a quién le toca hoy?" para un onboarder específico y
se dispara la skill una vez por cada cliente que corresponda. Si `onboarding-mails`
no está disponible en este proyecto, avisa y detente: no reconstruyas su lógica
desde cero acá.

## 0. Identificar al onboarder

- Si se pasó `$ARGUMENTS` (un email o nombre), usa eso.
- Si no, resuelve el onboarder desde el usuario de HubSpot/Gmail conectado en
  este entorno.
- Si no puedes determinarlo con confianza, pregúntalo — no adivines ni
  proceses los tickets de otro onboarder por error.

Resuelve el `owner_id` de HubSpot correspondiente (`search_owners` o
equivalente) antes de filtrar tickets.

## 1. Traer los tickets activos del onboarder

Busca en HubSpot los tickets del embudo/pipeline de onboarding (si no sabes
el `pipeline_id` de memoria, búscalo por nombre — no lo inventes ni reutilices
el de otro pipeline como SaaS/Servicio/Upsell) que:

- pertenecen al `owner_id` resuelto en el paso 0, y
- están en un stage activo (no cerrado/ganado/perdido).

Para cada ticket, obtén su fecha de creación (`createdate`, propiedad nativa
de HubSpot) y el `deal`/organización asociada. No uses `fecha_inicio_ob` para
este cálculo — esa propiedad es custom y depende de que alguien la cargue a
mano; `createdate` la pone HubSpot automáticamente al crear el ticket, así
que es la fuente confiable para saber "hace cuánto entró este cliente al
embudo de onboarding".

## 2. Calcular a quién le toca hoy

Para cada ticket, calcula los días transcurridos entre `createdate` y hoy:

| Días transcurridos | Mail que corresponde |
|---|---|
| 14 | Semana 2 |
| 28 | Semana 4 |
| 42 | Semana 6 |

Usa coincidencia **exacta** de día, no un rango — esta rutina está pensada
para correr una vez al día (manual o por cron), así que el día exacto ya es
suficiente y evita procesar el mismo cliente dos veces por una ventana
demasiado ancha.

Ignora los tickets que no calzan exactamente con 14, 28 o 42 días. Si el
usuario pide explícitamente "generá el de {{cliente}} aunque no le toque
hoy", trátalo como una ejecución manual puntual y sáltate el filtro de fecha
solo para ese caso.

## 3. Evitar duplicados

Antes de generar un borrador nuevo, revisa los drafts existentes en Gmail con
asunto `¿Cómo va {{nombre_empresa}} en Clay? — Semana {{n}}` para ese cliente
y esa semana específica. Si ya existe uno, no crees otro — repórtalo como
"ya existía" en el resumen final en lugar de duplicarlo.

## 4. Generar cada borrador

Para cada ticket que sí corresponde hoy (y no tiene draft duplicado), sigue
el flujo de **Mails 2, 3 y 4** de la skill `onboarding-mails`
(`references/mails_seguimiento.md`), sin saltarte ningún paso de esa skill:
fuentes de datos, fallback de product health, tabla de avance, tabla de
próximos pasos y sugerencias de uso.

La sección "Trigger de envío" de esa skill todavía describe el cálculo en
base a `fecha_inicio_ob` (así estaba en la spec original de Josefa). Esta
rutina reemplaza esa parte: el número de semana ya viene decidido por el
paso 2 de acá arriba, usando `createdate`. No vuelvas a calcular la semana
con `fecha_inicio_ob` al entrar a `onboarding-mails` — usa el `{{n}}` que ya
determinaste.

**Comentario libre del onboarder:**
- Si estás corriendo de forma **interactiva** (hay una persona respondiendo
  en el chat), pregúntale su comentario para cada cliente antes de cerrar el
  borrador de ese cliente.
- Si estás corriendo de forma **desatendida** (cron, sin nadie para
  responder), no te quedes esperando una respuesta que no va a llegar: deja
  el borrador con el marcador `[[Agregar tu comentario acá antes de
  enviar]]` en el lugar del comentario, y súmalo a la lista de pendientes del
  resumen final.

El resultado siempre es un **borrador de Gmail**, nunca un envío — esa regla
de `onboarding-mails` aplica también acá, sin excepción, corras manual o por
cron.

## 5. Resumen final

Al terminar de recorrer todos los tickets del onboarder, entrega una tabla
con:

| Cliente | Semana | Resultado |
|---|---|---|
| {{nombre_empresa}} | 2/4/6 | Borrador creado / Ya existía / Sin dato de X (marcado en el borrador) / Error: {{detalle}} |

Y una lista aparte de "Pendientes para revisar antes de enviar" (comentarios
sin completar, datos marcados como no disponibles, adjuntos que fallaron,
etc.), para que el onboarder sepa exactamente qué mirar antes de aprobar los
borradores.

## Notas para correrla por cron (opcional)

Si se agenda con cron usando `claude --print`, conviene:

- Fijar el `$ARGUMENTS` con el email del onboarder explícitamente (no
  depender de detectar el usuario conectado en una sesión no interactiva).
- Correrla una vez al día — no más seguido, porque el filtro de día exacto
  del paso 2 asume una sola pasada diaria.
- Redirigir la salida a un log y avisar al canal del equipo si el resumen
  final contiene algún "Error".
- Limitar las herramientas permitidas a las de HubSpot, Clay, clay-dw y
  Gmail (drafts), para que la ejecución desatendida no pueda tocar nada fuera
  de este flujo.
