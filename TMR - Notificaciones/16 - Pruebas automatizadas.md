# Pruebas automatizadas

Parte de [[00 - Índice Notificaciones]].

Carpeta: `tmr-backend/tests/tmr-backend.Tests/Notificaciones/` (xUnit). Usan `UseInMemoryDatabase` y dobles (fakes) del bus y del servicio de correo.

| Archivo | Qué prueba | Dueño |
|---|---|---|
| `CalculadorHorasFaltantesTests` | Regla de horas (fines de semana, feriados, jornada) | Allan |
| `ConsultaHorasFaltantesTests` | Consultas contra la BD en memoria | Allan |
| `DetectarHorasFaltantesHandlerTests` | Filtros del detector (ya notificados, umbral, sin correo) y, desde el 15/10, los `NotificacionOmitida` del flujo manual | Allan |
| `EnviarCorreoHandlerTests` | Ver abajo | **Harold** |
| `RevisarHorasFaltantesJobTests` | Ver abajo | **Harold** |
| `RegistrarEnvioHandlerTests` | Inserta `Enviado`/`Fallido`/`Omitido` con los campos correctos | Roberto (Allan agregó `Omitido`) |
| `EventDispatcherIntegrationTests` | Cadena completa con `AddMessaging()` real: flujo 1 desde `HorasFaltantesDetectadas` y flujo 2 desde `NotificacionManualSolicitada`, hasta la fila en `notificacion_envio` | Milena |

## `EnviarCorreoHandlerTests` (8)

1. `Envia_correo_y_publica_NotificacionEnviada_para_evento_automatico`
2. `Envia_correo_y_publica_NotificacionEnviada_para_evento_manual`
3. `Empleado_sin_email_publica_NotificacionFallida`
4. `Automatico_sin_FechaCorte_publica_NotificacionFallida`
5. `Automatico_ya_notificado_en_corte_no_se_vuelve_a_enviar`
6. `Manual_dentro_del_umbral_no_se_vuelve_a_enviar`
7. `Manual_fuera_del_umbral_si_se_envia`
8. `Fallo_en_EmailService_publica_NotificacionFallida`

## `RevisarHorasFaltantesJobTests` (7 métodos, 18 casos — desde el 16/10)

1. `Cuando_deshabilitado_no_publica_eventos`
2. `Corte_de_quincena_publica_el_periodo_del_1_al_15`
3. `Corte_de_fin_de_mes_publica_el_periodo_del_16_al_ultimo_dia`
4. `PeriodoDelCorte_usa_el_ultimo_dia_real_del_mes` (feb 28, feb 29 bisiesto, abril 30)
5. `UltimoCorte_devuelve_el_corte_mas_reciente` (5 casos, incluido el cambio de año)
6. `ProximoCorte_devuelve_el_siguiente_corte` (3 casos)
7. `Los_cron_se_disparan_el_15_y_el_ultimo_dia_a_las_18_de_Ecuador` (4 casos, con `CronExpression` real de Quartz)

Total de la carpeta: **75 casos en verde** (16/10/2026).

## Huecos

- **Sin tests de `AppriseNotificationService`** (endpoint con o sin estado, reemplazo de `{recipient}`, Basic Auth, error no 2xx). Se puede probar con un `HttpMessageHandler` falso.
- **Sin tests de `GET /estado`** (endpoint nuevo de Allan).
- El registro de triggers en `QuartzExtensions` no se prueba directamente; sí se prueban sus funciones de calendario y los cron.
- La BD en memoria **no aplica el índice único filtrado** → no detecta el caso de [[15 - Riesgos y pendientes]] #1. Para eso haría falta Postgres real (por ejemplo, Testcontainers).

## Ejecutar

```bash
dotnet test tmr-backend/tests/tmr-backend.Tests --filter "FullyQualifiedName~Notificaciones"
```
