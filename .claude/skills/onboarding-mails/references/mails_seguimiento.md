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
   corresponde).
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

**✅ Fuente vigente (septiembre 2026): el propio dashboard 607, pestaña
"Próximos Pasos"** (`analytics.clay.cl/dashboard/607-onboarding-progreso-y-checklist?tab=569-pr%C3%B3ximos-pasos`),
sobre la tabla `staging_marts.organizations_checklist_grupo`. Ya no se
detecta la estructura de grupo consultando propiedades de HubSpot — ver
`references/decisiones_pendientes.md` (decisión #6) para el porqué del
cambio.

Esa tabla trae, por cada tarea de checklist de cada **hija**, estas
columnas: `nombre_grupo` (nombre de la empresa **madre** del grupo), `rol`
(`Madre` o `Hija`), `nombre_empresa`, `area`, `tarea`, `estado`,
`fecha_completado`. Se accede vía `execute_card` con estos cards (todos en
`dashboard_id` 607, pestaña "Próximos Pasos"):

| Card ID | Nombre | Qué trae |
|---|---|---|
| 6230 | Checklist de Grupo - Detalle por Empresa y Tarea (madre + hijas) | Una fila por tarea de cada hija de cada grupo (`rol <> 'Madre'`) |
| 6231 | % Avance Global (Hijas) | % de tareas en `Ok` sobre el total, de todas las hijas de un grupo |
| 6232 | N° Empresas Hijas | Cantidad de hijas distintas de un grupo |
| 6233 | Hijas al 100% | Cuántas hijas tienen el 100% de sus tareas en `Ok` |
| 6234 | Tareas Pendientes (Hijas) | Suma de tareas en `Pendiente` de todas las hijas de un grupo |
| 6301 | Avance y Conciliación — Empresas Hijas (Grupo) | Por cada hija: match Cassius/usuario y % asientos Cassius/usuario (año en curso) |

**Cómo detectar si la empresa que dispara el mail es madre, hija o
independiente:** el MCP de Metabase no soporta pasar el parámetro de filtro
de estos cards (a diferencia de la UI del dashboard), así que hay que
ejecutar `execute_card` (card 6230) **sin filtro** — trae todas las filas de
todos los grupos — y buscar el nombre de la empresa (comparación
case-insensitive, igual que con `nombre_empresa` en las demás cards) en las
columnas `nombre_grupo` y `nombre_empresa`:

- Aparece como `nombre_grupo` → es una **madre**. Sus hijas son todas las
  filas con ese `nombre_grupo` (columna `nombre_empresa`, `rol = 'Hija'`).
- Aparece como `nombre_empresa` con `rol = 'Hija'` → es una **hija**; su
  madre es el valor de `nombre_grupo` de esa fila.
- No aparece en ninguna de las dos columnas → la empresa es independiente:
  seguí el flujo normal (secciones 1, 2 y 4 con los datos de esa única
  empresa) y **saltate la sección 3** (resumen agregado del grupo).

Si detectás una estructura de grupo (madre + una o más hijas):

1. El mail se sigue armando centrado en la empresa puntual que dispara el
   envío (madre o hija) — las secciones 1, 2 y 4 usan sus propios datos de
   la card 6206/6207, como siempre (buscá esa empresa puntual ahí por
   `nombre_empresa`, no por `nombre_grupo`).
2. La sección 3 (resumen agregado) usa los cards 6231-6234 y 6301 filtrados
   (localmente, igual que en el punto anterior) por el `nombre_grupo` de la
   madre — junta a **todas** las hijas del grupo, sin importar cuál de ellas
   disparó el mail.
3. Si alguna hija del grupo no aparece todavía en la card 6206/6207 (por
   ejemplo, recién conectada), incluila igual en la tabla agregada con `—`
   en las columnas que falten — no la omitas ni la saltes en silencio.

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

Esta sección solo aparece si el Paso 0 detectó una estructura de grupo. Los
datos salen de los cards 6230-6234 y 6301 del dashboard 607 (ver Paso 0),
filtrados localmente por el `nombre_grupo` de la madre del grupo detectado.

Primero, un apartado con el resumen a nivel de grupo (no por hija):

| Métrica | Valor |
|---|---|
| % Avance global de las hijas | `{{pct_avance_hijas}}%` (card 6231) |
| N° de empresas hijas | `{{empresas_hijas}}` (card 6232) |
| Hijas con checklist 100% completo | `{{hijas_100}}` (card 6233) |
| Tareas pendientes (todas las hijas) | `{{tareas_pendientes_hijas}}` (card 6234) |

Después, una fila por cada hija del grupo (la madre no va en esta tabla —
sus propios datos ya están en las secciones 1, 2 y 4 si es ella quien
dispara el mail), usando el checklist de la card 6230 y la conciliación de
la card 6301 para cada `nombre_empresa` con `rol = 'Hija'` de ese
`nombre_grupo`:

| Empresa | Tareas pendientes | % Match Cassius | % Asientos Cassius |
|---|---|---|---|
| `{{nombre_empresa}}` | `{{tareas_pendientes_lista}}` | `{{porcentaje_match_cassius}}%` | `{{porcentaje_asientos_cassius}}%` |

- `{{tareas_pendientes_lista}}` = nombres de tarea (sin el área) de la card
  6230 con `estado = Pendiente` para esa hija, unidos con `" · "`. Si no
  tiene ninguna pendiente, escribí "Sin tareas pendientes".
- `{{porcentaje_match_cassius}}` y `{{porcentaje_asientos_cassius}}` salen
  de la card 6301 para esa hija — si la hija no aparece ahí (sin
  movimientos/asientos en el año), dejá `—` en esas dos columnas.
- Ordená la tabla alfabéticamente por `nombre_empresa`, salvo que el
  onboarder pida otro orden.
- Si una hija del grupo no aparece en la card 6230 (por ejemplo, recién
  conectada), incluila igual en la tabla con `—` en las columnas que
  falten — no la excluyas.

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
