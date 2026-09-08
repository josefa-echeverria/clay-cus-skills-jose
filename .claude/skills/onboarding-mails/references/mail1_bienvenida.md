# Mail 1 — Minuta de Reunión de Bienvenida

Se envía (como borrador) dentro de las 24 hs posteriores a la primera reunión
con el cliente.

## Propósito

No es un resumen cualquiera: es el primer documento operativo que el cliente
recibe. Tiene que quedar clarísimo qué se hizo en la reunión, qué falta, y
cuál es el primer paso concreto que el cliente tiene que dar.

## Fuentes de datos

| Dato | Fuente | Cómo buscarlo |
|---|---|---|
| Nombre empresa y contacto principal | HubSpot | `search_crm_objects` |
| Estructura de grupo (madre/hijas) | Dashboard 607, pestaña "Próximos Pasos" | `staging_marts.organizations_checklist_grupo` (cards 6230-6234, 6301) — ver detalle abajo |
| Resumen de la reunión (notas Diio) | Diio | `summarize_client_interactions_content` |
| Estado de checklist (bancos, SII, usuarios, etc.) | Dashboard 607, card **6207** | `execute_card` (dashboard_id 607, card_id 6207), filtrar por empresa — ver detalle abajo |
| Avance y automatización Cassius | Dashboard 607, card **6206** | `execute_card` (dashboard_id 607, card_id 6206), filtrar por empresa — ver detalle abajo. **Columnas cambiaron en agosto 2026** — ver `references/mails_seguimiento.md` para la lista actualizada |
| Facturador y XML | Diio | `get_deal_details` |
| Comentario libre del onboarder | Manual | Se lo pides al onboarder antes de enviar |

## Formato del cuerpo del mail (crítico)

**El cuerpo del draft tiene que ser HTML real, con tablas `<table>` de
verdad — nunca la sintaxis markdown de pipes (`| Col | Col |`) pegada como
texto literal.** Gmail no interpreta markdown: si el borrador queda con
barras `|` visibles en vez de una tabla, es porque se generó como texto
plano en lugar de HTML. Convertí siempre las tablas de este mail (próximos
pasos, Cassius, resumen de grupo si aplica) a
`<table><tr><th>...</th></tr><tr><td>...</td></tr></table>` antes de pasarlo
al Gmail MCP.

**Cambio importante:** el estado de conexiones (bancos, SII, usuarios) ya no
se consulta vía Clay MCP ni tablas propias — sale de la misma card 6207 que
usan los mails de seguimiento (`references/mails_seguimiento.md`), porque
"Conectar cuentas bancarias" y "Conectar SII" son tareas del mismo checklist
que trae esa card. No dupliques lógica: es la misma fuente para las 4 tareas
del área "Ajustes Generales" (Activar conciliación automática, Conectar
cuentas bancarias, Conectar SII, Crear usuarios y permisos).

Si la empresa es tan nueva que todavía no aparece en el dashboard 607 (recién
salió de la reunión de bienvenida, antes de la primera sincronización), es
válido no tener estado todavía — decilo explícitamente en el mail en vez de
mostrar `⏳ Pendiente` en todo, que insinúa que ya se revisó. Lo mismo aplica
al bloque de Cassius más abajo: si no hay fila en la card 6206, no inventes
porcentajes.

## Estructura de grupo (empresa madre / hijas)

Antes de armar el mail, revisá si la empresa es madre, hija o independiente
usando el mismo dashboard que el resto de la skill: dashboard 607, pestaña
"Próximos Pasos" (`staging_marts.organizations_checklist_grupo`, cards
6230-6234 y 6301) — **ya no se usan propiedades de HubSpot para esto** (ver
`references/decisiones_pendientes.md`, decisión #6).

Si hay estructura de grupo, sigue exactamente el mismo procedimiento y
formato de tabla que la sección "0. Detectar estructura de grupo" y
"3. Resumen agregado del grupo" de `references/mails_seguimiento.md` — no
dupliques la lógica acá, solo aplicala. La única diferencia en el Mail 1 es
que es más probable que alguna empresa del grupo (o la propia empresa que
dispara el mail) todavía no tenga fila en la card 6206/6207 por ser recién
incorporada — en ese caso, usá `—` / "Sin dato en el dashboard" en vez de
omitir la fila, igual que en seguimiento.

## Tabla de próximos pasos

Usa las 11 tareas reales del checklist (área + tarea), tal como vienen de la
card 6207 — no la lista genérica de versiones anteriores de esta skill:

| Área | Tarea |
|---|---|
| Ajustes Generales | Activar conciliación automática |
| Ajustes Generales | Conectar cuentas bancarias |
| Ajustes Generales | Conectar SII |
| Ajustes Generales | Crear usuarios y permisos |
| Contabilidad | Cargar asiento de apertura |
| Contabilidad | Categorizar clientes y proveedores |
| Gestión Bancaria | Realizar primera conciliación bancaria |
| Gestión Bancaria | Revisar movimientos del último mes |
| Gestión del Negocio | Explorar flujo de caja |
| Gestión del Negocio | Explorar panel de control |
| Obligaciones | Crear Asiento contable Manual |

Si la empresa ya aparece en el dashboard, usa el `estado` real de cada fila
(`Ok` → `✅ Ok`, `Pendiente` → `⏳ Pendiente`) en vez de asumir. Si todavía no
aparece (ver nota arriba), usa esta lista solo como agenda de lo que falta,
sin columna de estado.

## Automatización con Cassius (apartado destacado)

Igual que en los mails de seguimiento, este bloque va siempre en un apartado
propio y visible, con los 4 porcentajes que la card 6206 trae ya calculados
(no hay que calcular nada a mano):

| Métrica | Valor |
|---|---|
| % de match hechos por Cassius | `{{pct_match_cassius}}` |
| % de match hechos por el usuario | `{{pct_match_usuario}}` |
| % de asientos contables hechos por Cassius | `{{pct_asientos_cassius}}` |
| % de asientos contables hechos manualmente | `{{pct_asientos_manual}}` |

Fuente (columnas reales de la card 6206 — ver `references/variables.md`):
`% match cassius`, `% match usuario`, `% asientos cassius`, `% asientos
manual`.

**Regla para dato faltante:** si alguno de estos 4 viene `null` (pasa
cuando `asientos_contables_totales = 0`, muy común en el Mail 1 porque
recién arrancó el onboarding), **dejá la celda vacía en la tabla del mail**,
sin guion ni texto — es a propósito, para que el onboarder la complete a
mano antes de enviar. Si la empresa no tiene fila todavía en la card 6206
(nunca se sincronizó), ahí sí es un problema real — decilo explícitamente en
el mail en vez de dejar el bloque completo vacío sin explicación.

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

## Adjunto

Adjunta automáticamente `onboarding_clay.pptx` al crear el draft en Gmail. Es
obligatorio en este mail — si falla, avisa en vez de omitirlo silenciosamente.

## Estructura del mail

| Campo | Contenido |
|---|---|
| Asunto | Minuta reunión de bienvenida — {{nombre_empresa}} |
| Para | {{email_contacto_principal}} |
| CC | ob@clay.cl |
| Saludo | Hola {{nombre_contacto}}, |
| Cuerpo | Diagnóstico inicial + tabla de próximos pasos (con o sin estado, según disponibilidad) |
| Bloque Cassius | Automatización con Cassius (apartado destacado) |
| Bloque grupo | Resumen agregado del grupo — solo si aplica (madre/hijas) |
| Cierre | "Cualquier duda me avisas. ¡Nos vemos en la próxima reunión!" |
| Firma | Firma del onboarder (nombre + cargo + Calendly) |
| Adjunto | onboarding_clay.pptx |
