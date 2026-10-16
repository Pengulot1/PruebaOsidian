# Eventos y handlers

Parte de [[00 - Índice Notificaciones]].

Eventos en `Features/Notificaciones/Domain/Events/`. Todos son `record` que implementan `IEvento`.

## Los 6 eventos

| Evento | Lo publica | Datos | Lo escucha |
|---|---|---|---|
| `CorteDeHorasAlcanzado` | `RevisarHorasFaltantesJob` (flujo 1) | `InicioPeriodo`, `FinPeriodo`, `FechaCorte` | `DetectarHorasFaltantesHandler` |
| `NotificacionManualSolicitada` | `POST /api/notificaciones/notificar` (flujo 2) | `IdsEmpleados`, `InicioPeriodo`, `FinPeriodo`, `IdUsuarioEjecutor` | `DetectarHorasFaltantesHandler` |
| `HorasFaltantesDetectadas` | `DetectarHorasFaltantesHandler` (uno por empleado) | `IdEmpleado`, `Nombre`, `Email`, `HorasFaltantes`, período, `Origen`, `FechaCorte?`, `IdUsuarioEjecutor?` | `EnviarCorreoHandler` |
| `NotificacionEnviada` | `EnviarCorreoHandler` | lo mismo + `FechaEnvio` | `RegistrarEnvioHandler` |
| `NotificacionFallida` | `EnviarCorreoHandler` | lo mismo + `FechaEnvio` + `Error` | `RegistrarEnvioHandler` |
| `NotificacionOmitida` *(Allan, 15/10)* | `DetectarHorasFaltantesHandler` y `EnviarCorreoHandler` — **solo avisos manuales** | `IdEmpleado`, período, `IdUsuarioEjecutor`, `FechaEnvio`, `Motivo` | `RegistrarEnvioHandler` |

`NotificacionOmitida` existe para que el líder sepa **por qué** no se envió un aviso manual. Motivos posibles:
- "No tiene horas pendientes en el período"
- "Ya recibió un aviso manual hace menos de N h"
- "No tiene correo registrado"

## Enums del dominio

- `OrigenNotificacion`: `Automatico = 1` (flujos 1 y 3), `Manual = 2` (flujo 2).
- `EstadoEnvio`: `Enviado = 1`, `Fallido = 2`, `Omitido = 3` *(nuevo)*.

En la BD se guardan como **texto** (`"Automatico"`, `"Enviado"`), por `HasConversion<string>()`.

## Registro en DI — `NotificacionesModule.cs`

```csharp
services.Configure<NotificacionHorasOptions>(configuration.GetSection("NotificacionHoras"));
services.Configure<AppriseOptions>(configuration.GetSection("Apprise"));
if (configuration.GetValue<bool>("Apprise:Enabled") && !string.IsNullOrWhiteSpace(configuration["Apprise:ApiUrl"]))
    services.AddHttpClient<IAppriseNotificationService, AppriseNotificationService>(...);

services.AddScoped<CalculadorHorasFaltantes>();
services.AddScoped<IConsultaHorasFaltantes, ConsultaHorasFaltantes>();

services.AddScoped<IEventHandler<CorteDeHorasAlcanzado>,       DetectarHorasFaltantesHandler>();
services.AddScoped<IEventHandler<NotificacionManualSolicitada>, DetectarHorasFaltantesHandler>();
services.AddScoped<IEventHandler<HorasFaltantesDetectadas>,     EnviarCorreoHandler>();
services.AddScoped<IEventHandler<NotificacionEnviada>,          RegistrarEnvioHandler>();
services.AddScoped<IEventHandler<NotificacionFallida>,          RegistrarEnvioHandler>();
services.AddScoped<IEventHandler<NotificacionOmitida>,          RegistrarEnvioHandler>();
```

Este es el "mapa" de quién escucha qué. Si un evento no tiene handler registrado, el dispatcher lo marca como procesado y deja un warning en el log.

## Orden en `Program.cs`

```csharp
builder.Services.AddMessaging();                                   // bus + dispatcher
builder.Services.AddNotificacionesModule(builder.Configuration);   // handlers, consulta, Apprise
builder.Services.AddNotificacionesQuartz(builder.Configuration);   // Quartz + job + trigger
...
app.MapNotificacionesEndpoints();                                  // /api/notificaciones/*
```

## Los 3 handlers, en una línea

- **`DetectarHorasFaltantesHandler`** (Allan): recibe el evento de entrada, calcula quién debe horas y publica un aviso por empleado. → [[06 - Cálculo de horas faltantes]]
- **`EnviarCorreoHandler`** (Harold): anti-duplicados, plantilla y envío. → [[07 - Envío del correo (EnviarCorreoHandler)]]
- **`RegistrarEnvioHandler`** (Roberto): inserta una fila en `notificacion_envio` por cada intento (`Enviado`, `Fallido` u `Omitido`). → [[11 - Base de datos]]
