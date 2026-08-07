---
name: onboarding-mails
description: "Genera los borradores de correo del proceso de onboarding de Clay: la minuta de bienvenida (Mail 1, enviada dentro de las 24 hs de la primera reunión) y los seguimientos de semana 2, 4 y 6 (Mails 2-4). Usar esta skill siempre que el usuario pida armar, generar o redactar el mail/correo de bienvenida de onboarding, el seguimiento semanal de un cliente en onboarding, la minuta post-reunión, o cuando mencione frases como 'mail de bienvenida', 'seguimiento semana 2/4/6', 'borrador de onboarding para {cliente}', 'próximos pasos del cliente X', o pida actualizar el avance de conciliación/adopción de un cliente en onboarding para enviarlo por correo. También activarla si preguntan por el estado de las tareas de onboarding de una empresa y el objetivo final es comunicárselo al cliente, o si la empresa forma parte de un grupo madre/hijas."
compatibility: "Requiere HubSpot MCP, Gmail MCP, Diio MCP (solo Mail 1), y Metabase Clay (execute_card sobre el dashboard 607, cards 6206 y 6207) para todo dato de avance/checklist. No usa Clay MCP ni consultas SQL directas a sources.* — ver references/decisiones_pendientes.md para el historial de por qué."
---

# Mails de Onboarding — Clay CUS

## Qué hace esta skill

Arma el borrador de uno de los cuatro correos del ciclo de onboarding y lo deja
como **borrador en Gmail**, nunca enviado. El onboarder humano siempre revisa,
edita y decide cuándo enviar — ver "Regla de oro" más abajo.

| Mail | Cuándo | Referencia |
|---|---|---|
| 1 — Minuta de bienvenida | Dentro de 24 hs de la primera reunión | `references/mail1_bienvenida.md` |
| 2, 3, 4 — Seguimiento semana 2/4/6 | `createdate` del ticket + 14/28/42 días, o a pedido | `references/mails_seguimiento.md` |

## Fuente de datos de avance: SOLO el dashboard de Metabase

**Toda la información de avance de conciliación y checklist de próximos
pasos viene exclusivamente del dashboard `Onboarding - Progreso y Checklist`
(id 607, `analytics.clay.cl/dashboard/607-onboarding-progreso-y-checklist`),
vía la herramienta `execute_card` de Metabase Clay (cards 6206 y 6207).** No
consultes `sources.*` directamente, no uses `clay_empresas_avance` (Clay
MCP), no inventes nombres de tabla ni de columna. El detalle completo de qué
card usar para qué dato, y las columnas exactas (que cambiaron en agosto
2026 — no asumas que las viejas siguen vigentes), está en
`references/mails_seguimiento.md`, `references/mail1_bienvenida.md` y
`references/variables.md`.

Esto es así porque Sole no tiene ni quiere acceso directo a producción, y
este dashboard ya integra todo lo necesario con datos reales y actualizados
— ver `references/decisiones_pendientes.md` para el historial completo de
cómo se llegó a esta decisión.

## Empresas con estructura de grupo (madre/hijas)

Algunas empresas están agrupadas (una madre con una o más hijas). Antes de
armar cualquier mail, revisá si la empresa forma parte de un grupo — el
procedimiento exacto (propiedades de HubSpot a chequear, cómo armar el
resumen agregado) está en la sección "0. Detectar estructura de grupo" de
`references/mails_seguimiento.md`. Aplica tanto a los mails de seguimiento
como al Mail 1.

## Formato del cuerpo del mail (crítico)

**El borrador en Gmail tiene que llevar HTML real, con tablas `<table>` de
verdad — nunca pipes de markdown (`| Col | Col |`) pegados como texto
literal.** Gmail no interpreta markdown; si el borrador queda con barras `|`
visibles, se generó mal. Las tablas de las referencias de esta skill están
en markdown solo porque es más legible en la spec — al armar el mail real,
convertí cada una a HTML antes de pasarla al Gmail MCP. Detalle y plantilla
de ejemplo en `references/mails_seguimiento.md` (sección "Formato del
cuerpo del mail").

## Regla de oro: nunca se envía solo

Ningún mail de esta skill sale directo al cliente. El resultado final siempre
es un **borrador de Gmail** dirigido al onboarder, quien puede:

- agregar su comentario libre,
- editar cualquier parte del texto,
- cancelar y reprogramar.

Si en algún momento se te pide "envíalo directamente" sin pasar por el
onboarder, aclara que esta skill está diseñada para dejarlo en borrador y
confirma si de verdad quiere saltarse la revisión.

## Paso 0 — Identificar cliente y tipo de mail

1. **Empresa/cliente** — nombre o RUT. Si es ambiguo, búscalo en HubSpot
   (`search_crm_objects`) antes de preguntar; solo pregunta si hay más de un
   match razonable. Al cruzar contra el dashboard de Metabase, comparar
   `nombre_empresa` sin distinguir mayúsculas/minúsculas — pueden no
   coincidir exactamente (ej. "Carotrini" en HubSpot vs. "CAROTRINI SPA" en
   el dashboard).
2. **¿Es parte de un grupo?** Ver sección de arriba — revisalo antes de
   seguir, porque cambia si hace falta un bloque de resumen agregado.
3. **Qué mail toca.** Si el usuario no lo dice explícitamente, calcúlalo desde
   `createdate` (fecha de creación del ticket en HubSpot):
   - Sin reunión de bienvenida registrada todavía → Mail 1.
   - Reunión ya hecha → semana correspondiente según los días transcurridos
     desde `createdate` (14/28/42 ± unos días de margen). Si cae justo
     entre dos, pregunta cuál corresponde en vez de asumir.

## Paso 1 — Recolectar datos

Sigue la tabla de fuentes de datos del mail correspondiente (están en los
archivos de referencia). Reglas generales:

- **HubSpot es la fuente de verdad** para datos de cliente/contacto/fechas y
  para la estructura de grupo (`rut_empresa_madre`, `rut_empresas_hijas`).
- **El dashboard 607 (cards 6206 y 6207, vía `execute_card`) es la fuente de
  verdad para avance, conciliación con Cassius y checklist** — no la
  reemplaces por otra cosa, y no asumas nombres de columna viejos sin
  revisar `references/variables.md`.
- El comentario libre del onboarder es manual — pregúntaselo al usuario antes
  de generar el borrador final si no te lo dio ya. No lo redactes tú en su
  nombre salvo que te lo pida explícitamente.
- Si una empresa no aparece en los resultados del dashboard (ni con
  comparación case-insensitive ni parcial), recién ahí es válido reportar
  "sin dato" — no antes de intentar el cruce.

## Paso 2 — Armar el contenido

Cada archivo de referencia trae la estructura exacta de tablas, el tono
esperado y las variables `{{...}}` a reemplazar. Reemplaza siempre todas las
variables con datos reales — nunca dejes un `{{placeholder}}` sin resolver en
el borrador final. Recordá convertir todo a HTML real (ver sección de
arriba) antes de crear el draft.

Para el diccionario completo de variables y de dónde sale cada una, ver
`references/variables.md`.

## Paso 3 — Adjuntos (solo Mail 1)

El Mail 1 lleva adjunto `onboarding_clay.pptx`. Adjúntalo al crear el borrador
vía Gmail MCP — si no encuentras el archivo o el MCP no soporta adjuntar en el
draft, avisa al onboarder en vez de enviar el mail sin el adjunto.

## Paso 4 — Crear el borrador

Usa el Gmail MCP para crear un **draft** (nunca `send`) con:

- Para: `{{email_contacto_principal}}`
- CC: `ob@clay.cl`
- Asunto y cuerpo según la plantilla del mail correspondiente, en HTML real
- Firma del onboarder (nombre + cargo; el link de Calendly es el que el
  onboarder haya indicado — ver decisión pendiente en
  `references/decisiones_pendientes.md`)

Confirma al usuario que el borrador quedó listo y resume qué datos se
completaron con éxito y cuáles quedaron marcados como "no disponible" o con
celda vacía a propósito (ver reglas de dato faltante en cada archivo de
referencia), para que sepa qué revisar antes de enviar.

## Cuando algo no calza

Esta spec todavía tiene puntos abiertos que Josefa y OPS no han cerrado del
todo (Calendly por onboarder vs. general, cómo se ingresa el comentario,
múltiples contactos por cliente). Si el caso concreto que te piden depende
de una de esas decisiones, dilo explícitamente en vez de asumir en silencio
— usa el default documentado en `references/decisiones_pendientes.md` pero
menciona que es un supuesto.
