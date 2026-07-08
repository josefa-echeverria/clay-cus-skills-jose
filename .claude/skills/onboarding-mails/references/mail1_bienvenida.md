# Mail 1 — Minuta de Reunión de Bienvenida

Se envía (como borrador) dentro de las 24 hs posteriores a la primera reunión
con el cliente.

## Propósito

No es un resumen cualquiera: es el primer documento operativo que el cliente
recibe. Tiene que quedar clarísimo qué se hizo en la reunión, qué falta, y
cuál es el primer paso concreto que el cliente tiene que dar. Si el borrador
queda ambiguo en el "y ahora qué hago", no cumplió su función.

## Fuentes de datos

| Dato | Fuente | Herramienta MCP |
|---|---|---|
| Nombre empresa y contacto principal | HubSpot | `search_crm_objects` |
| Resumen de la reunión (notas Diio) | Diio | `summarize_client_interactions_content` |
| Bancos declarados/conectados | Diio + Clay MCP | `clay_conexiones_list` |
| SII conectado | Clay MCP | `clay_conexiones_list` |
| Tarjeta de crédito configurada | Clay MCP | `clay_empresas_avance` |
| Facturador y XML | Diio | `get_deal_details` |
| Comentario libre del onboarder | Manual | Se lo pides al onboarder antes de enviar |

## Diagnóstico inicial

Genera un párrafo introductorio breve con el diagnóstico de la reunión. Tono
esperado (ejemplo real de la spec, adapta el contenido pero mantén el tono
cercano y directo, sin relleno corporativo):

> "Hola {{nombre_contacto}}, gracias por la reunión de hoy. Aquí va el resumen
> de lo que vimos y los primeros pasos para que {{nombre_empresa}} arranque
> con buen pie en Clay."

## Tabla de próximos pasos

En el Mail 1, todos los ítems van marcados como Pendiente (`⏳`) o ya resuelto
(`✅ Ok`) según lo detectado en la reunión/conexiones — no hay estado
"En progreso" todavía, porque el cliente recién empieza.

| Área | Tarea |
|---|---|
| Ajustes Generales | Crear usuarios y permisos |
| Ajustes Generales | Conectar cuentas bancarias |
| Ajustes Generales | Conectar SII |
| Ajustes Generales | Activar conciliación automática |
| Contabilidad | Configurar plan de cuentas |
| Contabilidad | Categorizar clientes y proveedores |
| Contabilidad | Cargar asiento de apertura |
| Gestión Bancaria | Revisar movimientos del último mes |
| Gestión Bancaria | Realizar primera conciliación bancaria |
| Obligaciones | Revisar documentos por pagar |
| Obligaciones | Revisar documentos por cobrar |
| Gestión del Negocio | Explorar panel de control y flujo de caja |

Marca `✅ Ok` solo lo que confirmaste con datos reales (p. ej. si
`clay_conexiones_list` ya muestra un banco o el SII conectado). No asumas que
algo está listo solo porque se mencionó en la reunión.

## Adjunto

Adjunta automáticamente `onboarding_clay.pptx` al crear el draft en Gmail. Es
obligatorio en este mail — si falla, avisa en vez de omitirlo silenciosamente.

## Estructura del mail

| Campo | Contenido |
|---|---|
| Asunto | Minuta reunión de bienvenida — {{nombre_empresa}} |
| Para | {{email_contacto_principal}} |
| CC | ob@clay.cl |
| Saludo | Hola {{nombre_contacto}}, |
| Cuerpo | Diagnóstico inicial + tabla de conexiones detectadas + tabla de próximos pasos |
| Cierre | "Cualquier duda me avisas. ¡Nos vemos en la próxima reunión!" |
| Firma | Firma del onboarder (nombre + cargo + Calendly) |
| Adjunto | onboarding_clay.pptx |
