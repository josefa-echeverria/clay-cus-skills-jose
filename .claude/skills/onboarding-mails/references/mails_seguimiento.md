# Mails 2, 3 y 4 — Seguimiento Semana 2, 4 y 6

Misma estructura base para los tres; el contenido evoluciona con el avance
real del cliente. No copies el mail anterior — vuelve a consultar todas las
fuentes cada vez, porque el objetivo es mostrar avance real, no repetir texto.

## Qué hacer en cada uno

1. Consultar `clay_empresas_avance` para las métricas de conciliación
   actualizadas.
2. Consultar `clay-dw:product-health` para el health score y adopción por
   módulo.
3. Actualizar la tabla de próximos pasos marcando lo que ya se completó.
4. Generar sugerencias personalizadas basadas en los módulos con adopción más
   baja (top 3).

## Fallback si no hay datos de product health

Si el cliente es muy nuevo o no tiene actividad registrada en LogRocket,
`clay-dw:product-health` puede no devolver nada. En ese caso no dejes el
bloque de sugerencias vacío ni inventes números — usa este texto tal cual:

> "Aún estamos registrando tu actividad en la plataforma. Mientras tanto,
> aquí van tus próximos pasos pendientes."

Y sáltate la tabla de sugerencias de uso (sección 3 más abajo) para ese mail.

## Fuentes de datos

| Dato | Fuente | Herramienta / campo MCP |
|---|---|---|
| Movimientos totales y sin match | Clay MCP | `clay_empresas_avance` |
| Matches Cassius vs. usuario | Clay MCP | `clay_empresas_avance` → `matches_by_user` |
| Asientos por usuario | Clay MCP | `clay_empresas_avance` → `entries_by_user` |
| DTEs por pagar/cobrar sin completar | Clay MCP | `clay_empresas_avance` → `dte`, `boletas` |
| TC y medios de pago sin match | Clay MCP | `clay_empresas_avance` → `creditcard` |
| Health score y adopción por módulo | DW/Metabase | `clay-dw:product-health` |
| Módulos visitados / acciones de valor | LogRocket/Metabase | `adoption_scores_daily` + `usage_scores_daily` — dato parcial, puede faltar |
| Fecha inicio OB (para calcular la semana) | HubSpot | propiedad nativa `createdate` del ticket |
| Comentario del onboarder | Manual | se lo pides al onboarder antes de enviar |

## 1. Tabla de avance de la empresa

Muestra el estado de conciliación diferenciando lo automático (Cassius) de lo
manual (usuario). Compárala con la semana anterior cuando tengas ese dato; si
no lo tienes, deja el guion `—` en vez de inventar un número.

| Métrica | Valor actual | Semana anterior |
|---|---|---|
| Movimientos totales | {{total_movimientos}} | — |
| Movimientos sin match | {{movimientos_sin_match}} | — |
| % Conciliación Cassius (auto) | {{pct_cassius}}% | {{pct_cassius_prev}}% |
| % Conciliación por usuario | {{pct_usuario}}% | {{pct_usuario_prev}}% |
| Asientos contables totales | {{asientos_total}} | — |
| Asientos por Cassius | {{asientos_cassius}} | — |
| Asientos manuales | {{asientos_manual}} | — |
| DTEs por cobrar sin completar | {{dtes_cobrar}} | — |
| DTEs por pagar sin completar | {{dtes_pagar}} | — |
| TC/Medios de pago sin match | {{tc_sin_match}} | — |

## 2. Tabla de próximos pasos actualizada

Misma tabla del Mail 1, pero con estado real calculado desde
`clay_empresas_avance` — no le preguntes al onboarder cuál es el estado, se
calcula. Usa exactamente estos tres símbolos:

- `✅ Ok` — completado
- `🔄 En progreso` — parcialmente hecho
- `⏳ Pendiente` — no iniciado

| Área | Tarea | Estado semana {{n}} |
|---|---|---|
| Ajustes Generales | Crear usuarios y permisos | |
| Ajustes Generales | Conectar cuentas bancarias | |
| Ajustes Generales | Conectar SII | |
| Ajustes Generales | Activar conciliación automática | |
| Contabilidad | Configurar plan de cuentas | |
| Contabilidad | Categorizar clientes y proveedores | |
| Contabilidad | Cargar asiento de apertura | |
| Gestión Bancaria | Revisar movimientos del último mes | |
| Gestión Bancaria | Realizar primera conciliación bancaria | |
| Obligaciones | Revisar documentos por pagar | |
| Obligaciones | Revisar documentos por cobrar | |
| Gestión del Negocio | Explorar panel de control y flujo de caja | |

## 3. Sugerencias de uso (product health)

Toma los 3 módulos con `adopt_pct` más bajo desde `adoption_scores_daily` y
genera un texto personalizado por cada uno, cruzando la señal de uso vs.
adopción (ej.: usa el módulo pero no lo domina, vs. ni siquiera lo visita).
Sigue este formato — señal del dato concreta, no genérica:

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
de creación del ticket en HubSpot), no de `fecha_inicio_ob`. `createdate` es
la fuente confiable porque HubSpot la pone sola; `fecha_inicio_ob` es una
propiedad custom que depende de que alguien la cargue a mano.

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
| Bloque 1 | Tabla de avance (Cassius vs. usuario) |
| Bloque 2 | Tabla de próximos pasos actualizada |
| Bloque 3 | Sugerencias de uso (top 3 módulos) o texto de fallback |
| Cierre | Invitación a la siguiente reunión de seguimiento + Calendly |
| Firma | Onboarder (nombre + cargo) |
