# Diccionario de variables

## Empresa y contacto

| Variable | Descripción / Fuente |
|---|---|
| `{{nombre_empresa}}` | Nombre legal — HubSpot / DW `organizations` |
| `{{rut_empresa}}` | RUT formateado — DW `staging.organizations` |
| `{{nombre_contacto}}` | Nombre del contacto principal — HubSpot contacts |
| `{{email_contacto}}` / `{{email_contacto_principal}}` | Email del usuario — HubSpot contacts |
| `{{fecha_inicio_ob}}` | Fecha de inicio del onboarding, usada para calcular la semana — se toma de `createdate` (fecha de creación del ticket en HubSpot), no de la propiedad custom `fecha_inicio_ob` |
| `{{nombre_onboarder}}` | Propietario del ticket en HubSpot |
| `{{calendly_onboarder}}` | Link Calendly del onboarder (hardcodeado por usuario — ver decisión pendiente #3) |

## Avance (`clay_empresas_avance`)

| Variable | Campo MCP/DW |
|---|---|
| `{{total_movimientos}}` | `clay_empresas_avance` → `total_movements` |
| `{{movimientos_sin_match}}` | `clay_empresas_avance` → `unmatched_movements` |
| `{{pct_cassius}}` | `clay_empresas_avance` → `matches_by_user` (Cassius) |
| `{{pct_usuario}}` | `clay_empresas_avance` → `matches_by_user` (humano) |
| `{{asientos_total}}` | `clay_empresas_avance` → `entries_by_user` (total) |
| `{{asientos_cassius}}` | `clay_empresas_avance` → `entries_by_user` (Cassius) |
| `{{asientos_manual}}` | `clay_empresas_avance` → `entries_by_user` (manual) |
| `{{dtes_cobrar}}` | `clay_empresas_avance` → `dte` (por cobrar) |
| `{{dtes_pagar}}` | `clay_empresas_avance` → `dte` (por pagar) |
| `{{tc_sin_match}}` | `clay_empresas_avance` → `creditcard` |

## Product health

| Variable | Tabla DW |
|---|---|
| `{{health_pct}}` | `staging.health_user_daily` → `health_pct` |
| `{{health_category}}` | `staging.health_user_daily` → `health_category` |
| `{{adopt_component}}` | `staging.health_user_daily` → `adopt_component` |
| `{{usage_component}}` | `staging.health_user_daily` → `usage_component` |
| `{{modulos_criticos}}` | `staging.adoption_scores_daily` → top 3 `adopt_pct` más bajo |
| `{{sugerencias_uso}}` | Generado por Claude a partir de `adoption_scores_daily` |

## Manual

| Variable | Origen |
|---|---|
| `{{comentario_onboarder}}` | El onboarder lo agrega antes de generar el borrador final — pregúntaselo si no te lo dio, no lo inventes. |

Regla general: si una variable no tiene dato disponible, no la dejes como
`{{placeholder}}` crudo en el texto final ni la borres en silencio — reemplázala
por algo explícito como "dato no disponible" para que el onboarder lo note al
revisar el borrador.
