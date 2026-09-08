# Decisiones pendientes para OPS

Estas son las decisiones que la spec original (Josefa, CUS, v1.0 — julio
2026) y los ajustes posteriores dejan abiertas. Mientras OPS no las
resuelva formalmente, la skill opera con el default indicado. Si el usuario
te pide algo que choca con un default, dilo explícitamente en vez de
aplicarlo en silencio.

| # | Decisión pendiente | Default actual de la skill |
|---|---|---|
| 1 | ¿Fuente de datos de avance/checklist? | **✅ Resuelto (julio 2026):** dashboard de Metabase `Onboarding - Progreso y Checklist` (id 607), cards 6206 y 6207, vía `execute_card`. Ver "Historial" abajo — se descartaron Clay MCP y consultas directas a `sources.*` en el DW. |
| 2 | ¿El comentario del onboarder se ingresa en el chat o en un formulario previo? | En el chat: pregúntaselo al usuario/onboarder como parte de la conversación antes de generar el borrador final. |
| 3 | ¿Calendly por onboarder o uno general de Clay? | Por onboarder: usa el link que el onboarder te indique en la conversación. Si no lo tenés, pregúntalo — no inventes ni dejes un link genérico. |
| 4 | ¿Cómo se maneja el envío si el cliente tiene múltiples usuarios? | Enviar solo al contacto principal (`{{email_contacto_principal}}` de HubSpot) en Para + `ob@clay.cl` en CC. Si el onboarder pide incluir a más gente, agrégalos en CC, pero no lo hagas por defecto. |
| 5 | ¿El skill puede ejecutarse manualmente fuera de las fechas automáticas? | Sí — siempre se ejecuta a pedido del onboarder en el chat (o de la rutina `seguimiento-onboarding` de Claude Code), calculando igual la fecha/semana correspondiente para dar contexto. |
| 6 | ¿Cómo se detecta si una empresa tiene estructura de grupo (madre/hijas)? | **✅ Resuelto — actualizado (septiembre 2026):** dashboard 607, pestaña "Próximos Pasos" (`staging_marts.organizations_checklist_grupo`, cards 6230-6234 y 6301 vía `execute_card`). Reemplaza el approach anterior de propiedades de HubSpot (`rut_empresa_madre`, `rut_empresas_hijas`, `hs_parent_company_id`) — ver "Historial: por qué se dejó de usar HubSpot para grupo" abajo. |
| 7 | ¿El % de asientos hechos por Cassius viene calculado en el dashboard 607? | **✅ Resuelto (actualizado el 7 de agosto de 2026):** sí — Piero agregó las columnas `% asientos cassius` y `% asientos manual` directo a la card 6206 el 6 de agosto. Ya no hace falta calcularlas a mano (`asientos_por_cassius / asientos_contables_totales`); usar las columnas directo, ver `references/variables.md`. |

## Historial: cómo se llegó a usar el dashboard 607 (julio 2026)

Vale la pena dejarlo escrito para que si algo vuelve a fallar, no se repita
todo el proceso de investigación:

1. **Intento 1 — Clay MCP (`clay_empresas_avance`):** no traía datos de
   forma confiable. Se detectó con Carotrini, que devolvió tablas vacías al
   probarlo con el equipo.
2. **Intento 2 — Consultar `sources.*` directo en el DW dev (database_id
   41):** se identificaron las tablas reales (`sources.movements` con
   `zero_difference`, `sources.accounting_entries` con `created_by`,
   `sources.accounting_entries_move` con `created_by`/`accounted_at`), pero
   **el DW dev estaba vacío para estos datos** — se confirmó comparando
   contra producción (`database_id 40`): GECO tenía 0 movimientos en dev vs.
   30 en prod; HakaLab, 0 en dev vs. 143 en prod. Sole no tiene ni quiere
   acceso directo a producción, así que se pausó esta vía.
3. **Intento 3 (actual) — Dashboard 607 vía `execute_card`:** Sole señaló
   que ya existe un dashboard armado (`Onboarding - Progreso y Checklist`,
   id 607, colección "Onboarding", armado por Piero Oporto) que integra todo
   esto con datos reales, sobre tablas ya preparadas
   (`staging_marts.organizations_onboarding_progress` y
   `staging_marts.organizations_checklist_status`, en `database_id 40`).
   Como se accede vía cards ya curadas de Metabase (no SQL directo a
   `sources.*`), esto no requiere que Sole tenga acceso a producción —
   solo permiso para ver/ejecutar ese dashboard, que ya tiene. Se validó con
   Carotrini: la card 6207 mostró checklist real (tareas Ok/Pendiente con
   fechas), y la card 6206 mostró `pct_avance = 54.5%` con movimientos y
   conciliación reales — nada de ceros. **Esta es la fuente vigente.**

Nota aparte: durante el intento 2 se detectó que Carotrini tiene **dos
registros de organización duplicados** en `sources.organizations` (RUTs
`13471954` y `77948925`) y que el campo `organizations.activated` está en
`false` para las 5.434 organizaciones de la tabla (no sirve como señal). Si
en el futuro se vuelve a tocar `sources.*` directamente, tener esto en
cuenta.

## Historial: por qué se dejó de usar HubSpot para detectar grupo (septiembre 2026)

La v1 de esta skill (julio-agosto 2026) detectaba la estructura de grupo
consultando `rut_empresa_madre` y `rut_empresas_hijas` en HubSpot
(companies). En la práctica esto fallaba en el chat: al no tener un lugar
único y confiable donde verificar esos campos (propiedades a veces vacías,
o el modelo no sabía bien dónde ir a buscarlas), la detección de grupo
quedaba inconsistente.

Josefa señaló que Piero ya había armado, directo en el dashboard 607
("Onboarding - Progreso y Checklist"), pestaña **"Próximos Pasos"**
(`analytics.clay.cl/dashboard/607-onboarding-progreso-y-checklist?tab=569-pr%C3%B3ximos-pasos`),
una sección completa dedicada a esto: la tabla
`staging_marts.organizations_checklist_grupo` (columnas `nombre_grupo`,
`rol` = `Madre`/`Hija`, `nombre_empresa`, `area`, `tarea`, `estado`,
`fecha_completado`) y los cards 6230-6234 y 6301 que calculan sobre ella el
checklist y la conciliación agregada de las hijas. Se validó con
"INVERSIONES FOIL SPA" (la madre) y su hija "CONSTRUCTORA PACIFICO BOX
SPA": el card 6230 sin filtro trae la fila
`nombre_grupo=INVERSIONES FOIL SPA, rol=Hija, nombre_empresa=CONSTRUCTORA
PACIFICO BOX SPA, ...` — dato real, no inventado.

Como esta tabla ya vive en la misma colección "Onboarding" y el mismo
`database_id` (40) que `organizations_onboarding_progress` y
`organizations_checklist_status` (las que ya usa el resto de la skill), se
adoptó como fuente única también para grupo — evita depender de que las
propiedades de HubSpot estén bien cargadas y mantiene todo el dato de
onboarding en un solo lugar. **Esta es la fuente vigente**, ver
`references/mails_seguimiento.md` sección "0. Detectar estructura de
grupo".

### Bug encontrado en la primera prueba real (Camila, septiembre 2026): % Cassius de las hijas venía vacío

Camila (onboarder) probó la skill con INVERSIONES FOIL SPA: el resumen
agregado trajo bien las 11 hijas y sus tareas pendientes (cards 6230-6234),
pero las columnas de % Match Cassius y % Asientos Cassius (card 6301)
salieron `—` en todas las filas, aun en empresas con asientos y movimientos
reales.

Causa: a diferencia de las cards 6230-6234, la SQL de la card 6301 tiene el
filtro `WHERE nombre_grupo = {{nombre_empresa}}` **sin** envolver en
`[[ ]]` — es un parámetro obligatorio. El MCP de Metabase no tiene forma de
pasarle ese parámetro a `execute_card`, así que ejecutarla sin filtro no
devuelve "todo sin filtrar" (como sí pasa con 6230-6234) sino un error 400.
El modelo interpretó ese error como "sin datos" y completó todo con `—` en
vez de reportar el problema.

Se confirmó el fix con `execute_query` (database_id 40) corriendo la SQL
exacta de la card 6301 con `nombre_grupo = 'INVERSIONES FOIL SPA'` puesto
directo en el texto — trajo los 11 registros con datos reales (ej.
Constructora Pacifico Box SPA: 99.3% de asientos por Cassius; Parque
Chagual SPA: 0%, consistente con que le falta "Realizar primera
conciliación bancaria"). Queda documentado en
`references/mails_seguimiento.md` (sección "0", nota bajo la tabla de
cards) y `references/variables.md`.
