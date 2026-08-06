| name          | onboarding-mails                                                                                                                                                                                         |
| ------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| description   | Genera los borradores de correo del proceso de onboarding de Clay: la minuta de bienvenida (Mail 1, enviada dentro de las 24 hs de la primera reunión) y los seguimientos de semana 2, 4 y 6 (Mails 2-4). Usar esta skill siempre que el usuario pida armar, generar o redactar el mail/correo de bienvenida de onboarding, el seguimiento semanal de un cliente en onboarding, la minuta post-reunión, o cuando mencione frases como 'mail de bienvenida', 'seguimiento semana 2/4/6', 'borrador de onboarding para {cliente}', 'próximos pasos del cliente X', o pida actualizar el avance de conciliación/adopción de un cliente en onboarding para enviarlo por correo. También activarla si preguntan por el estado de las tareas de onboarding de una empresa y el objetivo final es comunicárselo al cliente. |
| compatibility | Requiere HubSpot MCP, Gmail MCP, Diio MCP (solo Mail 1), y Metabase Clay (execute_card sobre el dashboard 607) para todo dato de avance/checklist. No usa Clay MCP ni consultas SQL directas a sources.* — ver references/decisiones_pendientes.md para el historial de por qué. |

# Mails de Onboarding — Clay CUS

## Qué hace esta skill

Arma el borrador de uno de los cuatro correos del ciclo de onboarding y lo deja
como **borrador en Gmail**, nunca enviado. El onboarder humano siempre revisa,
edita y decide cuándo enviar — ver "Regla de oro" más abajo.

| Mail                               | Cuándo                                              | Referencia                        |
| ----------------------------------- | ---------------------------------------------------- | ---------------------------------- |
| 1 — Minuta de bienvenida           | Dentro de 24 hs de la primera reunión               | `references/mail1_bienvenida.md`  |
| 2, 3, 4 — Seguimiento semana 2/4/6 | `createdate` del ticket + 14/28/42 días, o a pedido | `references/mails_seguimiento.md` |

**Los 4 mails comparten la misma estructura base de datos** desde agosto 2026:
tabla de tareas (Área / Tarea / Estado ✅ Ok / ⏳ Pendiente), un bloque destacado
de **automatización con Cassius** (% de match y % de asientos hechos por
Cassius), y — cuando la empresa forma parte de un grupo madre/hijas — un
**resumen agregado de todas las empresas del grupo**. El detalle exacto de cada
bloque está en los archivos de referencia de cada mail; no lo dupliques acá.

## Fuente de datos de avance: SOLO el dashboard de Metabase

**Toda la información de avance de conciliación y checklist de próximos
pasos viene exclusivamente del dashboard `Onboarding - Progreso y Checklist` (id 607, `analytics.clay.cl/dashboard/607-onboarding-progreso-y-checklist`),
vía la herramienta `execute_card` de Metabase Clay.** No consultes `sources.*` directamente, no uses `clay_empresas_avance` (Clay MCP), no
inventes nombres de tabla. El detalle completo de qué card usar para qué
dato está en `references/mails_seguimiento.md` y `references/mail1_bienvenida.md`.

Esto es así porque Sole no tiene ni quiere acceso directo a producción, y
este dashboard ya integra todo lo necesario con datos reales y actualizados
— ver `references/decisiones_pendientes.md` para el historial de cómo se
llegó a esta decisión (hubo varios intentos previos con Clay MCP y con
tablas del DW dev que no funcionaron).

## Estructura de grupo: empresa madre / hijas

Algunas empresas en onboarding tienen filiales (ej. "Inversiones Foil SpA"
como madre con varias hijas). Antes de armar cualquier mail, la skill revisa
en HubSpot (companies) las propiedades `rut_empresa_madre` y
`rut_empresas_hijas` (con `hs_parent_company_id` como respaldo/cruce) para
saber si la empresa es parte de un grupo. Si lo es, el mail incluye además
un resumen agregado con todas las empresas del grupo (% avance y tareas
pendientes de cada una). El detalle paso a paso está en
`references/mails_seguimiento.md` (sección "Detectar estructura de grupo")
y se aplica igual en el Mail 1.

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
2. **Qué mail toca.** Si el usuario no lo dice explícitamente, calcúlalo desde
`createdate` (fecha de creación del ticket en HubSpot):
  - Sin reunión de bienvenida registrada todavía → Mail 1.
  - Reunión ya hecha → semana correspondiente según los días transcurridos
desde `createdate` (14/28/42 ± unos días de margen). Si cae justo
entre dos, pregunta cuál corresponde en vez de asumir.

## Paso 1 — Recolectar datos

Sigue la tabla de fuentes de datos del mail correspondiente (están en los
archivos de referencia). Reglas generales:

- **HubSpot es la fuente de verdad** para datos de cliente/contacto/fechas y
para la estructura de grupo (`rut_empresa_madre` / `rut_empresas_hijas`).
- **El dashboard 607 (cards 6206 y 6207, vía `execute_card`) es la fuente de
verdad para avance, checklist y automatización con Cassius** — no la
reemplaces por otra cosa.
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
el borrador final.

Para el diccionario completo de variables y de dónde sale cada una, ver `references/variables.md`.

## Paso 3 — Adjuntos (solo Mail 1)

El Mail 1 lleva adjunto `onboarding_clay.pptx`. Adjúntalo al crear el borrador
vía Gmail MCP — si no encuentras el archivo o el MCP no soporta adjuntar en el
draft, avisa al onboarder en vez de enviar el mail sin el adjunto.

## Paso 4 — Crear el borrador

Usa el Gmail MCP para crear un **draft** (nunca `send`) con:

- Para: `{{email_contacto_principal}}`
- CC: `ob@clay.cl`
- Asunto y cuerpo según la plantilla del mail correspondiente
- Firma del onboarder (nombre + cargo; el link de Calendly es el que el
onboarder haya indicado — ver decisión pendiente en
`references/decisiones_pendientes.md`)

Confirma al usuario que el borrador quedó listo y resume qué datos se
completaron con éxito y cuáles quedaron marcados como "no disponible", para
que sepa qué revisar antes de enviar.

## Cuando algo no calza

Esta spec todavía tiene puntos abiertos que Josefa y OPS no han cerrado del
todo (Calendly por onboarder vs. general, cómo se ingresa el comentario,
múltiples contactos por cliente, DTEs/tarjetas de todos modos ya están
cubiertos por el dashboard). Si el caso concreto que te piden depende de una
de esas decisiones, dilo explícitamente en vez de asumir en silencio — usa el
default documentado en `references/decisiones_pendientes.md` pero menciona
que es un supuesto.
