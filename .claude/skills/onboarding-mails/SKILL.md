---
name: onboarding-mails
description: "Genera los borradores de correo del proceso de onboarding de Clay: la minuta de bienvenida (Mail 1, enviada dentro de las 24 hs de la primera reunión) y los seguimientos de semana 2, 4 y 6 (Mails 2-4). Usar esta skill siempre que el usuario pida armar, generar o redactar el mail/correo de bienvenida de onboarding, el seguimiento semanal de un cliente en onboarding, la minuta post-reunión, o cuando mencione frases como 'mail de bienvenida', 'seguimiento semana 2/4/6', 'borrador de onboarding para {cliente}', 'próximos pasos del cliente X', o pida actualizar el avance de conciliación/adopción de un cliente en onboarding para enviarlo por correo. También activarla si preguntan por el estado de las tareas de onboarding de una empresa y el objetivo final es comunicárselo al cliente."
compatibility: "Requiere HubSpot MCP, Clay MCP, Gmail MCP, Diio MCP (solo Mail 1) y clay-dw:product-health (Mails 2-4)."
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

Este documento es una spec viva escrita por Josefa (CUS) — hay decisiones de
producto todavía sin resolver por OPS (ver `references/decisiones_pendientes.md`).
La skill usa un default razonable para cada una y lo deja explícito en el
borrador o en tu respuesta al usuario, para que sea fácil de corregir.

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

Antes de tocar cualquier fuente de datos, confirma (o infiere de la
conversación):

1. **Empresa/cliente** — nombre o RUT. Si es ambiguo, búscalo en HubSpot
   (`search_crm_objects`) antes de preguntar; solo pregunta si hay más de un
   match razonable.
2. **Qué mail toca.** Si el usuario no lo dice explícitamente, calcúlalo desde
   `createdate` (fecha de creación del ticket en HubSpot — no uses la
   propiedad custom `fecha_inicio_ob` para esto, es menos confiable porque
   depende de carga manual):
   - Sin reunión de bienvenida registrada todavía → Mail 1.
   - Reunión ya hecha → semana correspondiente según los días transcurridos
     desde `createdate` (14/28/42 ± unos días de margen). Si cae justo
     entre dos, pregunta cuál corresponde en vez de asumir.

## Paso 1 — Recolectar datos

Sigue la tabla de fuentes de datos del mail correspondiente (están en los
archivos de referencia). Reglas generales:

- **HubSpot es la fuente de verdad** para datos de cliente/contacto/fechas,
  igual que en el resto de los reportes de Clay.
- Todos los números de avance (movimientos, matches, asientos, DTEs, tarjetas)
  salen de `clay_empresas_avance` — no los calcules a mano ni los inventes.
- El health score y adopción por módulo salen de `clay-dw:product-health`.
- **Si `clay-dw:product-health` no devuelve datos** (cliente nuevo, sin
  actividad en LogRocket), no lo dejes en blanco ni inventes números: usa el
  texto de fallback exacto que está en `references/mails_seguimiento.md`.
- El comentario libre del onboarder es manual — pregúntaselo al usuario antes
  de generar el borrador final si no te lo dio ya. No lo redactes tú en su
  nombre salvo que te lo pida explícitamente.

## Paso 2 — Armar el contenido

Cada archivo de referencia trae la estructura exacta de tablas, el tono
esperado (ejemplo de saludo/diagnóstico) y las variables `{{...}}` a
reemplazar. Reemplaza siempre todas las variables con datos reales — nunca
dejes un `{{placeholder}}` sin resolver en el borrador final. Si falta un dato
puntual, dilo explícitamente en el cuerpo ("dato no disponible") en vez de
omitir la fila silenciosamente, para que el onboarder lo note.

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
- Asunto y cuerpo según la plantilla del mail correspondiente
- Firma del onboarder (nombre + cargo; el link de Calendly es el que el
  onboarder haya indicado — ver decisión pendiente #3 en
  `references/decisiones_pendientes.md`)

Confirma al usuario que el borrador quedó listo y resume qué datos se
completaron con éxito y cuáles usaron un fallback o quedaron marcados como
"no disponible", para que sepa qué revisar antes de enviar.

## Cuando algo no calza

Esta spec todavía tiene puntos abiertos que Josefa y OPS no han cerrado
(autenticación MCP, cómo se ingresa el comentario del onboarder, Calendly por
persona vs. general, envío a múltiples usuarios, ejecución manual fuera de
fecha). Si el caso concreto que te piden depende de una de esas decisiones,
dilo explícitamente en vez de asumir en silencio — usa el default documentado
en `references/decisiones_pendientes.md` pero menciona que es un supuesto.
