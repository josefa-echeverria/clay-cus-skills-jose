# Mails 2, 3 y 4 — Seguimiento Semana 2, 4 y 6

Misma estructura base para los tres; el contenido evoluciona con el avance
real del cliente. No copies el mail anterior — vuelve a consultar el
dashboard cada vez, porque el objetivo es mostrar avance real, no repetir
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
| Onboarding - Avance y Conciliación por Empresa | **6206** | Una fila por empresa: `rut_empresa`, `nombre_empresa`, `semana_onboarding`, `onboarder_asignado`, `pct_avance`, `tareas_ok`, `tareas_pendientes`, `movimientos_totales`, `movimientos_tarjeta`, `movimientos_sin_match`, `pct_conciliacion_cassius_auto`, `pct_conciliacion_usuario`, `tc_medios_pago_sin_match`, `asientos_contables_totales`, `asientos_por_cassius`, `asientos_manuales`, `dtes_por_cobrar_sin_contabilizar`, `dtes_por_pagar_sin_contabilizar` |
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
3. El mail se sigue armando centrado en la empresa puntual que dispara el
   envío (madre o hija) — las secciones 1, 2 y 4 usan sus propios datos,
   como siempre. La sección 3 (resumen agregado) es la única que junta a
   **todas** las empresas del grupo.
4. Si alguna empresa del grupo no aparece todavía en la card 6206 (por
   ejemplo, recién conectada), incluila igual en la tabla agregada con `—`
   en las columnas que falten — no la omitas ni la saltes en silencio.

## 1. Tabla de avance de la empresa

Sale directo de la fila de la card 6206 para esa empresa. Compará contra la
semana anterior solo si tenés ese dato guardado de un mail previo (no lo
inventes ni lo dejes en blanco — usá `—` si no lo tenés).

| Métrica | Valor actual | Semana anterior |
|---|---|---|
| % Avance total del checklist | `{{pct_avance}}%` | — |
| Tareas completadas / pendientes | `{{tareas_ok}}` / `{{tareas_pendientes}}` | — |
| Movimientos totales | `{{movimientos_totales}}` | — |
| Movimientos sin match | `{{movimientos_sin_match}}` | — |
| % Conciliación Cassius (auto) | `{{pct_conciliacion_cassius_auto}}%` | — |
| % Conciliación por usuario | `{{pct_conciliacion_usuario}}%` | — |
| Movimientos de tarjeta | `{{movimientos_tarjeta}}` | — |
| TC/Medios de pago sin match | `{{tc_medios_pago_sin_match}}` | — |
| Asientos contables totales | `{{asientos_contables_totales}}` | — |
| Asientos por Cassius | `{{asientos_por_cassius}}` | — |
| Asientos manuales | `{{asientos_manuales}}` | — |
| DTEs por cobrar sin contabilizar | `{{dtes_por_cobrar_sin_contabilizar}}` | — |
| DTEs por pagar sin contabilizar | `{{dtes_por_pagar_sin_contabilizar}}` | — |

## 2. Automatización con Cassius (apartado destacado)

Este bloque va **siempre**, en un apartado propio y visible (no mezclado
dentro de la tabla de avance de la sección 1, aunque los datos de origen
sean los mismos), porque es lo que más le interesa mostrar al cliente sobre
cuánto está trabajando la plataforma por él:

| Métrica | Valor |
|---|---|
| % de match (conciliación) hechos por Cassius | `{{pct_match_cassius}}%` |
| % de asientos contables hechos por Cassius | `{{pct_asientos_cassius}}%` |

Cómo se calcula cada variable (ver también `references/variables.md`):

- `{{pct_match_cassius}}` = `pct_conciliacion_cassius_auto` de la card 6206,
  tal cual viene — es el mismo dato que en la tabla de avance, solo que acá
  se destaca solo.
- `{{pct_asientos_cassius}}` = **no viene directo de la card, hay que
  calcularlo**: `asientos_por_cassius / asientos_contables_totales * 100`,
  redondeado a 1 decimal. Si `asientos_contables_totales` es 0, no dividas
  por cero — mostrá "sin asientos registrados aún" en esa fila.

Si la empresa todavía no tiene fila en la card 6206, mostrá el bloque
completo con el texto "Aún no hay datos de automatización disponibles" en
vez de inventar porcentajes o dejarlo vacío.

## 3. Resumen agregado del grupo (solo si hay empresa madre/hijas)

Esta sección solo aparece si el Paso 0 detectó una estructura de grupo.
Una fila por cada empresa del grupo (incluida la que dispara el mail),
usando `pct_avance` de la card 6206 y las tareas en estado `Pendiente` de la
card 6207 para cada RUT del grupo:

| Empresa | % Avance | Tareas pendientes |
|---|---|---|
| `{{nombre_empresa}}` | `{{pct_avance}}%` | `{{tareas_pendientes_lista}}` |

- `{{tareas_pendientes_lista}}` = nombres de tarea (sin el área) de la card
  6207 con `estado = Pendiente` para esa empresa, unidos con `" · "`. Si no
  tiene ninguna pendiente, escribí "Sin tareas pendientes".
- Ordená la tabla con la empresa madre primero y las hijas debajo, en el
  mismo orden en que aparecen en `rut_empresas_hijas`.
- Si una empresa del grupo no aparece en la card 6206, poné `—` en % Avance
  y "Sin dato en el dashboard" en Tareas pendientes — no la excluyas de la
  tabla.

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
ticket en HubSpot — 14/28/42 días. Si te piden generar un mail de
seguimiento directo en el chat sin pasar por esa rutina, calculalo vos mismo
con la misma lógica, o usá la semana que el usuario te indique explícitamente.

Nota: la card 6206 también trae un campo `semana_onboarding` ya calculado
por el dashboard — puede servir como referencia cruzada si el número que
calculaste desde HubSpot no coincide, pero no lo reemplaces sin entender por
qué difieren (pueden estar contando desde fechas distintas).

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
