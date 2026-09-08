# Diccionario de variables

## Empresa y contacto

| Variable | Descripción / Fuente |
|---|---|
| `{{nombre_empresa}}` | Nombre legal — HubSpot, cruzado (case-insensitive) con `nombre_empresa` del dashboard 607 |
| `{{rut_empresa}}` | RUT — dashboard 607 (cards 6206/6207) o HubSpot |
| `{{nombre_contacto}}` | Nombre del contacto principal — HubSpot contacts |
| `{{email_contacto}}` / `{{email_contacto_principal}}` | Email del usuario — HubSpot contacts |
| `{{createdate}}` | Fecha de creación del ticket — HubSpot ticket (usada para calcular la semana) |
| `{{nombre_onboarder}}` | Propietario del ticket en HubSpot (cruzar contra `onboarder_asignado` del dashboard si hace falta confirmar) |
| `{{calendly_onboarder}}` | Link Calendly del onboarder (lo indica el onboarder — ver decisión pendiente) |

## Estructura de grupo — Dashboard 607, pestaña "Próximos Pasos"

**✅ Fuente vigente (septiembre 2026):** ya no se detecta con propiedades de
HubSpot — se detecta y se calcula todo desde la tabla
`staging_marts.organizations_checklist_grupo`, vía los cards 6230-6234 y
6301 del dashboard 607, pestaña "Próximos Pasos". Ver el procedimiento
completo (cómo detectar madre/hija/independiente) en
`references/mails_seguimiento.md`, sección "0. Detectar estructura de
grupo", y el historial de por qué se dejó de usar HubSpot en
`references/decisiones_pendientes.md` (decisión #6).

| Variable | Columna / card de origen |
|---|---|
| `{{nombre_grupo}}` | Columna `nombre_grupo` de la card 6230 — nombre de la empresa madre del grupo |
| `{{rol}}` | Columna `rol` de la card 6230 — `Madre` o `Hija` |

A partir de esto se deriva, para el resumen agregado (ver
`references/mails_seguimiento.md`, sección 3):

| Variable | Cómo se obtiene |
|---|---|
| `{{pct_avance_hijas}}` | Card 6231, columna `pct_avance_hijas`, filtrado por `nombre_grupo` = madre del grupo |
| `{{empresas_hijas}}` | Card 6232, columna `empresas_hijas` |
| `{{hijas_100}}` | Card 6233, columna `hijas_100` |
| `{{tareas_pendientes_hijas}}` | Card 6234, columna `tareas_pendientes_hijas` |
| `{{nombre_empresa}}` (por fila de hija) | Columna `nombre_empresa` de la card 6230 con `rol = 'Hija'` para ese `nombre_grupo` |
| `{{tareas_pendientes_lista}}` | Nombres de tarea (sin área) con `estado = Pendiente` en la card 6230 para esa hija, unidos con `" · "` |
| `{{porcentaje_match_cassius}}` / `{{porcentaje_asientos_cassius}}` (por fila de hija) | Card 6301, columnas `porcentaje_match_cassius` / `porcentaje_asientos_cassius`, para esa hija |

El MCP de Metabase no soporta pasar el parámetro de filtro de estos cards —
hay que ejecutar `execute_card` sin filtro (trae todas las filas de todos
los grupos) y filtrar localmente por `nombre_grupo`/`nombre_empresa`
(comparación case-insensitive), igual que con las cards 6206/6207.

## Avance — Dashboard 607, card 6206 (`execute_card`, dashboard_id 607, card_id 6206)

**No se usa `clay_empresas_avance` (Clay MCP) ni `sources.*` directamente.**
Filtrar el resultado de la card por `nombre_empresa` (case-insensitive).

**Columnas confirmadas directo en la card (verificado en Metabase el 7 de
agosto 2026 — Piero actualizó la card el 6 de agosto).** Varias tienen
espacios en el nombre; van tal cual las devuelve `execute_card`, no las
renombres:

| Variable | Columna en card 6206 |
|---|---|
| `{{pct_avance}}` | `pct_avance` |
| `{{tareas_ok}}` | `tareas_ok` |
| `{{tareas_pendientes}}` | `tareas_pendientes` |
| `{{movimientos_totales}}` | `movimientos_totales` |
| `{{movimientos_tarjeta}}` | `movimientos_tarjeta` |
| `{{match_cassius_n}}` | `match cassius (n)` |
| `{{match_usuario_n}}` | `match usuario (n)` |
| `{{movimientos_sin_match}}` | `movimientos_sin_match` |
| `{{pct_match_cassius}}` | `% match cassius` |
| `{{pct_match_usuario}}` | `% match usuario` |
| `{{tc_medios_pago_sin_match}}` | `tc_medios_pago_sin_match` |
| `{{asientos_contables_totales}}` | `asientos_contables_totales` |
| `{{asientos_por_cassius}}` | `asientos_por_cassius` |
| `{{asientos_manuales}}` | `asientos_manuales` |
| `{{pct_asientos_cassius}}` | `% asientos cassius` — **viene directo de la card**, no se calcula a mano (esto cambió el 6 de agosto 2026; antes había que calcularlo con `asientos_por_cassius / asientos_contables_totales`, ya no) |
| `{{pct_asientos_manual}}` | `% asientos manual` — mismo caso, viene directo |
| `{{dtes_por_cobrar_sin_contabilizar}}` | `dtes_por_cobrar_sin_contabilizar` |
| `{{dtes_por_pagar_sin_contabilizar}}` | `dtes_por_pagar_sin_contabilizar` |
| `{{semana_onboarding}}` | `semana_onboarding` (referencia cruzada, no reemplaza el cálculo desde `createdate`) |

Las columnas viejas `pct_conciliacion_cassius_auto` y
`pct_conciliacion_usuario` **ya no existen** — fueron reemplazadas por
`% match cassius` y `% match usuario` (más las columnas de cantidad `match
cassius (n)` / `match usuario (n)`).

## Automatización con Cassius — apartado destacado en todos los mails

Van en un apartado propio y visible en los 4 mails (bienvenida + los 3 de
seguimiento), no solo en la tabla general de avance:

| Variable | Fuente |
|---|---|
| `{{pct_match_cassius}}` | `% match cassius` (card 6206), directo |
| `{{pct_match_usuario}}` | `% match usuario` (card 6206), directo |
| `{{pct_asientos_cassius}}` | `% asientos cassius` (card 6206), directo |
| `{{pct_asientos_manual}}` | `% asientos manual` (card 6206), directo |

Si alguno viene `null` (pasa cuando `asientos_contables_totales = 0`, común
en el Mail 1), dejar la celda vacía en la tabla del mail — ver la regla
completa en `references/mails_seguimiento.md`, sección 2.

## Checklist / próximos pasos — Dashboard 607, card 6207 (`execute_card`, dashboard_id 607, card_id 6207)

| Variable | Columna en card 6207 |
|---|---|
| `{{area}}` | `area` |
| `{{tarea}}` | `tarea` |
| `{{estado}}` | `estado` (`Ok`/`Pendiente` → traducir a `✅ Ok` / `⏳ Pendiente`) |
| `{{fecha_completado}}` | `fecha_completado` (puede ser `null` → mostrar `—`) |

## Health / product health (sin cambios)

| Variable | Fuente |
|---|---|
| `{{health_pct}}` / `{{health_category}}` | `clay-dw:product-health` |
| `{{modulos_criticos}}` | `clay-dw:product-health` — top 3 `adopt_pct` más bajo |
| `{{sugerencias_uso}}` | Generado por Claude a partir de lo anterior |

## Manual

| Variable | Origen |
|---|---|
| `{{comentario_onboarder}}` | El onboarder lo agrega antes de generar el borrador final — pregúntaselo si no te lo dio, no lo inventes. |

Regla general: si una variable no tiene dato disponible (la empresa no
aparece en ninguna de las dos cards), no la dejes como `{{placeholder}}`
crudo en el texto final — reemplázala por algo explícito como "dato no
disponible" para que el onboarder lo note al revisar el borrador. Lo mismo
aplica a las filas del resumen agregado del grupo.
