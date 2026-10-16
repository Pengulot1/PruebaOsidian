# Configuración (appsettings)

Parte de [[00 - Índice Notificaciones]].

Archivos: `tmr-backend/appsettings.json` (base) y `appsettings.Development.json` (lo **sobrescribe** en dev).

## `NotificacionHoras` → `NotificacionHorasOptions`

| Clave | Base | Dev | Usada por |
|---|---|---|---|
| `Habilitado` | `true` | (hereda) | `QuartzExtensions` (programar los cron), `RevisarHorasFaltantesJob` |
| `InicioPeriodo` | `2026-10-01` | (hereda) | **ya no se usa** (desde el 16/10) |
| `FinPeriodo` | `2026-10-15` | (hereda) | **ya no se usa** |
| `FechaCorte` | `2026-10-15T18:00:00-05:00` | (hereda) | **ya no se usa** |
| `UmbralNotificacionManual` | `24` (horas) | `0` | Detector y `EnviarCorreoHandler` (flujo 2) |

> [!tip] Ya no hay que cambiar el corte a mano (desde el 15/10/2026)
> Los cortes son cron fijos en el código (día 15 y último día, 18:00 Ecuador). `InicioPeriodo`, `FinPeriodo` y `FechaCorte` **ya no se usan**; solo importan `Habilitado` y `UmbralNotificacionManual`. Revisar en el log: `Cortes de horas programados ... Próximo corte: ...`. Ver [[04 - Flujo 1 - Corte automático con Quartz]].

## `Apprise` → `AppriseOptions`

`Enabled`, `ApiUrl`, `Tag`, `UrlTemplates`, `Username`, `Password`, `TimeoutSeconds`. Ver [[08 - Apprise]].

- Base: `Enabled = false` → el correo sale por SMTP.
- Dev: `Enabled = true` → sale por Apprise hacia el MailDev de Dokploy.

## `EmailSettings` (SMTP, `EmailService`)

`SmtpServer` (gmail por defecto), `Port` 587, `SenderName`, `SenderEmail`, `Username`, `Password`, `OverrideEmailRecipient`, `FrontendUrl`.

- `OverrideEmailRecipient`: si tiene valor, **todos** los correos SMTP van a esa dirección, con el asunto `[PRUEBA para: real@...]`.
- `FrontendUrl`: base del botón "Registrar mis horas" del correo. Hoy vale `http://localhost:3000` → hay que ajustarlo por ambiente.

> [!warning] Secretos
> Las contraseñas reales (`EmailSettings:Password`, `Apprise:Password`) no deberían subirse al repositorio. Es mejor ponerlas en variables de entorno o en user-secrets.

## `WeeklyNotificationSettings` (aviso semanal, fuera del módulo)

`Enabled = false`, `DayOfWeek = Friday`, `Hour = 17`, `Minute = 0`, `CheckIntervalMinutes = 5`. Ver [[13 - Otros avisos fuera del módulo]].
