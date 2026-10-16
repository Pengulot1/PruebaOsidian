# Apprise

Parte de [[00 - Índice Notificaciones]]. Relacionado con [[07 - Envío del correo (EnviarCorreoHandler)]] y [[14 - Mi parte (Harold)]].

## ¿Qué es Apprise?

**Apprise** es una herramienta open source (Python) que manda notificaciones a **muchísimos servicios** con una sola interfaz: correo, Telegram, Slack, Discord, Teams, WhatsApp, SMS, etc.

La gracia es que **cada destino se describe con una URL**:

| Servicio | URL de ejemplo |
|---|---|
| Correo SMTP | `mailto://servidor:puerto?to=persona@x.com&from=yo@x.com` |
| Correo con TLS | `mailtos://usuario:clave@servidor?to=...` |
| Telegram | `tgram://token_bot/chat_id` |
| Slack | `slack://tokenA/tokenB/tokenC` |
| Discord | `discord://webhook_id/webhook_token` |

**Apprise API** es la versión **servidor web** (normalmente un contenedor Docker) que expone Apprise por HTTP. Así, cualquier backend, en cualquier lenguaje, puede mandar notificaciones con un simple `POST`.

Tiene dos modos:

| Modo | Endpoint | Qué se manda |
|---|---|---|
| **Sin estado** (*stateless*) | `POST /notify` | En el JSON van **las URLs de destino** + título + cuerpo |
| **Con estado** (*stateful*) | `POST /notify/{KEY}` | Los destinos ya están **guardados en el servidor** bajo una clave; solo se manda el mensaje |

## ¿Por qué se usa aquí?

- Desacopla el backend del proveedor de correo: el backend no necesita credenciales SMTP, solo la URL del servidor Apprise.
- El día de mañana se puede avisar también por Teams o Telegram **sin tocar código**: basta con agregar otra URL en la configuración.
- En el entorno de la empresa (Dokploy) ya hay un servidor Apprise desplegado.

## Cómo se conecta en el código

### 1. Configuración — `AppriseOptions` ← sección `Apprise`

| Clave | Para qué | Prod (`appsettings.json`) | Dev (`appsettings.Development.json`) |
|---|---|---|---|
| `Enabled` | Activa Apprise | `false` | `true` |
| `ApiUrl` | URL base del servidor Apprise | vacío | `https://apprise.dokploy.integritysolutions.com.ec` |
| `Tag` | *(ya no se usa desde el 15/10; antes era la clave del modo con estado)* | `notificaciones` | `notificaciones` |
| `UrlTemplates` | Lista de URLs destino; `{recipient}` se reemplaza por el correo | `[]` | `mailto://smtp-maildev....:1025?to={recipient}&from=soporte@...` |
| `Username` / `Password` | Basic Auth si el servidor la pide | vacío | — |
| `TimeoutSeconds` | Timeout del `HttpClient` | 30 | 30 |

> [!info] Dev apunta a un "MailDev"
> El destino de desarrollo es `smtp-maildev...:1025`. Por el nombre y el puerto (1025), parece un **MailDev**: un SMTP de pruebas que **atrapa** los correos y los muestra en una web, sin entregarlos a nadie. Así en dev no se molesta a los empleados reales.

### 2. Registro en DI — `NotificacionesModule`

```csharp
services.Configure<AppriseOptions>(configuration.GetSection("Apprise"));
// Sin ApiUrl no se registra: el handler cae a SMTP en vez de fallar en cada envío
if (configuration.GetValue<bool>("Apprise:Enabled")
    && !string.IsNullOrWhiteSpace(configuration["Apprise:ApiUrl"]))
{
    services.AddHttpClient<IAppriseNotificationService, AppriseNotificationService>((sp, client) =>
    {
        var options = sp.GetRequiredService<IOptions<AppriseOptions>>().Value;
        client.Timeout = TimeSpan.FromSeconds(Math.Max(1, options.TimeoutSeconds));
    });
}
```

- Es un **typed HttpClient**: .NET crea y reutiliza el `HttpClient` correctamente.
- Si `Enabled = false` **o** `ApiUrl` está vacío, **no se registra**. Entonces `EnviarCorreoHandler` recibe `null` (parámetro opcional `= null`) y usa SMTP.

### 3. Uso — `EnviarCorreoHandler`

```csharp
if (_appriseNotificationService is not null)
{
    var destinatario = evento.Email;
    var overrideEmail = _configuration["EmailSettings:OverrideEmailRecipient"];
    if (!string.IsNullOrWhiteSpace(overrideEmail))
    {
        asunto = $"[PRUEBA para: {destinatario}] {asunto}";
        destinatario = overrideEmail;          // modo pruebas, igual que en SMTP
    }
    await _appriseNotificationService.SendEmailAsync(destinatario, asunto, html, ct);
}
else
    await _emailService.SendEmailAsync(evento.Email, asunto, html);
```

### 4. El cliente — `AppriseNotificationService.SendEmailAsync(toEmail, subject, htmlBody, ct)`

1. Si `ApiUrl` está vacío → `InvalidOperationException`.
2. **`BuildEndpoint()`** → siempre `{ApiUrl}/notify` (modo **sin estado**). *Antes del 15/10, sin plantillas usaba `/notify/{Tag}`.*
3. **`BuildUrls(toEmail)`**: toma cada plantilla y reemplaza `{recipient}` por el correo (escapado con `Uri.EscapeDataString`). Si no hay plantillas, lanza `"Apprise no tiene destinos configurados. Configure Apprise:UrlTemplates."` → el envío queda `Fallido` con ese mensaje.
4. Arma el JSON (camelCase). `urls` va como **un solo string separado por comas**, que es el formato que acepta Apprise API:
   ```json
   {
     "urls":   "mailto://smtp-maildev...:1025?to=juan%40empresa.com&from=soporte@...",
     "body":   "<div>... HTML del correo ...</div>",
     "title":  "Recordatorio: tienes 16 horas pendientes...",
     "type":   "info",
     "format": "html"
   }
   ```
5. Header `Accept: application/json`. Si hay `Username`, agrega `Authorization: Basic base64(user:pass)`.
6. `POST` con el `HttpClient`.
7. Si la respuesta **no es 2xx** → log de error + `HttpRequestException("Apprise devolvió 4xx/5xx ... Detalle: ...")`.
8. Si es 2xx → log "Notificación enviada correctamente mediante Apprise".

Cualquier excepción (incluido el timeout) sube a `EnviarCorreoHandler`, que la convierte en `NotificacionFallida` con el mensaje → queda guardada en `notificacion_envio.detalle_error`.

```mermaid
sequenceDiagram
    participant H as EnviarCorreoHandler
    participant A as AppriseNotificationService
    participant API as Apprise API (Dokploy)
    participant M as Servidor de correo (MailDev / SMTP)
    participant E as Bandeja del empleado
    H->>A: SendEmailAsync(email, asunto, html)
    A->>API: POST /notify {urls, title, body, format: html}
    API->>M: SMTP (según la URL mailto://)
    M-->>E: correo
    API-->>A: 200 OK  /  4xx-5xx
    A-->>H: ok  /  HttpRequestException
```

## Cosas a tener en cuenta con Apprise

> [!success] Resuelto por Allan (15/10/2026)
> - **Modo con estado eliminado:** ya no se llama a `/notify/{Tag}`. Sin `UrlTemplates`, cada envío falla con un mensaje claro, en vez de mandar el correo a destinos que no son el empleado.
> - **`OverrideEmailRecipient` ya se respeta** con Apprise.
> - **Sin `ApiUrl` no se registra Apprise** → cae a SMTP.

> [!warning] El logo no aparece (sigue pendiente)
> El HTML usa `<img src='cid:logo_isc'>`, que solo funciona cuando el logo va **adjunto** al correo (lo hace `EmailService`). Por Apprise no se adjunta, así que la imagen sale rota. Opciones: usar una URL pública del logo o embeberlo en base64.

> [!warning] `Enabled = true` con `ApiUrl` pero sin `UrlTemplates`
> Apprise se registra, pero **cada** envío termina `Fallido` ("no tiene destinos configurados"). No hay respaldo automático a SMTP en ese caso.

- Solo `EnviarCorreoHandler` (flujos 1 y 2) usa Apprise. El correo del Dashboard y el semanal siguen por SMTP directo. Ver [[13 - Otros avisos fuera del módulo]].
- No hay tests automatizados de `AppriseNotificationService`. Ver [[16 - Pruebas automatizadas]].
