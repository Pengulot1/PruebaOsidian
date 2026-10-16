# Riesgos y pendientes

Parte de [[00 - Índice Notificaciones]]. Revisión del código al **15 oct 2026**. No se cambió nada en el código.

## De mi parte (envío y flujo 1)

### 1. Reintento automático después de un `Fallido` → choca con el índice único
- El índice `uq_idx_notificacion_envio_auto (id_empleado, fecha_corte) WHERE origen = 'Automatico'` **también** cuenta las filas `Fallido`.
- Escenario: el corte falla para Juan (SMTP caído) → se guarda `Fallido`. El backend se reinicia dentro de las 24 h → Quartz vuelve a disparar (`FireNow`) → `DebeEnviar` solo mira los `Enviado`, así que **envía** → `RegistrarEnvioHandler` intenta insertar `Enviado` → **viola el índice** → el dispatcher reintenta 5 veces y lo marca `Fallido`.
- Resultado: Juan **sí** recibió el correo, pero queda registrado solo el fallo. Si hay otra corrida, lo recibe de nuevo.
- Los tests no lo detectan porque usan `UseInMemoryDatabase`, que no aplica índices filtrados.
- Posibles arreglos (coordinar con Roberto): filtrar el índice también por `estado = 'Enviado'`, o actualizar la fila `Fallido` en vez de insertar otra.

### 2. Logo roto cuando se envía por Apprise
`<img src='cid:logo_isc'>` solo funciona con el logo **adjunto**, y eso lo hace `EmailService`, no Apprise. → usar una URL pública del logo o base64.

### 3. ✅ Resuelto por Allan (15/10): `OverrideEmailRecipient` con Apprise
Ahora `EnviarCorreoHandler` también aplica la redirección de pruebas cuando envía por Apprise.

### 4. ✅ Resuelto por Allan (15/10): modo con estado de Apprise
Siempre se usa `/notify`. Sin `UrlTemplates`, cada envío falla con un mensaje claro y queda `Fallido`. Queda un detalle: con `Enabled = true` y `ApiUrl` pero sin plantillas, **todo** falla, y no se usa SMTP como respaldo.

### 5. ✅ Resuelto (15/10/2026): actualizar el corte cada quincena
Antes cada `appsettings.json` traía **un solo corte** y había que editarlo y reiniciar. Ahora hay triggers cron recurrentes (día 15 y último día, 18:00 Ecuador) + recuperación al arrancar. Ver [[04 - Flujo 1 - Corte automático con Quartz]].
Queda: si el backend está apagado **más de 24 h** durante un corte, ese corte se pierde (lo resolvería el JobStore persistente).

### 6. `FrontendUrl = http://localhost:3000`
El botón "Registrar mis horas" del correo apunta a localhost. Hay que configurarlo por ambiente.

### 7. Quartz en memoria
Al reiniciar se pierde el estado. Pendiente: JobStore en Postgres + script de las tablas `QRTZ_*` (era tarea de BD).

## Del flujo 2 / front

- ✅ **Éxito engañoso — resuelto por Allan (15/10):** `POST /notificar` responde `202 "Solicitud recibida"`, y el front consulta `GET /estado` hasta saber si quedó `Enviado`, `Fallido` u `Omitido` (con el motivo).
- ✅ **`PendientesResponse`** del front ya coincide con el back.
- **Polling de ~20 s:** si el envío tarda más (SMTP lento, reintentos del dispatcher cada 30 s), el toast dice "sigue en cola" aunque luego se envíe.
- **Motivo "No tiene horas pendientes"** también aparece para un empleado **inactivo** o un id inexistente, porque `CalcularAsync` solo trae activos. Es un detalle de texto.
- **Botón sin control de permiso** en Seguimiento: sin `NOTIFICACIONES_EJECUTAR` da 403, y el toast dice "No se pudo registrar la solicitud".
- `GET /pendientes` no se usa. No hay pantalla con selección múltiple (RF-003/RF-004 a medias en la UI).

## Generales

- **Dos (tres) sistemas de correo en paralelo:** el módulo, el endpoint del Dashboard (sin permiso específico, sin historial ni umbral) y el aviso semanal (apagado). Ver [[13 - Otros avisos fuera del módulo]].
- **La migración `20261012193639_notificacion`**, además de crear el esquema `notificacion`, hace `DropTable` de 13 tablas existentes (`tbl_autenticacion_menu`, `tbl_autenticacion_menu_rol`, `Usuarios`, `ConfiguracionesSistema`, etc.). Revisarla con Roberto **antes** de aplicarla en un ambiente con datos.
- **Outbox sin depuración:** los eventos `Procesado` se acumulan para siempre (la propuesta sugiere depurarlos periódicamente).
- **Eventos `Fallido` en el Outbox:** no hay pantalla ni alerta, solo la consulta SQL de [[11 - Base de datos]].
- **Flujo 3** pendiente: endpoints de cortes, permiso `NOTIFICACIONES_CONFIGURAR`, pantalla de Contabilidad, job cada hora leyendo `notificacion_fecha_corte`.
