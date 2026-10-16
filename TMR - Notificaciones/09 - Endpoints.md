# Endpoints

Parte de [[00 - Índice Notificaciones]].

Base del front: `environment.apiUrl` = `http://localhost:5091/api`.

## Módulo Notificaciones (`/api/notificaciones`)

Archivo: `Features/Notificaciones/Presentation/Endpoint/NotificacionesEndpoints.cs`

### `GET /api/notificaciones/pendientes`

| | |
|---|---|
| Permiso | `NOTIFICACIONES_EJECUTAR` |
| Consume | `IConsultaHorasFaltantes.ObtenerPendientesAsync()` |
| Devuelve | `{ inicioPeriodo, finPeriodo, empleados: [{ idEmpleado, nombre, email, horasEsperadas, horasRegistradas, horasFaltantes, ultimoAvisoManual }] }` |
| Usa eventos | No: calcula en el momento |
| Lo llama el front | **No** (`getPendientes()` existe, con la interfaz ya corregida, pero ninguna pantalla lo usa) |

### `POST /api/notificaciones/notificar`

| | |
|---|---|
| Permiso | `NOTIFICACIONES_EJECUTAR` |
| Body | `{ "empleadoIds": [1, 2, 3] }` |
| Consume | `IEventBus`, `ICurrentUserService` (id del líder) |
| Hace | Publica `NotificacionManualSolicitada` con el período en curso |
| Devuelve | **`202 Accepted`** `{ mensaje: "Solicitud recibida. Los correos se procesan en segundo plano.", solicitados: N, solicitadoEnMs }` · `400` si la lista está vacía · `403` sin permiso |
| Lo llama el front | `seguimiento.ts → notificarColaborador()` |

*Antes del 15/10 respondía `200 "Notificaciones enviadas exitosamente"` aunque todavía no se había enviado nada.*

### `GET /api/notificaciones/estado?empleadoIds=1&empleadoIds=2&desdeMs=...` *(nuevo, Allan)*

| | |
|---|---|
| Permiso | `NOTIFICACIONES_EJECUTAR` |
| Query | `empleadoIds` (1 a 200, se repite el parámetro) y `desdeMs` (el `solicitadoEnMs` del POST) |
| Consume | `ApplicationDbContext` (`notificacion_envio`), `ICurrentUserService` |
| Hace | Busca los avisos **manuales del usuario actual** desde `desdeMs` y devuelve el último estado de cada empleado |
| Devuelve | `200 { completo, empleados: [{ idEmpleado, estado: "Pendiente" \| "Enviado" \| "Fallido" \| "Omitido", detalle }] }` |
| Lo llama el front | `seguimiento.ts → esperarResultadoNotificacion()` (cada 2 s, hasta 10 veces) |

### El permiso `NOTIFICACIONES_EJECUTAR`

- Se crea con `tmr-backend/Documentacion/Scripts_DB/14-Inv_tmr_db_permisos_notificaciones.sql`, asociado a los roles **LÍDER** y **ADMINISTRADOR**.
- `PermissionPolicyProvider` convierte cualquier nombre en MAYÚSCULAS_CON_GUIONES en una policy que exige el claim `permission = <nombre>` en el JWT.

## Endpoints del Dashboard relacionados (fuera del módulo)

Archivo: `Features/Dashboard/Endpoints/DashboardEndpoints.cs`

### `GET /api/dashboard/mis-horas-incompletas?rango=mes`

- Requiere solo estar autenticado.
- Calcula las horas faltantes **del usuario logueado**.
- Devuelve `{ tieneFaltantes, horasFaltantes, diasIncompletos }`.
- Lo usa la **campana del navbar**.

### `POST /api/dashboard/notificar-faltantes-email`

- Requiere solo estar autenticado (**no** pide `NOTIFICACIONES_EJECUTAR`).
- Body `{ idEmpleado, nombreCompleto, proyecto, horasFaltantes }`. Las horas **las manda el front**.
- Envía **directo por SMTP** (`IEmailService`), dentro de la misma petición.
- **No** usa eventos, **no** guarda historial, **no** aplica umbral, **no** usa Apprise.
- Lo usan `dashboard-ejecutivo` y `dashboard-page`.

Ver [[13 - Otros avisos fuera del módulo]].

## Flujo 3 (diseñado, no implementado)

Según la propuesta, irían `FechasCorteEndpoints` → `POST /fechas-corte` con el permiso `NOTIFICACIONES_CONFIGURAR`. **No existen** en el código todavía.
