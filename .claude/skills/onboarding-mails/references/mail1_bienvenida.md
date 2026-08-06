# Mail 1 — Minuta de Reunión de Bienvenida

Se envía (como borrador) dentro de las 24 hs posteriores a la primera reunión
con el cliente.

## Propósito

No es un resumen cualquiera: es el primer documento operativo que el cliente
recibe. Tiene que quedar clarísimo qué se hizo en la reunión, qué falta, y
cuál es el primer paso concreto que el cliente tiene que dar.

## Fuentes de datos

| Dato | Fuente | Cómo buscarlo |
|---|---|---|
| Nombre empresa y contacto principal | HubSpot | `search_crm_objects` |
| Estructura de grupo (madre/hijas) | HubSpot | `rut_empresa_madre`, `rut_empresas_hijas` (companies) — ver detalle abajo |
| Resumen de la reunión (notas Diio) | Diio | `summarize_client_interactions_content` |
| Estado de checklist (bancos, SII, usuarios, etc.) | Dashboard 607, card **6207** | `execute_card` (dashboard_id 607, card_id 6207), filtrar por empresa — ver detalle abajo |
| Avance y automatización Cassius | Dashboard 607, card **6206** | `execute_card` (dashboard_id 607, card_id 6206), filtrar por empresa — ver detalle abajo |
| Facturador y XML | Diio | `get_deal_details` |
| Comentario libre del onboarder | Manual | Se lo pides al onboarder antes de enviar |

**Cambio importante:** el estado de conexiones (bancos, SII, usuarios) ya no
se consulta vía Clay MCP ni tablas propias — sale de la misma card 6207 que
usan los mails de seguimiento (`references/mails_seguimiento.md`), porque
"Conectar cuentas bancarias" y "Conectar SII" son tareas del mismo checklist
que trae esa card. No dupliques lógica: es la misma fuente para las 4 tareas
del área "Ajustes Generales" (Activar conciliación automática, Conectar
cuentas bancarias, Conectar SII, Crear usuarios y permisos).

Si la empresa es tan nueva que todavía no aparece en el dashboard 607 (recién
salió de la reunión de bienvenida, antes de la primera sincronización), es
válido no tener estado todavía — decilo explícitamente en el mail en vez de
mostrar `⏳ Pendiente` en todo, que insinúa que ya se revisó. Lo mismo aplica
al bloque de Cassius más abajo: si no hay fila en la card 6206, no inventes
porcentajes.

## Estructura de grupo (empresa madre / hijas)

Antes de armar el mail, revisá en HubSpot (companies) las mismas propiedades
que usan los mails de seguimiento:

- **`rut_empresa_madre`** — si tiene valor, esta empresa es una **hija**.
- **`rut_empresas_hijas`** — si tiene valor, esta empresa es una **madre**
  con una o más hijas.
- **`hs_parent_company_id`** — respaldo/cruce si los dos campos de RUT
  faltan o no coinciden.

Si hay estructura de grupo, sigue exactamente el mismo procedimiento y
formato de tabla que la sección "0. Detectar estructura de grupo" y
"3. Resumen agregado del grupo" de `references/mails_seguimiento.md` — no
dupliques la lógica acá, solo aplicala. La única diferencia en el Mail 1 es
que es más probable que alguna empresa del grupo (o la propia empresa que
dispara el mail) todavía no tenga fila en la card 6206/6207 por ser recién
incorporada — en ese caso, usá `—` / "Sin dato en el dashboard" en vez de
omitir la fila, igual que en seguimiento.

## Tabla de próximos pasos

Usa las 11 tareas reales del checklist (área + tarea), tal como vienen de la
card 6207 — no la lista genérica de versiones anteriores de esta skill:

| Área | Tarea |
|---|---|
| Ajustes Generales | Activar conciliación automática |
| Ajustes Generales | Conectar cuentas bancarias |
| Ajustes Generales | Conectar SII |
| Ajustes Generales | Crear usuarios y permisos |
| Contabilidad | Cargar asiento de apertura |
| Contabilidad | Categorizar clientes y proveedores |
| Gestión Bancaria | Realizar primera conciliación bancaria |
| Gestión Bancaria | Revisar movimientos del último mes |
| Gestión del Negocio | Explorar flujo de caja |
| Gestión del Negocio | Explorar panel de control |
| Obligaciones | Crear Asiento contable Manual |

Si la empresa ya aparece en el dashboard, usa el `estado` real de cada fila
(`Ok` → `✅ Ok`, `Pendiente` → `⏳ Pendiente`) en vez de asumir. Si todavía no
aparece (ver nota arriba), usa esta lista solo como agenda de lo que falta,
sin columna de estado.

## Automatización con Cassius (apartado destacado)

Igual que en los mails de seguimiento, este bloque va siempre en un apartado
propio y visible, aunque en el Mail 1 sea más probable que todavía no haya
datos (recién arrancó el onboarding):

| Métrica | Valor |
|---|---|
| % de match (conciliación) hechos por Cassius | `{{pct_match_cassius}}%` |
| % de asientos contables hechos por Cassius | `{{pct_asientos_cassius}}%` |

- `{{pct_match_cassius}}` = `pct_conciliacion_cassius_auto` de la card 6206.
- `{{pct_asientos_cassius}}` = calculado como
  `asientos_por_cassius / asientos_contables_totales * 100`, redondeado a 1
  decimal (no viene directo de la card). Si `asientos_contables_totales` es
  0, mostrá "sin asientos registrados aún".

Si la empresa no tiene fila todavía en la card 6206 (lo más común recién
después de la primera reunión), mostrá el bloque completo con el texto
"Aún no hay datos de automatización disponibles — es normal recién
empezando" en vez de dejarlo vacío o inventar números.

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
| Cuerpo | Diagnóstico inicial + tabla de próximos pasos (con o sin estado, según disponibilidad) |
| Bloque Cassius | Automatización con Cassius (apartado destacado) |
| Bloque grupo | Resumen agregado del grupo — solo si aplica (madre/hijas) |
| Cierre | "Cualquier duda me avisas. ¡Nos vemos en la próxima reunión!" |
| Firma | Firma del onboarder (nombre + cargo + Calendly) |
| Adjunto | onboarding_clay.pptx |
