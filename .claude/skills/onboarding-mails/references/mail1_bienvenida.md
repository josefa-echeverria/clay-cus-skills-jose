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

**Checklist de próximos pasos:** intenta primero la query 2 de
`references/queries_dashboard.md` (dashboard Metabase "Onboarding - Progreso
y Checklist"). Como el Mail 1 se envía justo después de la primera reunión,
es normal que el dashboard todavía no tenga filas para esta empresa — en ese
caso no es un error, simplemente sigue con el método anterior (Diio +
`clay_conexiones_list` + `clay_empresas_avance`) descrito abajo. Si el
dashboard sí devuelve filas, úsalas en vez de inferir manualmente.

No se agrega checklist de empresas hijas en el Mail 1 (a diferencia de los
Mails 2-4) — a esta altura del onboarding es poco probable que ya haya datos
de grupo, y este mail busca simplicidad para la primera impresión. Ver
decisión pendiente #8.

## Diagnóstico inicial

Genera un párrafo introductorio breve con el diagnóstico de la reunión. Tono
esperado (ejemplo real de la spec, adapta el contenido pero mantén el tono
cercano y directo, sin relleno corporativo):

> "Hola {{nombre_contacto}}, gracias por la reunión de hoy. Aquí va el resumen
> de lo que vimos y los primeros pasos para que {{nombre_empresa}} arranque
> con buen pie en Clay."

**Agrega siempre un segundo párrafo sobre el objetivo de las 8 semanas de
onboarding** — confirmado en conversación con Camila (CX Leader) al probar
esta skill. La idea central a transmitir:

> "La idea de estas 8 semanas es que puedan automatizar al máximo su gestión
> contable. Les vamos a ir mostrando el avance semana a semana, para que vean
> cómo se va reduciendo el trabajo manual a medida que Cassius (nuestro robot)
> toma más tareas."

Adapta la redacción al contexto de la reunión (por ejemplo, si en la reunión
se desactivaron automatizaciones para partir con una base limpia, menciona
eso primero y luego este párrafo), pero no omitas la idea de fondo:
**automatización progresiva + reporte semanal de avance**. Esto conecta
directamente con los Mails 2-4, que son justamente ese reporte semanal.

## Tabla de próximos pasos

En el Mail 1, todos los ítems van marcados como Pendiente (`⏳`) o ya resuelto
(`✅ Ok`) según lo detectado en la reunión/conexiones — no hay estado
"En progreso" todavía, porque el cliente recién empieza.

> La tabla de abajo está en markdown solo para que quede clara la estructura
> de columnas en esta spec. **En el draft real de Gmail va como tabla HTML**,
> nunca como markdown — ver la regla y el estilo exacto en el Paso 4 del
> `SKILL.md` principal.

| Área | Tarea |
|---|---|
| Ajustes Generales | Crear usuarios y permisos |
| Ajustes Generales | Conectar cuentas bancarias |
| Ajustes Generales | Conectar SII |
| Ajustes Generales | Activar conciliación automática |
| Contabilidad | Cargar asiento de apertura |
| Contabilidad | Categorizar clientes y proveedores |
| Gestión Bancaria | Realizar primera conciliación bancaria |
| Gestión Bancaria | Revisar movimientos del último mes |
| Gestión del Negocio | Explorar flujo de caja |
| Gestión del Negocio | Explorar panel de control |
| Obligaciones | Crear asiento contable manual |

> Esta lista de 11 tareas en 5 áreas es la misma que trae el dashboard
> (`organizations_checklist_status`) — se corrigió para que coincida
> exactamente, ya que la spec original tenía una lista de 12 tareas distinta
> (combinaba "panel de control y flujo de caja" en una sola fila, y usaba
> "Revisar documentos por pagar/por cobrar" en vez de "Crear asiento contable
> manual"). Si el dashboard todavía no tiene filas para esta empresa (caso
> normal en el Mail 1, ver más arriba), usa esta misma lista como plantilla
> para armar la tabla a mano con lo que confirmes vía Diio/Clay MCP.

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
| Cierre | "Cualquier duda me avisas.
