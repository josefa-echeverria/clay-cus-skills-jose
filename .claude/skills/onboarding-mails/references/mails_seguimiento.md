# Mails 2 a 8 — Seguimiento Semana 2 a 8

Misma estructura base para las siete instancias; el contenido evoluciona con
el avance real del cliente. No copies el mail anterior — vuelve a consultar
el dashboard cada vez, porque el objetivo es mostrar avance real, no repetir
texto.

## Fuente de datos: SOLO el dashboard de Metabase (julio 2026)

**Toda la información de avance y próximos pasos viene exclusivamente del
dashboard `Onboarding - Progreso y Checklist` (id 607).** No consultes
`sources.*`, no uses `clay_empresas_avance`, no uses la skill `clay-dw`
genérica para esto — este dashboard ya integra todo con datos reales y
actualizados (`staging_marts.organizations_onboarding_progress` y
`staging_marts.organizations_checklist_status`), y Sole no tiene ni quiere
acceso directo a producción.

Usa la herramienta `execute_card` de Metabase Clay con `dashboard_id = 607`:

| Card | `card_id` | Qué trae |
|---|---|---|
| Onboarding - Avance y Conciliación por Empresa | **6206** | Una fila por empresa: `rut_empresa`, `nombre_empresa`, `semana_onboarding`, `onboarder_asignado`, `pct_avance`, `tareas_ok`, `tareas_pendientes`, `movimientos_totales`, `movimientos_tarjeta`, `match cassius (n)`, `match usuario (n)`, `movimientos_sin_match`, `% match cassius`, `% match usuario`, `tc_medios_pago_sin_match`, `asientos_contables_totales`, `asientos_por_cassius`, `asientos_manuales`, `% asientos cassius`, `% asientos manual`, `dtes_por_cobrar_sin_contabilizar`, `dtes_por_pagar_sin_contabilizar`. **Columnas con espacio en el nombre — van entre comillas/tal cual las devuelve `execute_card`.** |
| Checklist - Detalle por Empresa y Tarea | **6207** | Una fila por tarea: `rut_empresa`, `nombre_empresa`, `onboarder_asignado`, `area`, `tarea`, `estado` (`Ok`/`Pendiente`), `fecha_completado` |
| Avance y Conciliación — Empresas Hijas (Grupo) | **6301** | **Solo aplica a empresas hijas dentro de una estructura de grupo (sección 3 más abajo).** Una fila por empresa hija del grupo: `nombre_empresa`, `match_cassius_n`, `match_usuario_n`, `porcentaje_match_cassius`, `porcentaje_match_usuario`, `asientos_totales`, `asientos_cassius`, `asientos_usuario`, `porcentaje_asientos_cassius`, `porcentaje_asientos_usuario`. Vive en la colección "Onboarding" de Metabase (`analytics.clay.cl/question/6301-...`) — se agregó en septiembre 2026 (IAD-255) porque las hijas suelen no tener fila propia en la card 6206, así que su % de conciliación no se podía mostrar hasta ahora. |

**Ninguno de los dos cards acepta filtro de empresa desde `execute_card`**
— ambos devuelven todas las empresas en onboarding de una vez (~18 filas en
6206, ~200 en 6207). Filtrá el resultado localmente por `nombre_empresa`,
sin distinguir mayúsculas/minúsculas (ej. "Carotrini" debe matchear con
"CAROTRINI SPA"). Si no hay match exacto, probá comparación parcial antes de
reportar "sin dato".

Si en algún momento se agregan filtros a estos cards o cambian sus
`card_id`, actualizar esta tabla — no asumir que los IDs son estables para
siempre; volver a inspeccionar el dashboard 607 si algo no calza.

## Qué hacer en cada uno

1. Revisar en HubSpot si la empresa forma parte de una estructura de grupo
   (ver sección "0. Detectar estructura de grupo" más abajo).
2. Ejecutar `execute_card` para los cards 6206 y 6207 (dashboard 607),
   filtrar ambos resultados por la empresa (y por el resto del grupo, si
   corresponde). Si es una empresa madre con hijas, además correr la card
   6301 para obtener el % de conciliación de cada hija (ver sección 0 y
   sección 3 más abajo).
3. Consultar `clay-dw:product-health` para el health score y adopción por
   módulo (esto es independiente del dashboard 607 y sigue funcionando
   normal).
4. Armar la tabla de próximos pasos a partir de las filas de la card 6207
   para esa empresa (una fila del resultado = una fila de la tabla del
   mail).
5. Generar sugerencias personalizadas basadas en los módulos con adopción más
   baja (top 3, desde `clay-dw:product-health`).

## Fallback si no hay datos de product health

Si el cliente es muy nuevo o no tiene actividad registrada en LogRocket,
`clay-dw:product-health` puede no devolver nada. En ese caso no dejes el
bloque de sugerencias vacío ni inventes números — usa este texto tal cual:

> "Aún estamos registrando tu actividad en la plataforma. Mientras tanto,
> aquí van tus próximos pasos pendientes."

Y sáltate la tabla de sugerencias de uso (sección 5 más abajo) para ese mail.
Esto es distinto de no encontrar la empresa en el dashboard 607 — si pasa
eso, es un problema real que hay que reportar, no un fallback esperado.

## 0. Detectar estructura de grupo (empresa madre / hijas)

Antes de armar cualquier tabla, revisa en HubSpot (companies) estas
propiedades de la empresa que dispara el mail:

- **`rut_empresa_madre`** — si tiene valor, esta empresa es una **hija** de
  otra (el valor es el RUT de la madre).
- **`rut_empresas_hijas`** — si tiene valor, esta empresa es una **madre**
  con una o más hijas (puede traer varios RUT juntos; sepáralos por coma o
  punto y coma según venga el dato).
- **`hs_parent_company_id`** (asociación nativa de HubSpot) — úsalo como
  respaldo/cruce si los dos campos anteriores faltan o no coinciden entre
  sí. No lo reemplaces por completo: los campos de RUT son los que Sole usa
  para cruzar contra el dashboard 607, así que en caso de conflicto,
  prioriza tener ambos RUT y avisa del conflicto en vez de elegir uno en
  silencio.

Si ninguno de los tres campos tiene valor, la empresa es independiente:
seguí el flujo normal (secciones 1, 2 y 4 con los datos de esa única
empresa) y **saltate la sección 3** (resumen agregado del grupo).

Si detectás una estructura de grupo (madre + una o más hijas):

1. Reuní el RUT de todas las empresas del grupo: la madre + todas las hijas
   listadas en `rut_empresas_hijas` (o las que aparezcan asociadas vía
   `hs_parent_company_id` si el campo de texto está incompleto).
2. Ejecutá `execute_card` (card 6206) una sola vez para todo el grupo — ya
   trae todas las empresas de onboarding — y filtrá localmente por esos RUT
   (comparación case-insensitive, igual que con `nombre_empresa`).
3. Ejecutá también la card **6301** ("Avance y Conciliación — Empresas
   Hijas (Grupo)") para traer el % de conciliación (match y asientos) de
   cada hija — este dato normalmente **no** está en la card 6206 para las
   hijas, así que es la única fuente para completarlo. Ver "Cómo ejecutar la
   card 6301" más abajo (el tool `execute_card` no soporta pasarle
   parámetros, así que hay que correrla con `execute_query`).
4. El mail se sigue armando centrado en la empresa puntual que dispara el
   envío (madre o hija) — las secciones 1, 2 y 4 usan sus propios datos,
   como siempre. La sección 3 (resumen agregado) es la única que junta a
   **todas** las empresas del grupo.
5. Si alguna empresa del grupo no aparece todavía en la card 6206 (por
   ejemplo, recién conectada), incluila igual en la tabla agregada con `—`
   en las columnas de % Avance / tareas pendientes que falten — no la
   omitas ni la saltes en silencio. Lo mismo si no aparece en la card 6301:
   dejá vacíos los 4 porcentajes de conciliación de esa fila (misma regla de
   dato faltante que la sección 2).

### Cómo ejecutar la card 6301 (parámetro `nombre_empresa` = nombre de la madre/grupo)

La card 6301 vive en la colección "Onboarding" de Metabase
(`analytics.clay.cl/question/6301-avance-y-conciliacion-empresas-hijas-grupo`)
y tiene un parámetro obligatorio `nombre_empresa` que en realidad corresponde
al **nombre del grupo** (el nombre de la empresa madre, tal como aparece en
`staging_marts.organizations_checklist_grupo.nombre_grupo` — no confundir con
el RUT). El tool MCP `execute_card` **no acepta parámetros** para cards
nativas con template tags (falla con "Unrecognized key(s): parameters"), así
que para esta card puntual hay que usar `execute_query` (database_id 40) con
el SQL exacto de la card (podés confirmarlo con `get_card` si cambia),
reemplazando el placeholder por el nombre de la madre entre comillas simples
— duplicá cualquier comilla simple interna del nombre para no romper la
consulta:

```sql
WITH hijas AS (
  SELECT DISTINCT organization_id, nombre_empresa
  FROM staging_marts.organizations_checklist_grupo
  WHERE nombre_grupo = 'NOMBRE_DE_LA_MADRE' AND rol = 'Hija'
)
SELECT
  h.nombre_empresa,
  SUM(COALESCE(w.movimientos_conciliados_cassius,0)) AS match_cassius_n,
  SUM(COALESCE(w.movimientos_conciliados_usuario,0)) AS match_usuario_n,
  ROUND(100.0 * SUM(COALESCE(w.movimientos_conciliados_cassius,0)) / NULLIF(SUM(COALESCE(w.movimientos_conciliados_cassius,0)) + SUM(COALESCE(w.movimientos_conciliados_usuario,0)),0),1) AS porcentaje_match_cassius,
  ROUND(100.0 * SUM(COALESCE(w.movimientos_conciliados_usuario,0)) / NULLIF(SUM(COALESCE(w.movimientos_conciliados_cassius,0)) + SUM(COALESCE(w.movimientos_conciliados_usuario,0)),0),1) AS porcentaje_match_usuario,
  COUNT(ae.accounting_entry_id) AS asientos_totales,
  SUM(CASE WHEN ae.is_cassius THEN 1 ELSE 0 END) AS asientos_cassius,
  SUM(CASE WHEN NOT ae.is_cassius THEN 1 ELSE 0 END) AS asientos_usuario,
  ROUND(100.0 * SUM(CASE WHEN ae.is_cassius THEN 1 ELSE 0 END) / NULLIF(COUNT(ae.accounting_entry_id),0),1) AS porcentaje_asientos_cassius,
  ROUND(100.0 * SUM(CASE WHEN NOT ae.is_cassius THEN 1 ELSE 0 END) / NULLIF(COUNT(ae.accounting_entry_id),0),1) AS porcentaje_asientos_usuario
FROM hijas h
LEFT JOIN staging.movements_reconciliation_weekly_by_movement_date w
  ON w.organization_id = h.organization_id AND w.week_start >= '2026-01-01' AND w.week_start < '2027-01-01'
LEFT JOIN staging.accounting_entries ae
  ON ae.organization_id = h.organization_id AND ae.created_at >= '2026-01-01' AND ae.created_at < '2027-01-01'
GROUP BY h.nombre_empresa
ORDER BY h.nombre_empresa;
```

Esta excepción (usar `execute_query` en vez de `execute_card`) es solo por
la limitación del tool MCP con parámetros — sigue siendo el SQL curado de la
card 6301, no una consulta libre a `sources.*`. Si en el futuro `execute_card`
soporta parámetros, usalo directo con `card_id: 6301` en su lugar.

## 1. Tabla de avance de la empresa

Sale directo de la fila de la card 6206 para esa empresa. Compará contra la
semana anterior solo si tenés ese dato guardado de un mail previo (no lo
inventes ni lo dejes en blanco — usá `—` si no lo tenés).

**Cambio (agosto 2026): la card 6206 se modificó.** Las columnas viejas
`pct_conciliacion_cassius_auto` y `pct_conciliacion_usuario` ya no existen —
fueron reemplazadas por `% match cassius` y `% match usuario` (más las
columnas nuevas de asientos, ver sección 2). Si esta tabla no calza con lo
que devuelve `execute_card`, volvé a inspeccionar las columnas reales antes
de asumir que siguen igual.

| Métrica | Valor actual | Semana anterior |
|---|---|---|
| % Avance total del checklist | `{{pct_avance}}%` | — |
| Tareas completadas / pendientes | `{{tareas_ok}}` / `{{tareas_pendientes}}` | — |
| Movimientos totales | `{{movimientos_totales}}` | — |
| Movimientos sin match | `{{movimientos_sin_match}}` | — |
| Match Cassius (cantidad) | `{{match_cassius_n}}` | — |
| Match usuario (cantidad) | `{{match_usuario_n}}` | — |
| Movimientos de tarjeta | `{{movimientos_tarjeta}}` | — |
| TC/Medios de pago sin match | `{{tc_medios_pago_sin_match}}` | — |
| Asientos contables totales | `{{asientos_contables_totales}}` | — |
| Asientos por Cassius | `{{asientos_por_cassius}}` | — |
| Asientos manuales | `{{asientos_manuales}}` | — |
| DTEs por cobrar sin contabilizar | `{{dtes_por_cobrar_sin_contabilizar}}` | — |
| DTEs por pagar sin contabilizar | `{{dtes_por_pagar_sin_contabilizar}}` | — |

Los 4 porcentajes destacados (% match Cassius, % match usuario, % asientos
Cassius, % asientos manual) **no van en esta tabla** — van en el apartado
propio de la sección 2, que es donde el onboarder los espera ver primero.

## 2. Automatización con Cassius (apartado destacado)

Este bloque va **siempre**, en un apartado propio y visible (no mezclado
dentro de la tabla de avance de la sección 1), con los 4 porcentajes que la
card 6206 ya trae calculados — no hay que calcular nada a mano:

| Métrica | Valor |
|---|---|
| % de match hechos por Cassius | `{{pct_match_cassius}}` |
| % de match hechos por el usuario | `{{pct_match_usuario}}` |
| % de asientos contables hechos por Cassius | `{{pct_asientos_cassius}}` |
| % de asientos contables hechos manualmente | `{{pct_asientos_manual}}` |

Fuente de cada variable (columnas reales de la card 6206, ver también
`references/variables.md`):

- `{{pct_match_cassius}}` = columna `% match cassius`
- `{{pct_match_usuario}}` = columna `% match usuario`
- `{{pct_asientos_cassius}}` = columna `% asientos cassius`
- `{{pct_asientos_manual}}` = columna `% asientos manual`

**Regla para dato faltante (agosto 2026): si alguno de estos 4 viene
`null`** (pasa cuando `asientos_contables_totales = 0`, por ejemplo empresas
muy nuevas) **dejá la celda vacía en la tabla del mail**, sin guion `—` ni
texto de "sin datos disponibles" — es a propósito, para que el onboarder la
complete a mano antes de enviar. No confundas esto con que la empresa no
aparezca en la card 6206 (eso sí sigue siendo un problema real a reportar,
no un vacío esperado).

**Plantilla HTML de este bloque (no markdown, ver "Formato del cuerpo del
mail" más abajo):**

```html
<h3>Automatización con Cassius</h3>
<table style="border-collapse: collapse; width: 100%;">
  <tr>
    <th style="border: 1px solid #ddd; padding: 6px; text-align: left; background:#f5f5f5;">Métrica</th>
    <th style="border: 1px solid #ddd; padding: 6px; text-align: left; background:#f5f5f5;">Valor</th>
  </tr>
  <tr>
    <td style="border: 1px solid #ddd; padding: 6px;">% de match hechos por Cassius</td>
    <td style="border: 1px solid #ddd; padding: 6px;">{{pct_match_cassius}}</td>
  </tr>
  <tr>
    <td style="border: 1px solid #ddd; padding: 6px;">% de match hechos por el usuario</td>
    <td style="border: 1px solid #ddd; padding: 6px;">{{pct_match_usuario}}</td>
  </tr>
  <tr>
    <td style="border: 1px solid #ddd; padding: 6px;">% de asientos contables hechos por Cassius</td>
    <td style="border: 1px solid #ddd; padding: 6px;">{{pct_asientos_cassius}}</td>
  </tr>
  <tr>
    <td style="border: 1px solid #ddd; padding: 6px;">% de asientos contables hechos manualmente</td>
    <td style="border: 1px solid #ddd; padding: 6px;">{{pct_asientos_manual}}</td>
  </tr>
</table>
```

Dejá la celda de `<td>` vacía (sin texto entre las etiquetas) cuando el
valor sea `null`, tal como indica la regla de arriba.

## 3. Resumen agregado del grupo (solo si hay empresa madre/hijas)

Esta sección solo aparece si el Paso 0 detectó una estructura de grupo.
Una fila por cada empresa del grupo (incluida la que dispara el mail), con
el avance de checklist **y** el avance de conciliación de cada una — antes
esta tabla solo traía tareas pendientes; desde septiembre 2026 se agregan
también los 4 porcentajes de conciliación (mismo criterio que el apartado
"Automatización con Cassius" de la sección 2, pero por empresa del grupo):

| Empresa | % Avance | % Match Cassius | % Match Usuario | % Asientos Cassius | % Asientos Usuario | Tareas pendientes |
|---|---|---|---|---|---|---|
| `{{nombre_empresa}}` | `{{pct_avance}}%` | `{{pct_match_cassius_hija}}` | `{{pct_match_usuario_hija}}` | `{{pct_asientos_cassius_hija}}` | `{{pct_asientos_usuario_hija}}` | `{{tareas_pendientes_lista}}` |

Fuente de cada columna:

- `{{pct_avance}}` y `{{tareas_pendientes_lista}}` — igual que antes: card
  6206 (`pct_avance`) y card 6207 (tareas con `estado = Pendiente`,
  filtradas por el RUT de esa empresa puntual).
- Los 4 porcentajes de conciliación:
  - Para la **empresa madre**, ya vienen en su propia fila de la card 6206
    (`% match cassius`, `% match usuario`, `% asientos cassius`, `% asientos
    manual` — igual fuente que la sección 2).
  - Para cada **hija**, la card 6206 normalmente no trae estos datos (las
    hijas no siempre tienen fila propia ahí). Usá en su lugar la card
    **6301**, filtrada por el nombre de la empresa madre/grupo (ver "Cómo
    ejecutar la card 6301" más arriba): `porcentaje_match_cassius` →
    `{{pct_match_cassius_hija}}`, `porcentaje_match_usuario` →
    `{{pct_match_usuario_hija}}`, `porcentaje_asientos_cassius` →
    `{{pct_asientos_cassius_hija}}`, `porcentaje_asientos_usuario` →
    `{{pct_asientos_usuario_hija}}`.
- `{{tareas_pendientes_lista}}` = nombres de tarea (sin el área) de la card
  6207 con `estado = Pendiente` para esa empresa, unidos con `" · "`. Si no
  tiene ninguna pendiente, escribí "Sin tareas pendientes".

Reglas de dato faltante:

- Ordená la tabla con la empresa madre primero y las hijas debajo, en el
  mismo orden en que aparecen en `rut_empresas_hijas`.
- Si una empresa del grupo no aparece en la card 6206, poné `—` en % Avance
  y "Sin dato en el dashboard" en Tareas pendientes — no la excluyas de la
  tabla.
- Si una hija no aparece en la card 6301 (o el grupo todavía no tiene
  ninguna hija con movimientos/asientos), dejá vacías sus 4 celdas de
  porcentaje — no uses `—` ni "sin datos disponibles", misma regla que la
  sección 2 para los `null`. Esto es distinto de que la hija no aparezca en
  la card 6206: eso sí se marca con `—` como indica el punto anterior.

## 4. Tabla de próximos pasos

Se construye directamente de las filas de la card 6207 para esa empresa —
no la inventes ni reutilices la lista fija de versiones anteriores de esta
skill. Cada fila del resultado (`area`, `tarea`, `estado`, `fecha_completado`)
es una fila de esta tabla. Traducí `estado`:

- `Ok` → `✅ Ok`
- `Pendiente` → `⏳ Pendiente`

| Área | Tarea | Estado | Completado |
|---|---|---|---|
| `{{area}}` | `{{tarea}}` | `{{estado}}` | `{{fecha_completado}}` (o `—` si es null) |

## 5. Sugerencias de uso (product health)

Toma los 3 módulos con `adopt_pct` más bajo desde `clay-dw:product-health` y
genera un texto personalizado por cada uno, cruzando la señal de uso vs.
adopción. Sigue este formato:

| Módulo | Señal del dato | Nivel adopción | Sugerencia de uso |
|---|---|---|---|
| {{modulo}} | {{uso_pct}} pero adopción {{adopt_pct}}. {{interpretación breve}} | 🔴/🟡/🟢 {{categoría}} ({{adopt_pct}}) | {{acción concreta a sugerir}} |

## Trigger de envío (cómo se decide la semana)

El número de semana lo calcula quien invoca esta skill (por ejemplo la
rutina `seguimiento-onboarding` de Claude Code) a partir de `createdate` del
ticket en HubSpot, usando esta correspondencia días→semana:

| Días transcurridos | Semana |
|---|---|
| 14 | 2 |
| 21 | 3 |
| 28 | 4 |
| 35 | 5 |
| 42 | 6 |
| 49 | 7 |
| 56 | 8 |

Si te piden generar un mail de seguimiento directo en el chat sin pasar por
esa rutina, calculalo vos mismo con la misma lógica (día exacto, no rango), o
usá la semana que el usuario te indique explícitamente.

Nota: la card 6206 también trae un campo `semana_onboarding` ya calculado
por el dashboard — puede servir como referencia cruzada si el número que
calculaste desde HubSpot no coincide, pero no lo reemplaces sin entender por
qué difieren (pueden estar contando desde fechas distintas).

## Formato del cuerpo del mail (crítico)

**El cuerpo del draft tiene que ser HTML real, con tablas `<table>` de
verdad — nunca la sintaxis markdown de pipes (`| Col | Col |`) pegada como
texto literal.** Gmail no interpreta markdown: si el borrador queda con
barras `|` visibles en vez de una tabla, es porque se generó como texto
plano en lugar de HTML. Las tablas de este archivo (avance, Cassius,
resumen de grupo, próximos pasos) están en markdown solo porque es más
legible en esta spec — al armar el mail real, convertí cada una a
`<table><tr><th>...</th></tr><tr><td>...</td></tr></table>` con estilos
simples (bordes finos, encabezado en negrita) antes de pasarlo al Gmail MCP.

## Estructura del mail

| Campo | Contenido |
|---|---|
| Asunto | ¿Cómo va {{nombre_empresa}} en Clay? — Semana {{n}} |
| Para | {{email_contacto_principal}} |
| CC | ob@clay.cl |
| Intro | Párrafo corto con el diagnóstico general del período |
| Bloque 1 | Tabla de avance (card 6206) |
| Bloque 2 | Automatización con Cassius (apartado destacado) |
| Bloque 3 | Resumen agregado del grupo — solo si aplica (madre/hijas) |
| Bloque 4 | Tabla de próximos pasos (card 6207) |
| Bloque 5 | Sugerencias de uso (top 3 módulos) o texto de fallback |
| Cierre | Invitación a la siguiente reunión de seguimiento + Calendly |
| Firma | Onboarder (nombre + cargo) |
