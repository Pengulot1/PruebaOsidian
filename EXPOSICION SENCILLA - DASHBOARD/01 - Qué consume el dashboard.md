# 01 - Qué consume el dashboard

Vuelve al [[00 - Guion corto]].

> [!info] "Consumir" un endpoint
> Significa que la app **le pide algo a la API** por una dirección, como `GET /api/cuentas`, y la API le **responde** con los datos en formato JSON.

---

## Lo que se ve y de dónde sale

```
┌─────────────────────────────┐
│ Hola, Andrea                │ ← del login (ya lo tenía la app)
│ ┌─────────────────────────┐ │
│ │ Saldo Total     👁      │ │ ← 1️⃣ GET /api/cuentas
│ │ $200.00                 │ │
│ │ Cuenta de ahorro        │ │
│ │ Cuenta corriente        │ │
│ └─────────────────────────┘ │
│ [Transferir] [Efectivo] [Pagar] [Recibir] │
│ Mis Tarjetas                │ ← 2️⃣ GET /api/tarjetas
│ [VISA •••• 4821]            │
│ Actividad reciente          │ ← 3️⃣ GET /api/movimientos?limite=5
│ McDonalds        -$4.50     │
└─────────────────────────────┘
```

| # | Qué pide | Endpoint | Qué le llega | Dónde se ve |
|---|---|---|---|---|
| 1️⃣ | Mis cuentas | `GET /api/cuentas` | Número, tipo (ahorros/corriente) y **saldo** de cada cuenta | Tarjeta negra de arriba |
| 2️⃣ | Mis tarjetas | `GET /api/tarjetas` | Marca, últimos 4 dígitos, vencimiento | "Mis Tarjetas" |
| 3️⃣ | Mis movimientos | `GET /api/movimientos?limite=5` | Los **5 últimos** (descripción, monto, fecha, si entra o sale) | "Actividad reciente" |

**Las 3 se piden al mismo tiempo** cuando se abre el inicio, para que cargue más rápido.

> [!note] ¿Y el "Saldo Total"?
> La API **no** lo manda. La app **suma** los saldos de todas las cuentas: $100 (ahorro) + $100 (corriente) = **$200**.

> [!note] ¿Cómo sabe la API que soy Andrea?
> Al iniciar sesión la API da un **token** (como una pulsera de entrada). La app lo manda en cada pedido y la API **solo devuelve los datos de esa persona**.

---

## Otras cosas del inicio

| Qué toco | Qué pasa | ¿Pide algo a la API? |
|---|---|---|
| 👁 **Ojito** | Oculta el saldo y los números de cuenta (`•••• 0001`) | No, es solo visual |
| **Ver más** de Actividad reciente | Abre "Movimientos" con los **50** últimos | Sí: `GET /api/movimientos?limite=50` |
| **Ver más** de Mis Tarjetas / **Tarjetas** abajo | Abre "Mis tarjetas" con el número completo | Sí: `GET /api/tarjetas` |
| **Tu cuenta** abajo | Abre el perfil | (otra parte del proyecto) |
| Arrastrar hacia abajo | Recarga todo | Sí: los 3 de arriba otra vez |

---

## Las carpetas del dashboard (app)

```
lib/features/dashboard/
├── model/          ← cómo son los datos: cuenta, tarjeta, movimiento
├── data/           ← quién le pide los datos a la API
├── viewmodel/      ← el "cerebro": guarda los datos y avisa a la pantalla
└── view/           ← la pantalla y sus partes (saldo, botones, tarjetas…)
```
