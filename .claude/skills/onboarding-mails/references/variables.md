# Diccionario de variables

## Empresa y contacto

| Variable | Descripción / Fuente |
|---|---|
| `{{nombre_empresa}}` | Nombre legal — HubSpot / DW `organizations` |
| `{{rut_empresa}}` | RUT formateado — dashboard Metabase `staging_marts.organizations_onboarding_progress` |
| `{{nombre_contacto}}` | Nombre del contacto principal — HubSpot contacts |
| `{{email_contacto}}` / `{{email_contacto_principal}}` | Email del usuario — HubSpot contacts |
| `{{fecha_inicio_ob}}` | Fecha de inicio del onboarding, usada para calcular la semana — se toma de `createdate` (fecha de creación del ticket en HubSpot), no de la propiedad custom `fecha_inicio_ob`. Esta sigue siendo la fuente para decidir "qué mail toca" (ver Paso 0 del SKILL.md principal) |
| `{{nombre_onboarder}}` | Propietario del ticket en HubSpot |
| `{{calendly_onboarder}}` | Link Calendly del onboarder (hardcodeado por usuario — ver decisión pendiente #3) |

> Nota: el dashboard también trae `onboarder_asignado` y `semana_onboarding` propios
> (ver más abajo). Son útiles como dato de contexto/cruce, pero **no reemplazan**
> a `nombre_onboarder` (HubSpot) ni al cálculo de semana por `createdate` — ese
> sigue siendo el criterio oficial para decidir qué mail enviar.

## Avance — fuente principal: dashboard Metabase

**Fuente principal (dashboard "Onboarding - Progreso y Checklist", dashboard_id 607,
tabla `staging_marts.organizations_onboarding_progress`).** Se consulta vía
Metabase Clay MCP (`execute_query`, `database_id: 40`) filtrando por
`nombre_empresa`. Ver la query exacta en `references/queries_dashboard.md`.

**Fallback:** si la query no devuelve fila para esa empresa (no está marcada
`es_onboarding = true` en el dashboard, o aún no fue cargada), usar
`clay_empresas_avance` (Clay MCP) para lo que se pueda mapear, y dejar
explícito en el borrador que ese dato vino del fallback y no del dashboard
(para que quede claro si algún número no calza con lo que el onboarder ve en
el panel).

| Variable | Campo dashboard (principal) | Campo Clay MCP (fallback) |
|---|---|---|
| `{{pct_avance}}` | `pct_avance` | No tiene equivalente directo — si se usa fallback, dejar "dato no disponible" |
| `{{tareas_ok}}` | `tareas_ok` | No tiene equivalente directo |
| `{{tareas_pendientes}}` | `tareas_pendientes` | No tiene equivalente directo |
| `{{total_movimientos}}` | `movimientos_totales` | `clay_empresas_avance` → `total_movements` |
| `{{movimientos_tarjeta}}` | `movimientos_tarjeta` | `clay_empresas_avance` → `creditcard` (aproximado) |
| `{{movimientos_sin_match}}` | `movimientos_sin_match` | `clay_empresas_avance` → `unmatched_movements` |
| `{{pct_match}}` | Calculado: `(movimientos_totales - movimientos_sin_match) / movimientos_totales × 100`, redondeado a 1 decimal | Mismo cálculo con los campos equivalentes de `clay_empresas_avance` |
| `{{asientos_total}}` | `asientos_contables_totales` | `clay_empresas_avance` → `entries_by_user` (total) |
| `{{asientos_cassius}}` | `asientos_por_cassius` | `clay_empresas_avance` → `entries_by_user` (Cassius) |
| `{{asientos_manual}}` | `asientos_manuales` | `clay_empresas_avance` → `entries_by_user` (manual) |
| `{{dtes_cobrar}}` | `dtes_por_cobrar_sin_contabilizar` | `clay_empresas_avance` → `dte` (por cobrar) |
| `{{dtes_pagar}}` | `dtes_por_pagar_sin_contabilizar` | `clay_empresas_avance` → `dte` (por pagar) |
| `{{tc_sin_match}}` | `tc_medios_pago_sin_match` | `clay_empresas_avance` → `creditcard` |

`{{pct_match}}` reemplaza en el mail a los campos crudos `pct_conciliacion_cassius_auto`
y `pct_conciliacion_usuario` — siguen existiendo en el dashboard y se pueden
seguir consultando si alguien pide el desglose específico, pero ya no se
muestran por defecto en la tabla de avance del mail (ver decisión #10c en
`decisiones_pendientes.md`).

Las filas `{{asientos_total}}`, `{{asientos_cassius}}`, `{{asientos_manual}}`,
`{{dtes_cobrar}}` y `{{dtes_pagar}}` **se omiten por completo** en la tabla de
avance cuando `{{product_category}}` es `Software Gestión Financiera` o
`API Bancaria & SII` — ver decisión #11.

| Variable | Fuente |
|---|---|
| `{{product_category}}` | `staging.organizations` → `product_category`, filtrado por `real_name = {{nombre_empresa}}` (query 6 de `queries_dashboard.md`) |

## Checklist de próximos pasos — fuente principal: dashboard Metabase

Reemplaza el cálculo manual anterior. Fuente:
`staging_marts.organizations_checklist_status`, filtrado por `nombre_empresa`
(join con `organizations_onboarding_progress`). Devuelve filas reales de
`area`, `tarea`, `estado`, `fecha_completado`.

**Importante — cambio de estados:** esta tabla solo distingue dos estados,
`Ok` y `Pendiente` (no existe un tercer estado tipo "En progreso" en esta
fuente). Se mapea:

- `Ok` → `✅ Ok`
- `Pendiente` → `⏳ Pendiente`

El estado `🔄 En progreso` que usaban los Mails 2-4 antes de este cambio se
elimina — decisión confirmada (no es un default a validar, ver decisión #7
en `decisiones_pendientes.md`). No lo reconstruyas cruzando otra fuente de
datos ni lo infieras a partir de valores parciales de `clay_empresas_avance`.

Si la query no devuelve ninguna fila para la empresa, no se debe asumir "todo
pendiente" — hay que dejarlo explícito como "checklist no disponible en el
dashboard" y avisar, porque puede significar que la empresa no está marcada
`es_onboarding = true` todavía.

| Variable | Fuente |
|---|---|
| `{{checklist_filas}}` | `organizations_checklist_status` → filas `area`/`tarea`/`estado` para `{{nombre_empresa}}` |

## Empresas hijas (grupo empresarial)

Solo aplica a Mails 2, 3 y 4 (seguimiento) — Mail 1 no las incluye por
defecto (ver decisión pendiente #8). Fuente:
`staging_marts.organizations_checklist_grupo`, filtrando `nombre_grupo =
{{nombre_empresa}}` y `rol = 'Hija'`.

Antes de armar el mail, consulta si existen hijas (ver query en
`references/queries_dashboard.md`). Si no hay filas, la empresa no tiene
grupo y se omite toda esta sección sin mencionarla — no hace falta decir "no
tiene hijas".

Si sí hay hijas, la decisión tomada es: **una tabla de checklist completa por
cada hija**, igual formato que la tabla de la madre, cada una bajo su propio
subtítulo con el nombre de la hija. No se resume en una sola fila por hija.

| Variable | Fuente |
|---|---|
| `{{n_hijas}}` | Conteo de empresas con `rol = 'Hija'` para ese `nombre_grupo` |
| `{{nombre_hija}}` | Nombre de cada empresa hija (una tabla por cada una) |
| `{{checklist_filas_hija}}` | `organizations_checklist_grupo` → filas `area`/`tarea`/`estado` para esa hija específica |
| `{{pct_avance_hijas}}` | % de avance agregado de todas las hijas (contexto, opcional en el texto introductorio del bloque) |

## Product health

| Variable | Tabla DW |
|---|---|
| `{{health_pct}}` | `staging.health_user_daily` → `health_pct` |
| `{{health_category}}` | `staging.health_user_daily` → `health_category` |
| `{{adopt_component}}` | `staging.health_user_daily` → `adopt_component` |
| `{{usage_component}}` | `staging.health_user_daily` → `usage_component` |
| `{{modulos_criticos}}` | `staging.adoption_scores_daily` → top 3 `adopt_pct` más bajo |
| `{{sugerencias_uso}}` | Generado por Claude a partir de `adoption_scores_daily` |

Esta sección no cambia — el dashboard de onboarding no cubre adopción de
producto, así que `clay-dw:product-health` sigue siendo la fuente única para
health score y sugerencias de uso. **Excepción:** si `{{product_category}}`
es `Software Gestión Financiera` o `API Bancaria & SII`, no se consulta esta
sección — los módulos de menor adopción en esos casos son justamente los
contables, que la empresa no usa (ver decisión #11).

## Manual

| Variable | Origen |
|---|---|
| `{{comentario_onboarder}}` | El onboarder lo agrega antes de generar el borrador final — pregúntaselo si no te lo dio, no lo inventes. |

Regla general: si una variable no tiene dato disponible (ni en el dashboard
ni en el fallback), no la dejes como `{{placeholder}}` crudo en el texto
final ni la borres en silencio — reemplázala por algo explícito como "dato no
disponible" para que el onboarder lo note al revisar el borrador.
