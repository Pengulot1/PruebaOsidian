# Base de datos

Parte de [[00 - Índice Notificaciones]]. Dueño según el reparto: **Roberto**.

- Mapeo EF: `Infrastructure/Database/ApplicationDbContext.cs` (al final de `OnModelCreating`).
- Migración: `Migrations/20261012193639_notificacion.cs`.
- `Infrastructure/Persistence/Configurations/*Configuration.cs` están **comentados**: el mapeo real vive en el DbContext.
- Todo va en el **esquema `notificacion`**.

## `notificacion.notificacion_envio` (historial de envíos)

Una fila por **cada intento** de correo (flujos 1 y 2).

| Columna | Tipo | Nota |
|---|---|---|
| `id` | int identity, PK | |
| `id_empleado` | int, FK → empleado | |
| `inicio_periodo` / `fin_periodo` | date | |
| `fecha_corte` | timestamptz, null | `null` en los manuales |
| `origen` | varchar(20) | `"Automatico"` / `"Manual"` |
| `estado` | varchar(20) | `"Enviado"` / `"Fallido"` / `"Omitido"` (solo manuales, desde el 15/10) |
| `horas_faltantes` | numeric(6,2) | |
| `id_usuario_ejecutor` | int, null | líder que pidió el aviso manual |
| `fecha_envio` | timestamptz, default `now()` | |
| `detalle_error` | text, null | mensaje de la excepción (`Fallido`) o **motivo** (`Omitido`) |

**Índice único** `uq_idx_notificacion_envio_auto` sobre `(id_empleado, fecha_corte)` **filtrado** `WHERE origen = 'Automatico'` → máximo una fila automática por empleado y corte (RF-007).

- **Escribe:** `RegistrarEnvioHandler`, con un `HandleAsync` para cada evento: `NotificacionEnviada`, `NotificacionFallida` y `NotificacionOmitida`. Todos hacen `Add` + `SaveChanges`. `Omitido` guarda `HorasFaltantes = 0`, `FechaCorte = null` y el motivo en `detalle_error`.
- **Leen:** `EnviarCorreoHandler.DebeEnviarAsync`, `ConsultaHorasFaltantes.NotificadosEnCorteAsync` y `UltimosAvisosManualesAsync` (solo cuentan los `Enviado`), y `GET /api/notificaciones/estado` (cualquier estado).

> [!note] `Omitido` no choca con el índice único
> El índice es solo para `origen = 'Automatico'`, y `Omitido` solo se registra para avisos manuales.

> [!warning] El índice también cuenta los `Fallido`
> Ver [[15 - Riesgos y pendientes]]: si un aviso automático falla y luego se reintenta con éxito en el mismo corte, el `INSERT` del `Enviado` choca con el índice.

## `notificacion.evento_outbox` (cola persistente)

| Columna | Tipo |
|---|---|
| `id` | uuid, PK |
| `tipo_evento` | varchar(100) |
| `payload` | jsonb |
| `estado` | varchar(20): Pendiente / Procesado / Fallido |
| `intentos` | int |
| `ultimo_error` | text, null |
| `fecha_creacion` | timestamptz |
| `fecha_procesado` | timestamptz, null |

Índice `idx_evento_outbox_estado_fecha (estado, fecha_creacion)` → encontrar rápido los pendientes al arrancar.

- **Escribe:** `EventBus`. **Actualiza:** `EventDispatcher`. Ver [[02 - Bus de eventos y Outbox]].

## `notificacion.notificacion_fecha_corte` (flujo 3, sin uso aún)

`anio`, `mes` (CHECK 1–12), `tipo`, `inicio_periodo`, `fin_periodo`, `fecha_corte`, `ejecutada`, `fecha_ejecucion`, `id_usuario_creador`, `fecha_creacion`. Índice único `(anio, mes, tipo)`.

## Tablas existentes que se leen (no se modifican)

- `TblAdministracionEmpleados` (+ Persona, TipoContrato, historial de contratos)
- `TblTimeReportActividadDiaria`
- `TblTimeReportFeriados`

## Consultas útiles para revisar

```sql
-- Últimos envíos
SELECT id_empleado, origen, estado, horas_faltantes, fecha_corte, fecha_envio, detalle_error
FROM notificacion.notificacion_envio ORDER BY fecha_envio DESC LIMIT 50;

-- Eventos atascados o fallidos
SELECT tipo_evento, estado, intentos, ultimo_error, fecha_creacion
FROM notificacion.evento_outbox WHERE estado <> 'Procesado' ORDER BY fecha_creacion;
```
