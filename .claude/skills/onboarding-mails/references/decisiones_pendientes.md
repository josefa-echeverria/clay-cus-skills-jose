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

## Otros pendientes de la spec (no bloquean el uso de la skill, pero avisar si aplican)

- Falta confirmar disponibilidad de `create_draft` en el Gmail MCP para
  onboarding. Si al intentar crear el borrador el MCP falla o no tiene esa
  función, avisa al usuario en vez de intentar enviar el mail directo.
- Josefa iba a sumar 1-2 ejemplos reales de mails para calibrar tono; hasta
  que eso llegue, el único ejemplo de referencia es el de Miguel Ángel Vargas
  (via2.cl) en `mails_seguimiento.md`. Si el tono generado se siente
  desalineado, es razón para pedir esos ejemplos, no para inventar un tono
  nuevo.
