# TMR - Notificaciones de horas faltantes

Módulo que avisa por correo a los colaboradores que **no registraron todas sus horas** en una quincena.

- Código: `D:\TMR\tmr2`
  - `tmr-backend` → .NET (Minimal APIs + EF Core + Postgres)
  - `tmr-frontend` → Angular
- Propuesta de arquitectura: `D:\TMR\tmr2\Propuesta_Arquitectura_Notificacion.pdf`
- Reparto de trabajo: `D:\TMR\tmr2\Reparto_Trabajo_Notificaciones.docx`
- Mi parte: **Harold — Envío y flujo 1** → [[14 - Mi parte (Harold)]]

## Orden de lectura recomendado

1. [[01 - Cómo funciona (visión general)]] — la idea en 5 minutos.
2. [[02 - Bus de eventos y Outbox]] — el "motor" que mueve todo.
3. [[03 - Eventos y handlers]] — quién publica qué y quién lo escucha.
4. [[04 - Flujo 1 - Corte automático con Quartz]] — el aviso automático de quincena.
5. [[05 - Flujo 2 - Aviso manual del líder]] — el botón 🔔 de Seguimiento.
6. [[06 - Cálculo de horas faltantes]] — cómo se sabe cuántas horas debe alguien.
7. [[07 - Envío del correo (EnviarCorreoHandler)]] — plantilla, anti-duplicados y envío.
8. [[08 - Apprise]] — qué es y cómo se usa aquí.
9. [[09 - Endpoints]] — lo que el front puede llamar.
10. [[10 - Frontend]] — pantallas y servicios Angular.
11. [[11 - Base de datos]] — las 3 tablas nuevas.
12. [[12 - Configuración (appsettings)]] — todas las claves que afectan al módulo.
13. [[13 - Otros avisos fuera del módulo]] — Dashboard, campana y aviso semanal.
14. [[14 - Mi parte (Harold)]] — lo que me tocó, función por función.
15. [[15 - Riesgos y pendientes]] — cosas a revisar.
16. [[16 - Pruebas automatizadas]] — qué tests existen.
17. [[17 - Presentación - Cambios en Quartz]] — antes vs. después, decisiones, preguntas y guion de 1 minuto.
18. [[18 - Handlers de notificaciones paso a paso]] — los 3 handlers, qué consume cada clase y su paso a paso.

## Resumen en una tabla

| Pieza | Dónde | Qué hace |
|---|---|---|
| `RevisarHorasFaltantesJob` | `Features/Notificaciones/Infrastructure/Jobs/` | Quartz lo ejecuta en la fecha de corte; publica `CorteDeHorasAlcanzado` |
| `POST /api/notificaciones/notificar` | `Presentation/Endpoint/NotificacionesEndpoints.cs` | El líder pide avisar a empleados; publica `NotificacionManualSolicitada` y responde 202 |
| `GET /api/notificaciones/estado` | `Presentation/Endpoint/NotificacionesEndpoints.cs` | El front consulta si el aviso quedó Enviado / Fallido / Omitido |
| `EventBus` + `EventDispatcher` | `Infrastructure/Messaging/` | Guardan el evento en Postgres y lo reparten a los handlers en segundo plano |
| `DetectarHorasFaltantesHandler` | `Application/Handlers/` | Calcula quién debe horas y publica un aviso por empleado |
| `EnviarCorreoHandler` | `Application/Handlers/` | Evita duplicados, arma el correo y lo manda (Apprise o SMTP) |
| `RegistrarEnvioHandler` | `Application/Handlers/` | Guarda el resultado en la tabla `notificacion_envio` |
| `AppriseNotificationService` | `Infrastructure/Apprise/` | Cliente HTTP hacia el servidor Apprise |

> [!info] Últimos cambios (15–16 oct 2026)
> - **Harold** (`b2092f6`): el corte automático se dispara solo el día 15 y el último día de cada mes → [[04 - Flujo 1 - Corte automático con Quartz]].
> - **Allan** (`cc97a02` en el back, `2541160` en el front): estado `Omitido`, `GET /estado`, respuesta 202 y toast en Seguimiento; arreglos de Apprise → [[05 - Flujo 2 - Aviso manual del líder]], [[08 - Apprise]].

> [!info] Estado del alcance
> Flujos 1 y 2 implementados. El flujo 3 (Contabilidad programa los cortes desde una pantalla) está diseñado y su tabla existe, pero **no hay código** que lo use todavía.
