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
| 6 | ¿Cómo se detecta si una empresa tiene estructura de grupo (madre/hijas)? | **✅ Resuelto (agosto 2026):** propiedades de HubSpot en el objeto companies: `rut_empresa_madre` (si la empresa es hija) y `rut_empresas_hijas` (si es madre), confirmadas vía `search_properties`. Se usa `hs_parent_company_id` (asociación nativa) solo como respaldo/cruce si los dos campos de RUT faltan o no coinciden — no lo reemplaza. |
| 7 | ¿El % de asientos hechos por Cassius viene calculado en el dashboard 607? | **No.** La card 6206 trae los conteos (`asientos_contables_totales`, `asientos_por_cassius`) pero no el porcentaje ya calculado. La skill lo calcula ella misma: `asientos_por_cassius / asientos_contables_totales * 100`. Si en algún momento Piero agrega esta columna directo a la card 6206, hay que actualizar `references/variables.md` y dejar de calcularlo a mano. |

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
