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

**Nuevo (agosto 2026).** Antes de armar cualquier mail, revisar estas
propiedades de la empresa que dispara el envío:

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
| `{{tareas_pendientes_lista}}` | Nombres de tarea (sin área) con `estado = Pendiente` en la card 6207 para ese RUT, unidos con `" · "` |

## Avance — Dashboard 607, card 6206 (`execute_card`, dashboard_id 607, card_id 6206)

**No se usa `clay_empresas_avance` (Clay MCP) ni `sources.*` directamente.**
Filtrar el resultado de la card por `nombre_empresa` (case-insensitive).

| Variable | Columna en card 6206 |
|---|---|
| `{{pct_avance}}` | `pct_avance` |
| `{{tareas_ok}}` | `tareas_ok` |
| `{{tareas_pendientes}}` | `tareas_pendientes` |
| `{{movimientos_totales}}` | `movimientos_totales` |
| `{{movimientos_tarjeta}}` | `movimientos_tarjeta` |
| `{{movimientos_sin_match}}` | `movimientos_sin_match` |
| `{{pct_conciliacion_cassius_auto}}` | `pct_conciliacion_cassius_auto` |
| `{{pct_conciliacion_usuario}}` | `pct_conciliacion_usuario` |
| `{{tc_medios_pago_sin_match}}` | `tc_medios_pago_sin_match` |
| `{{asientos_contables_totales}}` | `asientos_contables_totales` |
| `{{asientos_por_cassius}}` | `asientos_por_cassius` |
| `{{asientos_manuales}}` | `asientos_manuales` |
| `{{dtes_por_cobrar_sin_contabilizar}}` | `dtes_por_cobrar_sin_contabilizar` |
| `{{dtes_por_pagar_sin_contabilizar}}` | `dtes_por_pagar_sin_contabilizar` |
| `{{semana_onboarding}}` | `semana_onboarding` (referencia cruzada, no reemplaza el cálculo desde `createdate`) |

## Automatización con Cassius — calculadas (apartado destacado en todos los mails)

**Nuevo (agosto 2026).** Estas dos variables van en un apartado propio y
visible en los 4 mails (bienvenida + los 3 de seguimiento), no solo en la
tabla general de avance:

| Variable | Cómo se obtiene |
|---|---|
| `{{pct_match_cassius}}` | = `pct_conciliacion_cassius_auto` de la card 6206, tal cual (alias más claro para el apartado destacado) |
| `{{pct_asientos_cassius}}` | **Calculado, no viene directo de la card:** `asientos_por_cassius / asientos_contables_totales * 100`, redondeado a 1 decimal. Si `asientos_contables_totales = 0`, usar el texto "sin asientos registrados aún" en vez de dividir por cero |

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
aplica a `{{pct_match_cassius}}`, `{{pct_asientos_cassius}}` y a las filas
del resumen agregado del grupo.
