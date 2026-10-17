# 05 - Demo en vivo

Vuelve al [[00 - Guion de la exposición]].

> [!warning] Preparación (5 minutos antes)
> 1. Base: ejecutar `Database/seed.sql` en pgAdmin → `andrea.demo` y `carlos.demo` con **$100** en cada cuenta.
> 2. API: en `bancopanda-api/BancoPanda.Api` → `dotnet run` (o confirmar que Render responde).
> 3. App: `flutter run -d windows` (o emulador). En el login, ⚙ → `http://localhost:5258` (emulador: `http://10.0.2.2:5258`).
> 4. Dejar la app en el **login**.

**Datos de prueba**

| Usuario | Contraseña | Cuentas | Celular |
|---|---|---|---|
| `andrea.demo` | `Panda2026` | `5000900001` (ahorros), `5000900002` (corriente) | +593 987654321 |
| `carlos.demo` | `Panda2026` | `5000900003` (ahorros), cédula `0923456781` | +593 991234567 |

---

## Guion paso a paso (~6 min)

| # | Qué hago | Qué digo |
|---|---|---|
| 1 | Login con `andrea.demo` / `Panda2026` | "Al iniciar sesión el API devuelve un token que la app guarda cifrado." |
| 2 | Muestro el inicio | "El inicio pide en paralelo cuentas, tarjetas y movimientos. El total ($200) lo suma la app." |
| 3 | Toco el **ojito** | "Oculta el saldo y los números de cuenta; es solo visual." |
| 4 | **Transferir** → ahorros → frecuente **Carlos Mendoza** → `10` → Confirmar → **Transferir** | "Transferencia dentro de Banco Panda: el API bloquea las dos cuentas, resta, suma y guarda todo en una transacción." |
| 5 | Muestro el **comprobante** → **Volver al inicio** | "Al volver, el inicio se recarga: ahora $190 y aparece la transferencia en actividad reciente." |
| 6 | (Opcional) Transferir con **nuevo destinatario**: Banco Pichincha, cuenta `2201998877`, cédula `0912121212`, `5` | "A otros bancos solo se descuenta: se simula el envío. El beneficiario queda guardado como frecuente." |
| 7 | **Efectivo móvil** → `20` → **Mi número** → Generar código | "Ya no hay que escribir el celular: se usa el registrado. El API genera dos códigos de 4 dígitos y los guarda cifrados." |
| 8 | Muestro la pantalla de **códigos** → **Finalizar** | "Estos son los pasos que se harían en el cajero. Los códigos se muestran una sola vez." |
| 9 | **Pagar servicios** → **Luz eléctrica** | "Es un prototipo visual con agua, luz e internet; el siguiente paso sería conectarlo al API." |
| 10 | **Recibir dinero** | "Funcionalidad marcada como próximamente." |
| 11 | Barra → **Tarjetas** → **Agregar tarjeta** | "Aquí se ven todos los datos de la tarjeta; agregar una nueva no está habilitado." |
| 12 | (Opcional) Cerrar sesión y entrar con `carlos.demo` | "A Carlos le llegó la transferencia: 'Transferencia recibida de Andrea Barroso'." |

---

## Si algo falla durante la demo

| Problema | Qué hacer |
|---|---|
| El login tarda mucho | Si usas Render, la primera llamada despierta el servidor (~1 min). Con la API local no pasa. |
| "No se pudo conectar con el servidor" | Revisar que `dotnet run` siga abierto y la dirección del ⚙. |
| Saldos raros por pruebas anteriores | Volver a ejecutar `seed.sql` (deja todo en $100). |
| "Saldo insuficiente" | Es correcto si el monto supera el saldo: sirve para mostrar la validación. |

## Plan B: probar la API sin la app

Abrir `BancoPanda.Api/BancoPanda.Api.http` y ejecutar en orden: **Login** → **Mis cuentas** → **Transferir a Carlos** → **Efectivo móvil con "Mi número"**. Se ven las respuestas JSON reales.
