# 04 - Chuleta

Vuelve al [[00 - Guion corto]]. Todo en una página para tener a mano.

## Los endpoints del dashboard

| Endpoint | Para qué |
|---|---|
| `GET /api/cuentas` | Saldo y cuentas |
| `GET /api/tarjetas` | Tarjetas |
| `GET /api/movimientos?limite=5` | Actividad reciente (50 en "Ver más") |
| `GET /api/bancos` · `GET /api/beneficiarios` | Para transferir |
| `POST /api/transferencias` | **Transferir** |
| `POST /api/efectivo-movil` | **Generar los códigos** |

## Los archivos clave

| Qué | Archivo |
|---|---|
| Pedir los 3 datos del inicio | `dashboard/data/repositories/dashboard_repository.dart` |
| Sumar el saldo total | `dashboard/model/resumen_dashboard.dart` |
| Cerebro del inicio | `dashboard/viewmodel/dashboard_view_model.dart` |
| Botones y recarga | `dashboard/view/dashboard_screen.dart` |
| Cerebro de Transferir | `transfers/viewmodel/transfer_view_model.dart` |
| Cerebro de Efectivo móvil | `mobile_cash/viewmodel/mobile_cash_view_model.dart` |
| Pagar servicios | `services/view/pay_services_screen.dart` |
| Transferencia en la API | `Features/Transfers/TransfersService.cs` |
| Efectivo móvil en la API | `Features/MobileCash/MobileCashService.cs` |

## 5 palabras que vas a usar

- **Endpoint**: la "dirección" de la API a la que la app le pide algo.
- **Token**: la "pulsera" que da el login; con ella la API sabe quién eres.
- **ViewModel**: el cerebro de cada pantalla.
- **`notifyListeners()`**: "¡pantalla, redibújate!".
- **Transacción**: guardar todo junto o nada.

## Preguntas rápidas

| Si te preguntan… | Respondes… |
|---|---|
| ¿De dónde sale el saldo total? | La app suma el saldo de cada cuenta que manda `GET /api/cuentas`. |
| ¿Puedo ver datos de otro usuario? | No, la API filtra todo por el token. |
| ¿Qué pasa si no alcanza el saldo? | La API responde "Saldo insuficiente" y no mueve nada. |
| ¿Y si dos transferencias llegan juntas? | La cuenta se bloquea; la segunda espera. |
| ¿Pagar servicios descuenta dinero? | Todavía no, es visual. |
| ¿Por qué se recarga el inicio al volver? | Para mostrar el saldo nuevo después de transferir o generar códigos. |
| ¿Dónde se guardan los códigos de efectivo móvil? | En la base, **cifrados**; se muestran una sola vez. |

## Usuario para la demo

`andrea.demo` / `Panda2026` → 2 cuentas de $100. Frecuente para transferir: **Carlos Mendoza** (Banco Panda).
