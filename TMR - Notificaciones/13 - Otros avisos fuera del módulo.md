# Otros avisos fuera del módulo

Parte de [[00 - Índice Notificaciones]].

Existen **antes** del módulo nuevo (o en paralelo a él) y **no** usan eventos, Outbox ni la tabla `notificacion_envio`.

## 1. Correo desde el Dashboard

- Endpoint: `POST /api/dashboard/notificar-faltantes-email` (`DashboardEndpoints.cs`).
- Front: `dashboard-ejecutivo.enviarCorreo()` y `dashboard-page.enviarAlertaEmailServer()`.

Paso a paso:
1. Busca al empleado activo por `IdEmpleado` (400 si no existe).
2. Correo = `Emailcorporativo ?? Persona.Email` (400 si no hay).
3. Arma un HTML parecido ("Recordatorio de Horas Pendientes", proyecto, horas, 3 pasos para registrar).
4. `emailService.SendEmailAsync(...)` → **SMTP directo**, dentro del request.
5. `200 { mensaje: "Notificación enviada exitosamente" }`.

Diferencias con el módulo:

| | Módulo (`/notificaciones/notificar`) | Dashboard |
|---|---|---|
| Permiso | `NOTIFICACIONES_EJECUTAR` | cualquier usuario autenticado |
| Horas | las calcula el back | las manda el front |
| Historial | `notificacion_envio` | ninguno |
| Anti-spam | umbral | ninguno |
| Canal | Apprise o SMTP | solo SMTP |
| Respuesta | inmediata (asíncrono) | espera al envío |
| Si el SMTP falla | `Fallido` registrado | 500 al front |

## 2. Campana del navbar

Solo es visual. Ver [[10 - Frontend]].

## 3. Aviso semanal — `WeeklyNotificationService`

`Infrastructure/BackgroundServices/WeeklyNotificationService.cs`, registrado en `Program.cs` como `HostedService`.

1. Espera 1 minuto al arrancar.
2. Cada `CheckIntervalMinutes` (5) lee `WeeklyNotificationSettings`.
3. Si `Enabled` y es el día y la hora configurados (y todavía no corrió hoy) → `SendWeeklyNotificationsAsync`.
4. Recorre **todos** los empleados activos desde el día 1 del mes hasta hoy. Días con < 8 h (fijo, no considera pasantes) → lista de días incompletos.
5. Manda un correo SMTP con una tabla de días.

- **Está desactivado** (`Enabled = false`).
- Usa `DateTime.Now` (hora del servidor) y `UtcNow` mezclados, no `HoyEcuador()`.
- La regla de 8 h fija no coincide con `CalculoHorasPeriodo`.
- `_lastRunDate` vive en memoria: si se reinicia el backend a esa hora, podría enviar dos veces.

> [!question] ¿Unificar?
> Lo natural sería que el Dashboard llame a `POST /api/notificaciones/notificar` y que el aviso semanal se retire o pase a ser un trigger de Quartz que publique eventos.
