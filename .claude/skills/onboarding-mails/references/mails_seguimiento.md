# Mails 2, 3 y 4 — Seguimiento Semana 2, 4 y 6

Misma estructura base para los tres; el contenido evoluciona con el avance
real del cliente. No copies el mail anterior — vuelve a consultar todas las
fuentes cada vez, porque el objetivo es mostrar avance real, no repetir texto.

## Qué hacer en cada uno

1. Consultar el dashboard Metabase "Onboarding - Progreso y Checklist" (ver
   `references/queries_dashboard.md`, queries 1 y 2) para el avance y el
   checklist actualizados. Si no devuelve fila, usar el fallback de
   `clay_empresas_avance` (Clay MCP) y dejarlo explícito.
2. Consultar si la empresa tiene hijas (query 3 de `queries_dashboard.md`).
   Si tiene, traer el checklist de cada una (query 4).
3. Consultar `product_category` (query 6 de `queries_dashboard.md`) para
   saber si la empresa tiene módulo contable activo. Si es
   `Software Gestión Financiera` o `API Bancaria & SII`, excluir los campos
   de asientos/DTE de la tabla de avance y omitir por completo la tabla de
   sugerencias de uso (ver decisión #11 en `decisiones_pendientes.md`).
4. Consultar `clay-dw:product-health` para el health score y adopción por
   módulo (esto no cambió — el dashboard de onboarding no cubre adopción de
   producto). Si el paso 3 determinó que la empresa no tiene módulo
   contable, sáltate este paso — no aplica.
5. Actualizar la tabla de próximos pasos con el estado real de cada tarea.
6. Generar sugerencias personalizadas basadas en los módulos con adopción más
   baja (top 3) — solo si el paso 3 no las excluyó.

## Fallback si no hay datos de product health

Si el cliente es muy nuevo o no tiene actividad registrada en LogRocket,
`clay-dw:product-health` puede no devolver nada. En ese caso no dejes el
bloque de sugerencias vacío ni inventes números — usa este texto tal cual:

> "Aún estamos registrando tu actividad en la plataforma. Mientras tanto,
> aquí van tus próximos pasos pendientes."

Y sáltate la tabla de sugerencias de uso (sección 3 más abajo) para ese mail.

## Fallback si el dashboard no tiene datos de avance/checklist

El dashboard es la **fuente principal**. Si la query de avance (query 1) o la
de checklist (query 2) no devuelven fila para la empresa:

- Para las métricas de avance: usa `clay_empresas_avance` (Clay MCP) como
  fallback para lo que se pueda mapear (ver tabla en `variables.md`). Lo que
  no tenga equivalente ahí, márcalo como "dato no disponible" — no lo
  inventes.
- Para el checklist: no hay un fallback equivalente a nivel de tarea. Si no
  hay filas en el dashboard, dilo explícitamente ("checklist no disponible en
  el dashboard todavía") en vez de mostrar la tabla vacía o asumir que todo
  está pendiente.
- Avisa siempre en tu resumen al usuario cuándo estás usando el fallback, para
  que sepa que esos números pueden no coincidir exactamente con lo que ve en
  el panel de Metabase.

## Fuentes de datos

| Dato | Fuente principal | Fallback |
|---|---|---|
| Avance, movimientos, matches, asientos, DTEs, TC | Dashboard Metabase (`organizations_onboarding_progress`) — ver `queries_dashboard.md` query 1 | `clay_empresas_avance` (Clay MCP) |
| Checklist de próximos pasos (área/tarea/estado) | Dashboard Metabase (`organizations_checklist_status`) — query 2 | Ninguno — marcar "no disponible" |
| Checklist de empresas hijas (si existen) | Dashboard Metabase (`organizations_checklist_grupo`) — queries 3 y 4 | Ninguno — si no hay hijas, se omite la sección |
| Health score y adopción por módulo | DW/Metabase | `clay-dw:product-health` (sin cambios) |
| Módulos visitados / acciones de valor | LogRocket/Metabase | `adoption_scores_daily` + `usage_scores_daily` — dato parcial, puede faltar |
| Fecha inicio OB (para calcular la semana) | HubSpot | propiedad nativa `createdate` del ticket — sigue siendo la fuente oficial, no el `semana_onboarding` del dashboard (ver nota abajo) |
| Comentario del onboarder | Manual | se lo pides al onboarder antes de enviar |

> **`semana_onboarding` del dashboard vs. `createdate` de HubSpot:** el
> dashboard trae su propio cálculo de semana, pero para decidir "qué mail
> toca" seguimos usando `createdate` de HubSpot (ver Paso 0 del SKILL.md
> principal y la rutina `seguimiento-onboarding`). Usa el `semana_onboarding`
> del dashboard solo como dato de contexto si quieres mencionarlo, no para
> decidir el número de mail.

## 1. Tabla de avance de la empresa

Muestra el estado de conciliación. Compárala con la semana anterior cuando
tengas ese dato; si no lo tienes, deja el guion `—` en vez de inventar un
número.

**`% de match` reemplaza a los antiguos `% Conciliación Cassius` y
`% Conciliación por usuario` por separado** — se calcula como
`(movimientos_totales - movimientos_sin_match) / movimientos_totales × 100`,
redondeado a 1 decimal. Es un único número que resume cuánto de lo cargado ya
está conciliado (automático + manual combinado), más fácil de leer de un
vistazo que dos porcentajes separados.

**Filas de asientos y DTE — condicionales según `product_category`** (query 6
de `queries_dashboard.md`): si la empresa es `Software Gestión Financiera` o
`API Bancaria & SII`, esas filas no se incluyen en la tabla — no tiene módulo
contable activo en Clay, así que esos campos van siempre en 0 y no aportan
información real. Para el resto de las categorías, la tabla incluye todas las
filas.

> Igual que en el resto de las tablas de esta skill: lo de abajo es markdown
> solo para mostrar la estructura en la spec. **En el draft real va como
> tabla HTML** (ver Paso 4 de `SKILL.md`) — Gmail no renderiza markdown y la
> tabla queda ilegible como texto plano.

Tabla completa (empresa con módulo contable):

| Métrica | Valor actual | Semana anterior |
|---|---|---|
| % Avance del checklist | {{pct_avance}}% | — |
| Movimientos totales | {{total_movimientos}} | — |
| Movimientos de tarjeta | {{movimientos_tarjeta}} | — |
| Movimientos sin match | {{movimientos_sin_match}} | — |
| % de match | {{pct_match}}% | {{pct_match_prev}}% |
| Asientos contables totales | {{asientos_total}} | — |
| Asientos por Cassius | {{asientos_cassius}} | — |
| Asientos manuales | {{asientos_manual}} | — |
| DTEs por cobrar sin completar | {{dtes_cobrar}} | — |
| DTEs por pagar sin completar | {{dtes_pagar}} | — |
| TC/Medios de pago sin match | {{tc_sin_match}} | — |

Tabla reducida (`Software Gestión Financiera` o `API Bancaria & SII` — sin
filas de asientos/DTE):

| Métrica | Valor actual | Semana anterior |
|---|---|---|
| % Avance del checklist | {{pct_avance}}% | — |
| Movimientos totales | {{total_movimientos}} | — |
| Movimientos de tarjeta | {{movimientos_tarjeta}} | — |
| Movimientos sin match | {{movimientos_sin_match}} | — |
| % de match | {{pct_match}}% | {{pct_match_prev}}% |
| TC/Medios de pago sin match | {{tc_sin_match}} | — |

## 2. Tabla de próximos pasos actualizada

Viene directo de `organizations_checklist_status` (query 2 de
`queries_dashboard.md`) — ya no se calcula a mano ni se infiere desde
`clay_empresas_avance`. Usa exactamente estos dos símbolos (la fuente no trae
un tercer estado intermedio):

- `✅ Ok` — completado (`estado = 'Ok'` en el dashboard)
- `⏳ Pendiente` — no completado (`estado = 'Pendiente'` en el dashboard)

| Área | Tarea | Estado semana {{n}} |
|---|---|---|
| {{area}} | {{tarea}} | {{estado}} |

*(recordatorio: tabla HTML en el draft real, no markdown — ver nota en Sección 1)*

Genera una fila por cada resultado real de la query — no reutilices la lista
fija de 12 tareas de versiones anteriores de esta skill; el dashboard es
ahora la fuente de verdad de qué áreas y tareas existen.

## 3. Checklist de empresas hijas (solo si el grupo tiene hijas)

Ejecuta primero la query 3 de `queries_dashboard.md` para saber si
`{{nombre_empresa}}` tiene hijas. Si no tiene ninguna, **omite esta sección
completa** sin mencionarla en el mail.

Si tiene una o más hijas, agrega un párrafo breve de contexto usando la query
5 (opcional, ej.: "Las {{n_hijas}} empresas del grupo tienen en promedio
{{pct_avance_hijas}}% de avance en su checklist") y luego **una tabla de
checklist completa por cada hija**, con el mismo formato que la tabla de la
madre (Sección 2), cada una bajo su propio subtítulo con el nombre de la
hija:

### Checklist — {{nombre_hija}}

| Área | Tarea | Estado |
|---|---|---|
| {{area}} | {{tarea}} | {{estado}} |

*(recordatorio: tabla HTML en el draft real, no markdown — ver nota en Sección 1)*

Repite este bloque (subtítulo + tabla) una vez por cada hija devuelta en la
query 3, en el mismo orden.

## 4. Sugerencias de uso (product health)

**Omitir esta sección completa si la empresa es `Software Gestión Financiera`
o `API Bancaria & SII`** (query 6 de `queries_dashboard.md`) — los módulos
con menor adopción en estas empresas suelen ser justamente los contables
(Estado de Resultados, Plan de Cuentas, Proveedores), que no aplican porque
la empresa no tiene ese módulo activo. No la reemplaces por nada, simplemente
no aparece en el mail para estos casos (ver decisión #11 en
`decisiones_pendientes.md`).

Para el resto de las categorías, toma los 3 módulos con `adopt_pct` más bajo
desde `adoption_scores_daily` y genera un texto personalizado por cada uno,
cruzando la señal de uso vs. adopción (ej.: usa el módulo pero no lo domina,
vs. ni siquiera lo visita). Sigue este formato — señal del dato concreta, no
genérica:

| Módulo | Señal del dato | Nivel adopción | Sugerencia de uso |
|---|---|---|---|
| {{modulo}} | {{uso_pct}} pero adopción {{adopt_pct}}. {{interpretación breve}} | 🔴/🟡/🟢 {{categoría}} ({{adopt_pct}}) | {{acción concreta a sugerir}} |

Ejemplo real (Miguel Ángel Vargas, via2.cl) para calibrar el nivel de
especificidad esperado:

| Módulo | Señal del dato | Nivel adopción | Sugerencia de uso |
|---|---|---|---|
| Transferencias salientes (/trans/out) | Uso 50% pero adopción 9%. Visita pero no domina el flujo. | 🔴 Intermedia (9%) | Mostrar flujo completo: conciliar pago → categorizar → asociar obligación. |
| Estado de Resultados (/accountingentries/result) | Uso 23% y adopción 15%. No lo ha explorado. | 🔴 Intermedia (15%) | Guía de lectura del ER: ingresos, costos, margen. Comparativa mensual. |
| Dashboard (/dashboard) | Uso 30%, adopción 22%. Entra pero no configura. | 🟡 Avanzada (22%) | Mostrar widgets de conciliación pendiente y estado de flujo de caja. |

## Trigger de envío (cómo se decide la semana)

Esta skill genera el contenido del mail para la semana que le indiquen —
no decide por sí sola cuándo corresponde enviar cada uno. Si te están
llamando desde la rutina `seguimiento-onboarding` de Claude Code, el número
de semana ya viene resuelto: se calcula ahí a partir de `createdate` (fecha
de creación del ticket en HubSpot), no de `fecha_inicio_ob` ni del
`semana_onboarding` del dashboard. `createdate` es la fuente confiable porque
HubSpot la pone sola.

Si te piden generar un mail de seguimiento sin que exista esa rutina
externa (por ejemplo, alguien te pide directamente "generame el de semana 4
de {{cliente}}" en el chat), puedes calcular la semana vos mismo con la
misma lógica: días desde `createdate` hasta hoy — 14/28/42 días — o usar la
semana que el usuario te indique explícitamente si te la da.

| Mail | Días desde `createdate` |
|---|---|
| Semana 2 | 14 |
| Semana 4 | 28 |
| Semana 6 | 42 |

## Estructura del mail

| Campo | Contenido |
|---|---|
| Asunto | ¿Cómo va {{nombre_empresa}} en Clay? — Semana {{n}} |
| Para | {{email_contacto_principal}} |
| CC | ob@clay.cl |
| Intro | Párrafo corto con el diagnóstico general del período |
| Bloque 1 | Tabla de avance — completa o reducida según `product_category` |
| Bloque 2 | Tabla de próximos pasos actualizada (checklist propio) |
| Bloque 3 | Checklist de empresas hijas, una tabla por hija (solo si aplica) |
| Bloque 4 | Sugerencias de uso (top 3 módulos) — **se omite** si `product_category` es `Software Gestión Financiera` o `API Bancaria & SII` |
| Cierre | Invitación a la siguiente reunión de seguimiento + Calendly |
| Firma | Onboarder (nombre + cargo) |
