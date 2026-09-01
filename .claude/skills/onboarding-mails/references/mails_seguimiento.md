# Mails 2 a 8 — Seguimiento Semana 2 a 8

Misma estructura base para las siete instancias; el contenido evoluciona con
el avance real del cliente. No copies el mail anterior — vuelve a consultar
el dashboard cada vez, porque el objetivo es mostrar avance real, no repetir
texto.

## Fuente de datos: SOLO el dashboard de Metabase (actualizado septiembre 2026)

**Toda la información de avance, conciliación y checklist viene
exclusivamente del dashboard `Onboarding - Progreso y Checklist` (id 607) y
de las cards de la colección "Onboarding" de Metabase.** No consultes
`sources.*` directamente, no uses `clay_empresas_avance` (Clay MCP), no
inventes nombres de tabla ni de columna.

Usa la herramienta `execute_card` de Metabase Clay con `dashboard_id = 607`
para 6206 y 6207. Las cards 6301 y 6230 tienen parámetro obligatorio y
`execute_card` no soporta pasarle parámetros — para esas dos se usa
`execute_query` con el SQL exacto de la card (ver las subsecciones "Cómo
ejecutar..." más abajo):

| Card | `card_id` | Qué trae |
|---|---|---|
| Onboarding - Avance y Conciliación por Empresa | **6206** | Una fila por empresa: `rut_empresa`, `nombre_empresa`, `semana_onboarding`, `onboarder_asignado`, `pct_avance`, `tareas_ok`, `tareas_pendientes`, `movimientos_totales`, `movimientos_tarjeta`, `match cassius (n)`, `match usuario (n)`, `movimientos_sin_match`, `% match cassius`, `% match usuario`, `tc_medios_pago_sin_match`, `asientos_contables_totales`, `asientos_por_cassius`, `asientos_manuales`, `% asientos cassius`, `% asientos manual`, `dtes_por_cobrar_sin_contabilizar`, `dtes_por_pagar_sin_contabilizar`. **Columnas con espacio en el nombre — van entre comillas/tal cual las devuelve `execute_card`.** |
| Checklist - Detalle por Empresa y Tarea | **6207** | Una fila por tarea: `rut_empresa`, `nombre_empresa`, `onboarder_asignado`, `area`, `tarea`, `estado` (`Ok`/`Pendiente`), `fecha_completado`. Cubre empresas independientes y la empresa madre — **no** trae filas de hijas. |
| Avance y Conciliación — Empresas Hijas (Grupo) | **6301** | **Solo aplica a hijas dentro de una estructura de grupo (sección 3 más abajo).** Una fila por hija: `nombre_empresa`, `match_cassius_n`, `match_usuario_n`, `porcentaje_match_cassius`, `porcentaje_match_usuario`, `asientos_totales`, `asientos_cassius`, `asientos_usuario`, `porcentaje_asientos_cassius`, `porcentaje_asientos_usuario`. Colección "Onboarding" (`analytics.clay.cl/question/6301-...`) — agregada en septiembre 2026 (IAD-255) porque las hijas suelen no tener fila propia en la card 6206. |
| Checklist de Grupo - Detalle por Empresa y Tarea (madre + hijas) | **6230** | **Solo aplica a hijas dentro de una estructura de grupo (sección 3 más abajo).** Una fila por tarea de cada hija: `nombre_grupo`, `rol` (siempre `Hija` — la card excluye a la madre), `nombre_empresa`, `area`, `tarea`, `estado` (`Ok`/`Pendiente`), `fecha_completado`. Colección "Onboarding" (`analytics.clay.cl/question/6230-checklist-de-grupo-detalle-por-empresa-y-tarea-madre-hijas`) — es el equivalente de la card 6207 pero para hijas. |

**Ninguna de las cuatro cards acepta filtro de empresa desde `execute_card`
directamente** (6206/6207 devuelven todas las empresas de una vez —
filtrálas localmente por `nombre_empresa`, sin distinguir
mayúsculas/minúsculas, ej. "Carotrini" debe matchear con "CAROTRINI SPA"; si
no hay match exacto, probá comparación parcial antes de reportar "sin
dato". 6301/6230 sí tienen parámetro de filtro pero hay que pasarlo vía
`execute_query`, ver más abajo).

Si en algún momento se agregan filtros a estos cards, cambian sus `card_id`,
o `execute_card` empieza a soportar parámetros, actualizar esta tabla y las
subsecciones "Cómo ejecutar..." — no asumir que lo de acá queda fijo para
siempre; volver a inspeccionar el dashboard/colección si algo no calza.

## Qué hacer en cada uno

1. Revisar en HubSpot si la empresa forma parte de una estructura de grupo
   (ver sección "0. Detectar estructura de grupo" más abajo) — esto decide
   si armás la sección 2 (empresa sin grupo) o la sección 3 (resumen del
   grupo), que son mutuamente excluyentes.
2. Ejecutar `execute_card` para los cards 6206 y 6207 (dashboard 607),
   filtrando por la empresa que dispara el mail. Si es un grupo (madre o
   hija), además correr las cards 6301 y 6230 vía `execute_query` (ver
   sección 0) para el % de conciliación y las tareas pendientes de cada
   hija.
3. Armar el mail con la sección 1 (siempre) y exactamente una de la sección
   2 o la sección 3, según corresponda.

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

**Estos tres campos pueden estar vacíos en HubSpot aunque la empresa sí sea
un grupo real** (pasó con Foil en septiembre 2026: los tres campos estaban
vacíos pero la tabla `staging_marts.organizations_checklist_grupo` sí la
tenía como "Madre" con 11 hijas). Si los tres campos de HubSpot vienen
vacíos, antes de asumir que la empresa es independiente, cruzá también
contra esa tabla (por ejemplo ejecutando la card 6301 o 6230 con el nombre
de la empresa — si devuelven filas, es un grupo) y avisá del desajuste para
que OPS complete los campos de HubSpot.

Si ningún campo (ni HubSpot ni el cruce anterior) confirma un grupo, la
empresa es independiente: seguí el flujo normal (secciones 1 y 2 con los
datos de esa única empresa) y **saltate la sección 3**.

Si detectás una estructura de grupo (madre + una o más hijas):

1. Reuní el RUT de todas las empresas del grupo: la madre + todas las hijas
   listadas en `rut_empresas_hijas` (o las que aparezcan asociadas vía
   `hs_parent_company_id`, o vía el cruce con 6301/6230 si el campo de texto
   está vacío o incompleto).
2. Ejecutá `execute_card` (card 6206) una sola vez para todo el grupo — ya
   trae todas las empresas de onboarding — y filtrá localmente por esos RUT
   (comparación case-insensitive, igual que con `nombre_empresa`).
3. Ejecutá también las cards **6301** y **6230** (ver subsecciones abajo)
   para traer, de cada hija, el % de conciliación (match y asientos) y sus
   tareas pendientes — estos datos normalmente **no** están en las cards
   6206/6207 para las hijas.
4. El mail se sigue armando centrado en la empresa puntual que dispara el
   envío (madre o hija) — la sección 1 usa sus propios datos, como siempre.
   La sección 3 (resumen agregado) es la que junta a **todas** las empresas
   del grupo, y reemplaza a la sección 2 (no armes las dos).
5. Si alguna empresa del grupo no aparece todavía en la card 6206, incluila
   igual en la tabla agregada con `—` en % Avance — no la omitas ni la
   saltes en silencio. Lo mismo si una hija no aparece en 6301 (dejá vacíos
   los 4 porcentajes de conciliación) o en 6230 (escribí "Sin dato en el
   dashboard" en vez de un punteo).

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

### Cómo ejecutar la card 6230 (parámetro `nombre_empresa` = nombre de la madre/grupo)

Misma lógica que la card 6301: vive en la colección "Onboarding"
(`analytics.clay.cl/question/6230-checklist-de-grupo-detalle-por-empresa-y-tarea-madre-hijas`),
el parámetro `nombre_empresa` es en realidad el nombre del grupo/madre, y
`execute_card` no soporta pasarle parámetros — usá `execute_query`
(database_id 40) con el SQL exacto de la card:

```sql
SELECT
  organizations_checklist_grupo.nombre_grupo,
  organizations_checklist_grupo.rol,
  organizations_checklist_grupo.nombre_empresa,
  organizations_checklist_grupo.area,
  organizations_checklist_grupo.tarea,
  organizations_checklist_grupo.estado,
  organizations_checklist_grupo.fecha_completado
FROM staging_marts.organizations_checklist_grupo
WHERE 1 = 1
  AND organizations_checklist_grupo.nombre_grupo = 'NOMBRE_DE_LA_MADRE'
  AND rol <> 'Madre'
ORDER BY
  organizations_checklist_grupo.rol DESC,
  organizations_checklist_grupo.nombre_empresa,
  organizations_checklist_grupo.area,
  organizations_checklist_grupo.tarea;
```

Esta card **excluye a la madre** (`rol <> 'Madre'`) — trae solo filas de
hijas. Las tareas de la madre siguen saliendo de la card 6207, como
siempre. Filtrá el resultado localmente por `estado = 'Pendiente'` y
agrupá por `nombre_empresa` para armar el punteo de cada hija en la
sección 3.

## 1. Tabla de avance de la empresa

Sale directo de la fila de la card 6206 para esa empresa. **No incluyas
columna de "semana anterior" ni ninguna comparación histórica** — el
onboarder no tiene forma confiable de recuperar ese dato de un mail previo,
así que se sacó de la plantilla. Solo dos columnas:

**Cambio (agosto 2026): la card 6206 se modificó.** Las columnas viejas
`pct_conciliacion_cassius_auto` y `pct_conciliacion_usuario` ya no existen —
fueron reemplazadas por `% match cassius` y `% match usuario` (más las
columnas nuevas de asientos). Si esta tabla no calza con lo que devuelve
`execute_card`, volvé a inspeccionar las columnas reales antes de asumir que
siguen igual.

| Métrica | Valor |
|---|---|
| % Avance total del checklist | `{{pct_avance}}%` |
| Tareas completadas / pendientes | `{{tareas_ok}}` / `{{tareas_pendientes}}` |
| Movimientos totales | `{{movimientos_totales}}` |
| Movimientos sin match | `{{movimientos_sin_match}}` |
| Match Cassius (cantidad) | `{{match_cassius_n}}` |
| Match usuario (cantidad) | `{{match_usuario_n}}` |
| Movimientos de tarjeta | `{{movimientos_tarjeta}}` |
| TC/Medios de pago sin match | `{{tc_medios_pago_sin_match}}` |
| Asientos contables totales | `{{asientos_contables_totales}}` |
| Asientos por Cassius | `{{asientos_por_cassius}}` |
| Asientos manuales | `{{asientos_manuales}}` |
| DTEs por cobrar sin contabilizar | `{{dtes_por_cobrar_sin_contabilizar}}` |
| DTEs por pagar sin contabilizar | `{{dtes_por_pagar_sin_contabilizar}}` |

Los 4 porcentajes destacados (% match Cassius, % match usuario, % asientos
Cassius, % asientos manual) **no van en esta tabla** — van en la sección 2
(empresa sin grupo) o en la fila correspondiente de la sección 3 (empresa
en grupo), nunca en las dos a la vez.

## 2. Automatización con Cassius y tareas pendientes (solo empresas SIN grupo)

**Esta sección se salta por completo si la empresa forma parte de un grupo**
(Paso 0) — en ese caso, todo esto ya sale en la fila correspondiente de la
sección 3, y repetirlo sería duplicado. Se arma solo para empresas
independientes.

Primero el apartado de Cassius, con los 4 porcentajes que la card 6206 ya
trae calculados — no hay que calcular nada a mano:

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

Debajo, agregá el punteo de tareas pendientes de esa empresa (card 6207,
`estado = Pendiente`, filtrado por su RUT) — **solo los nombres de tarea que
están pendientes**, como lista con viñetas, nunca la tabla completa con
todas las tareas Ok incluidas:

```html
<h3>Tareas pendientes</h3>
<ul>
  <li>{{tarea pendiente 1}}</li>
  <li>{{tarea pendiente 2}}</li>
</ul>
```

Si no tiene ninguna tarea pendiente, reemplazá la lista por el texto "Sin
tareas pendientes" (no dejes un `<ul>` vacío).

## 3. Resumen agregado del grupo (solo si hay empresa madre/hijas)

Esta sección solo aparece si el Paso 0 detectó una estructura de grupo, y
**reemplaza por completo a la sección 2** (no armes las dos: los mismos 4
porcentajes de Cassius y las tareas pendientes de la empresa disparadora ya
están en su fila de esta tabla). Una fila por cada empresa del grupo
(incluida la que dispara el mail), con su avance, sus 4 porcentajes de
conciliación y sus tareas pendientes como punteo:

| Empresa | % Avance | % Match Cassius | % Match Usuario | % Asientos Cassius | % Asientos Usuario | Tareas pendientes |
|---|---|---|---|---|---|---|
| `{{nombre_empresa}}` | `{{pct_avance}}%` | `{{pct_match_cassius_hija}}` | `{{pct_match_usuario_hija}}` | `{{pct_asientos_cassius_hija}}` | `{{pct_asientos_usuario_hija}}` | punteo de tareas pendientes |

Fuente de cada columna:

- `{{pct_avance}}` — card 6206 (`pct_avance`), filtrada por el RUT de esa
  empresa puntual. Si la empresa no aparece en 6206, poné `—`.
- Los 4 porcentajes de conciliación:
  - Para la **empresa madre**, salen de su propia fila de la card 6206
    (`% match cassius`, `% match usuario`, `% asientos cassius`, `%
    asientos manual`).
  - Para cada **hija**, la card 6206 normalmente no trae estos datos. Usá
    en su lugar la card **6301** (ver "Cómo ejecutar la card 6301" más
    arriba): `porcentaje_match_cassius` → `{{pct_match_cassius_hija}}`,
    `porcentaje_match_usuario` → `{{pct_match_usuario_hija}}`,
    `porcentaje_asientos_cassius` → `{{pct_asientos_cassius_hija}}`,
    `porcentaje_asientos_usuario` → `{{pct_asientos_usuario_hija}}`.
- Tareas pendientes, **como punteo (lista con viñetas) mostrando solo las
  tareas en estado `Pendiente`** — nunca como tabla completa ni como texto
  corrido:
  - Para la **empresa madre**, de la card 6207 (`estado = Pendiente`,
    filtrado por su RUT).
  - Para cada **hija**, de la card **6230** ("Cómo ejecutar la card 6230"
    más arriba), filtrando las filas de esa hija con `estado = Pendiente`.
  - Si una empresa (madre o hija) no tiene ninguna tarea pendiente,
    escribí "Sin tareas pendientes" en vez de un `<ul>` vacío.

**Plantilla HTML de la celda de tareas pendientes** (dentro de la fila de
cada empresa en la tabla de la sección 3):

```html
<ul style="margin:0; padding-left:16px;">
  <li>Cargar asiento de apertura</li>
</ul>
```

Reglas de dato faltante:

- Ordená la tabla con la empresa madre primero y las hijas debajo, en el
  mismo orden en que aparecen en `rut_empresas_hijas` (o alfabético si ese
  campo está vacío — avisá que se ordenó así por falta de dato).
- Si una empresa del grupo no aparece en la card 6206, poné `—` en % Avance
  — no la excluyas de la tabla.
- Si una hija no aparece en la card 6301 (o el grupo todavía no tiene
  ninguna hija con movimientos/asientos), dejá vacías sus 4 celdas de
  porcentaje — no uses `—` ni "sin datos disponibles", misma regla que la
  sección 2 para los `null`.
- Si una hija no aparece en la card 6230, escribí "Sin dato en el
  dashboard" en la celda de tareas pendientes (distinto de "Sin tareas
  pendientes", que significa que sí tiene datos y están todas Ok).

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

**El cuerpo del draft tiene que ser HTML real, con tablas `<table>` y listas
`<ul>` de verdad — nunca la sintaxis markdown de pipes (`| Col | Col |`) ni
guiones de lista pegados como texto literal.** Gmail no interpreta
markdown: si el borrador queda con barras `|` visibles en vez de una tabla,
es porque se generó como texto plano en lugar de HTML. Las tablas de este
archivo (avance, Cassius, resumen de grupo) están en markdown solo porque es
más legible en esta spec — al armar el mail real, convertí cada una a
`<table><tr><th>...</th></tr><tr><td>...</td></tr></table>` (y cada punteo a
`<ul><li>...</li></ul>`) con estilos simples (bordes finos, encabezado en
negrita) antes de pasarlo al Gmail MCP.

## Estructura del mail

| Campo | Contenido |
|---|---|
| Asunto | ¿Cómo va {{nombre_empresa}} en Clay? — Semana {{n}} |
| Para | {{email_contacto_principal}} |
| CC | ob@clay.cl |
| Intro | Párrafo corto con el diagnóstico general del período |
| Bloque 1 | Tabla de avance (card 6206, sin comparación histórica) |
| Bloque 2 | Automatización con Cassius + tareas pendientes — solo si la empresa NO tiene grupo |
| Bloque 2' | Resumen agregado del grupo (avance + conciliación + tareas pendientes por empresa) — solo si la empresa SÍ tiene grupo. Reemplaza al Bloque 2, nunca van los dos. |
| Cierre | Invitación a la siguiente reunión de seguimiento + Calendly |
| Firma | Onboarder (nombre + cargo) |
