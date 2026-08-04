# Decisiones pendientes para OPS

Estas son las 5 decisiones que la spec original (Josefa, CUS, v1.0 — julio
2026) deja abiertas en su sección 5.1. Mientras OPS no las resuelva
formalmente, la skill opera con el default indicado en la columna derecha.
Si el usuario te pide algo que choca con un default, dilo explícitamente en
vez de aplicar el default en silencio.

| # | Decisión pendiente | Default actual de la skill |
|---|---|---|
| 1 | ¿Cuenta de servicio Clay para el MCP o siempre DW? | Usar DW (`clay-dw:product-health` y consultas directas), tal como recomienda la propia spec. Usar `clay_empresas_avance` vía Clay MCP para lo que no está en el DW. |
| 2 | ¿El comentario del onboarder se ingresa en el chat o en un formulario previo? | En el chat: pregúntaselo al usuario/onboarder como parte de la conversación antes de generar el borrador final. |
| 3 | ¿Calendly por onboarder o uno general de Clay? | Por onboarder: usa el link que el onboarder te indique en la conversación. Si no lo tienes, pregúntalo — no inventes ni dejes un link genérico. |
| 4 | ¿Cómo se maneja el envío si el cliente tiene múltiples usuarios? | Enviar solo al contacto principal (`{{email_contacto_principal}}` de HubSpot) en CC + Para tal como está en la spec. Si el onboarder pide incluir a más gente, agrégalos en CC, pero no lo hagas por defecto. |
| 5 | ¿El skill puede ejecutarse manualmente fuera de las fechas automáticas? | Sí — esta skill no depende de un trigger automático real (no hay integración de cron); siempre se ejecuta a pedido del onboarder en el chat, calculando igual la fecha/semana correspondiente para dar contexto. |

## Decisiones sobre la migración al dashboard de Metabase (agosto 2026)

Estas se agregaron cuando se cambió la fuente de avance/checklist al
dashboard "Onboarding - Progreso y Checklist" (Metabase, dashboard_id 607).
A diferencia de las 5 anteriores, estas sí fueron confirmadas explícitamente
por el usuario en esa conversación — no son un default sin validar.

| # | Decisión | Resuelto como |
|---|---|---|
| 6 | ¿El dashboard reemplaza por completo a `clay_empresas_avance`, o queda como respaldo? | El dashboard (`staging_marts.organizations_onboarding_progress` y `organizations_checklist_status`) es la fuente **principal**. `clay_empresas_avance` (Clay MCP) queda como **fallback** solo para las métricas de avance cuando el dashboard no tiene fila para esa empresa. El checklist de tareas no tiene fallback — si el dashboard no tiene filas, se marca "no disponible", no se inventa ni se recalcula desde otra fuente. |
| 7 | El dashboard solo distingue `Ok`/`Pendiente`, pero la spec original de los Mails 2-4 tenía un tercer estado `🔄 En progreso` — ¿qué se hace? | **Confirmado:** se elimina `🔄 En progreso`. Se usan solo `✅ Ok` y `⏳ Pendiente`, tal como los devuelve el dashboard. Decisión cerrada — no se reconstruye ese estado intermedio cruzando otras fuentes. |
| 8 | ¿Cómo se muestran las empresas hijas (grupo empresarial) en el mail? | Una tabla de checklist completa por cada hija, mismo formato que la de la madre, cada una con su propio subtítulo. No se resume en una sola fila por hija. Se agrega solo en los Mails 2, 3 y 4 — el Mail 1 no incluye hijas por defecto (poco probable que haya datos de grupo tan temprano en el onboarding). |
| 9 | ¿Cómo se identifica si una empresa "tiene hijas"? | Se consulta `staging_marts.organizations_checklist_grupo` filtrando `nombre_grupo = {{nombre_empresa}}` y `rol = 'Hija'` (query 3 de `queries_dashboard.md`). Si no devuelve filas, se asume que no tiene grupo y se omite la sección sin mencionarla. |

## Decisiones confirmadas al probar la skill con MIGTRA SPA y FAUCON IMPORT COMPANY SPA (agosto 2026)

| # | Decisión | Resuelto como |
|---|---|---|
| 10a | Las tablas de los mails se veían "corridas" en Gmail | **Confirmado:** el draft de Gmail siempre debe usar `htmlBody` con una tabla `<table>` real (bordes, padding, encabezado con fondo gris), nunca markdown. Aplica a las 4 plantillas (Mail 1 y Mails 2-4) — Gmail no renderiza tablas markdown, quedan como texto plano con pipes. Ver Paso 4 de `SKILL.md`. |
| 10b | Mensaje sobre el objetivo de las 8 semanas de onboarding | **Confirmado, solo para Mail 1 por ahora:** agregar siempre un párrafo explicando que el objetivo de las 8 semanas es automatizar al máximo la gestión contable, y que se irá mostrando el avance semana a semana (conecta con los Mails 2-4). No se confirmó extenderlo como texto fijo a los Mails 2-4 — si se quiere ese eco en los de seguimiento, es una decisión aparte todavía sin tomar. |
| 10c | La tabla de avance mostraba `% Conciliación Cassius` y `% Conciliación por usuario` como dos filas separadas | **Confirmado:** se reemplazan por una sola fila `{{pct_match}}` = `(movimientos_totales - movimientos_sin_match) / movimientos_totales × 100`. Aplica a todas las empresas, no es condicional por categoría. |
| 11 | ¿Empresas sin módulo contable activo deberían ver campos de asientos/DTE/sugerencias de uso en el mail? | **Confirmado:** no. Se determina consultando `product_category` en `staging.organizations` (query 6 de `queries_dashboard.md`). Si es `Software Gestión Financiera` **o** `API Bancaria & SII`, se omiten en la tabla de avance las filas de asientos y DTE, y se omite por completo la tabla de sugerencias de uso (Sección 4 de `mails_seguimiento.md`). Para `Software Gestión Financiera & Contable`, `Servicio con/sin RRHH` y `Sin categoría`, el mail va completo sin cambios. |

## Otros pendientes de la spec (no bloquean el uso de la skill, pero avisar si aplican)

- Falta confirmar disponibilidad de `create_draft` en el Gmail MCP para
  onboarding. Si al intentar crear el borrador el MCP falla o no tiene esa
  función, avisa al usuario en vez de intentar enviar el mail directo.
- Josefa iba a sumar 1-2 ejemplos reales de mails para calibrar tono; hasta
  que eso llegue, el único ejemplo de referencia es el de Miguel Ángel Vargas
  (via2.cl) en `mails_seguimiento.md`. Si el tono generado se siente
  desalineado, es razón para pedir esos ejemplos, no para inventar un tono
  nuevo.
