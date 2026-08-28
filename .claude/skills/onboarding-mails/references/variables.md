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

## Estructura de grupo — HubSpot (companies)

Antes de armar cualquier mail, revisar estas propiedades de la empresa que
dispara el envío:

| Variable | Columna en HubSpot (companies) |
|---|---|
| `{{rut_empresa_madre}}` | `rut_empresa_madre` — si tiene valor, esta empresa es hija |
| `{{rut_empresas_hijas}}` | `rut_empresas_hijas` — si tiene valor, esta empresa es madre; puede traer varios RUT |
| `{{parent_company_id}}` | `hs_parent_company_id` (asociación nativa) — respaldo/cruce, no reemplaza a los dos anteriores |

A partir de estos campos se deriva, para el resumen agregado (ver
`references/mails_seguimiento.md`, sección 3):

| Variable | Cómo se obtiene |
|---|---|
| `{{empresas_del_grupo}}` | Lista de RUT: la madre + todas las hijas de `rut_empresas_hijas` |
| `{{nombre_empresa}}` (por fila del agregado) | Cruce de cada RUT del grupo contra `nombre_empresa` en la card 6206 |
| `{{pct_avance}}` (por fila del agregado) | `pct_avance` de la card 6206 para ese RUT — `—` si no aparece |
| `{{tareas_ok}}` / `{{tareas_pendientes}}` (por fila del agregado) | `tareas_ok` / `tareas_pendientes` de la card 6206 para ese RUT — `—` si no aparece |
| `{{pct_match_cassius}}` (por fila del agregado) | `% match cassius` de la card 6206 para ese RUT — celda vacía (no `—`) si viene `null` |
| `{{pct_asientos_cassius}}` (por fila del agregado) | `% asientos cassius` de la card 6206 para ese RUT — celda vacía (no `—`) si viene `null` |
| `{{tareas_pendientes_lista}}` | Nombres de tarea (sin área) con `estado = Pendiente` en la card 6207 para ese RUT, unidos con `" · "` |

Nota: estas dos últimas (`pct_match_cassius`, `pct_asientos_cassius`) son las
mismas columnas que usa el bloque "Automatización con Cassius" (sección 2 de
`mails_seguimiento.md`), solo que acá se leen una vez por cada empresa del
grupo en vez de solo para la que dispara el mail — así el resumen agregado
muestra el nivel de automatización de la hija, no solo su % de avance.

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
