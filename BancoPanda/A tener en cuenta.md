# A tener en cuenta

Parte de [[BancoPanda]]. Límites y detalles que no rompen nada, pero conviene saber antes de la entrega.

## Para la demo del APK

- **Celular real.** La API solo escucha en `localhost`, así que un teléfono físico no la alcanza (el emulador sí, por `10.0.2.2`). Para probar el APK en un celular:
  1. Arrancar la API con `dotnet run --urls http://0.0.0.0:5258`.
  2. Permitir el puerto 5258 en el firewall de Windows.
  3. En la app, engranaje del login → `http://IP-de-la-PC:5258` (PC y celular en la misma red Wi-Fi).
- El APK ya tiene permiso de internet y HTTP sin cifrar habilitados en `AndroidManifest.xml`.
- **Sesión de 60 minutos.** El token vence a la hora y no hay renovación. Con la app abierta mucho rato, al recargar manda al login. Se cambia en `appsettings.json` → `Jwt:MinutosExpiracion`.
- **Memoria del emulador.** Con poca RAM libre el emulador muestra "el sistema no responde" al arrancar. Elegir "Esperar" y cerrar programas que no se usen. La primera compilación de Android tarda varios minutos.

## Límites de lo construido

- **Máximo 50 movimientos.** La pantalla de Movimientos muestra los 50 más recientes; no hay paginación.
- **Saldo total en una sola moneda.** Suma las cuentas asumiendo que todas son USD.
- **Movimientos por tarjeta = movimientos de su cuenta.** Dos tarjetas de la misma cuenta muestran lo mismo. Ver [[Backend - cuentas, tarjetas y movimientos]].
- **El API no tiene tests automatizados.** Los endpoints se probaron a mano; los 43 tests son solo de la app.
- **Sin probar en Android.** La app se revisó en web contra la API local. Falta recorrerla en el emulador: dashboard, tocar una tarjeta y entrar con Carlos para ver la tarjeta inactiva.

## Cuidado con el seed

- Volver a ejecutarlo **reinicia a Andrea y Carlos**: se pierde cualquier cambio hecho a sus perfiles desde la app.
- En Visual Studio el archivo `seed.sql` aparece lleno de subrayados rojos. Son falsos: ese editor valida SQL Server y el script es de PostgreSQL. No ejecutarlo con el botón ▶ de Visual Studio; usar pgAdmin, DBeaver o `psql`. Ver [[Datos de prueba (seed SQL)]].

## Sigue en "Próximamente"

- Pestaña Tarjetas, accesos rápidos, notificaciones y "Ver más" de Mis Tarjetas.
- El `README.md` de la app sigue siendo la plantilla de Flutter; el taller pide documentar cómo ejecutarla.

## Respaldo del chat

La conversación completa se exportó el 13 oct 2026 a `Descargas\session-export-1791894989330.zip`. Incluye rutas de la PC, capturas y la contraseña de los usuarios de prueba; revisarlo antes de compartirlo.
