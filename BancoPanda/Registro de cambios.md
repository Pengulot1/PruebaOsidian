# Registro de cambios

Parte de [[BancoPanda]]. Lista completa de lo que se tocó, archivo por archivo, y cómo subirlo.

## 13 oct 2026 — Dashboard conectado al backend

### Repositorio `bancopanda-api` (rama `feature/dasboard2`)

**Archivos nuevos**

| Archivo | Para qué |
|---|---|
| `BancoPanda.Api/Common/ApiControllerBase.cs` | Lee el usuario del token y convierte `Resultado<T>` en respuesta HTTP |
| `BancoPanda.Api/Features/Accounts/AccountsController.cs` | `GET /api/cuentas` |
| `BancoPanda.Api/Features/Accounts/AccountsService.cs` | Consulta las cuentas del usuario |
| `BancoPanda.Api/Features/Accounts/AccountsDtos.cs` | `CuentaResponse` |
| `BancoPanda.Api/Features/Cards/CardsController.cs` | `GET /api/tarjetas` y `GET /api/tarjetas/{id}/movimientos` |
| `BancoPanda.Api/Features/Cards/CardsService.cs` | Consulta las tarjetas con el saldo de su cuenta |
| `BancoPanda.Api/Features/Cards/CardsDtos.cs` | `TarjetaResponse` |
| `BancoPanda.Api/Features/Movements/MovementsController.cs` | `GET /api/movimientos` |
| `BancoPanda.Api/Features/Movements/MovementsService.cs` | Movimientos recientes y por tarjeta |
| `BancoPanda.Api/Features/Movements/MovementsDtos.cs` | `MovimientoResponse` y `MovimientosRequest` (límite) |
| `Database/seed.sql` | Datos de prueba |

**Archivos modificados**

| Archivo | Cambio |
|---|---|
| `BancoPanda.Api/Program.cs` | Registra los 3 servicios nuevos; los enums salen como texto en el JSON |
| `BancoPanda.Api/BancoPanda.Api.http` | Peticiones de ejemplo de los endpoints nuevos |
| `README.md` | Estructura, paso 6 (datos de prueba), tabla de endpoints y convenciones |

**No se tocó:** entidades, `AppDbContext`, migraciones, `Features/Auth`, `Features/Users`.

Detalle en [[Backend - cuentas, tarjetas y movimientos]].

### Repositorio `bancopanda-app` (rama `feature/dashboard2`)

**Archivos nuevos en `lib/`**

| Archivo | Para qué |
|---|---|
| `core/utils/formatters.dart` | Formato de montos, fechas y vencimiento |
| `features/dashboard/model/cuenta.dart` | Modelo `Cuenta` |
| `features/dashboard/model/tarjeta.dart` | Modelo `Tarjeta` |
| `features/dashboard/model/movimiento.dart` | Modelo `Movimiento` y sus categorías |
| `features/dashboard/model/resumen_dashboard.dart` | Agrupa cuentas, tarjetas y movimientos; calcula el saldo total |
| `features/dashboard/data/datasources/dashboard_data_source.dart` | Interfaz: qué se le puede pedir al backend |
| `features/dashboard/data/datasources/remote_dashboard_data_source.dart` | Las llamadas HTTP reales |
| `features/dashboard/data/repositories/dashboard_repository.dart` | Pide el resumen en paralelo |
| `features/dashboard/viewmodel/dashboard_view_model.dart` | Estado del dashboard |
| `features/dashboard/viewmodel/movements_view_model.dart` | Estado de la pantalla de movimientos |
| `features/dashboard/view/movements_screen.dart` | Pantalla de movimientos (de una tarjeta o de todas las cuentas) |
| `features/dashboard/view/widgets/dashboard_header.dart` | Saludo y campana |
| `features/dashboard/view/widgets/balance_card.dart` | Saldo total y cuentas |
| `features/dashboard/view/widgets/quick_actions.dart` | Accesos rápidos |
| `features/dashboard/view/widgets/section_header.dart` | Título de sección con "Ver más" |
| `features/dashboard/view/widgets/cards_carousel.dart` | Carrusel de tarjetas |
| `features/dashboard/view/widgets/bank_card.dart` | Una tarjeta del carrusel |
| `features/dashboard/view/widgets/card_summary.dart` | Resumen de la tarjeta en la pantalla de movimientos |
| `features/dashboard/view/widgets/movimiento_tile.dart` | Fila de un movimiento |
| `features/dashboard/view/widgets/load_error.dart` | Aviso de error con "Reintentar" |
| `features/dashboard/view/widgets/dashboard_bottom_bar.dart` | Barra inferior |

**Archivos modificados en `lib/`**

| Archivo | Cambio |
|---|---|
| `core/network/api_client.dart` | Nuevo `getList` para respuestas que son listas |
| `core/router/app_router.dart` | Nueva ruta `/movimientos` |
| `core/di/providers.dart` | Registra `DashboardRepository` |
| `features/dashboard/view/dashboard_screen.dart` | Reemplaza el "en construcción" por el dashboard real |

**Tests (`test/`)**

| Archivo | Estado |
|---|---|
| `core/api_client_test.dart` | Modificado: 3 tests de `getList` |
| `core/formatters_test.dart` | Nuevo |
| `features/dashboard/dashboard_view_model_test.dart` | Nuevo |
| `features/dashboard/movements_view_model_test.dart` | Nuevo |
| `features/dashboard/remote_dashboard_data_source_test.dart` | Nuevo |
| `helpers/fake_auth.dart` | Nuevo: sesión iniciada para los tests |
| `helpers/fake_dashboard_data_source.dart` | Nuevo: backend falso para los tests |

**No se tocó:** `features/auth`, `features/profile`, `features/settings`, `pubspec.yaml`.

Detalle en [[Flutter - feature dashboard]].

### Base de datos

Se ejecutó `Database/seed.sql` en la base `bancopanda` local: 2 usuarios, 3 cuentas, 4 tarjetas, 34 movimientos. La estructura de las tablas no cambió. Ver [[Datos de prueba (seed SQL)]].

## Los archivos "generados" que salen como modificados

En la app aparecen con `M` siete archivos de `linux/`, `macos/` y `windows/` (`generated_plugin_registrant.*`, `generated_plugins.cmake`, `GeneratedPluginRegistrant.swift`).

- No son parte de este trabajo: ya estaban así antes de empezar.
- Su contenido es **idéntico** al último commit. Git los marca solo porque Flutter los reescribió con otro tipo de salto de línea.
- **No hay que borrarlos**: están versionados y Flutter los necesita para compilar en escritorio.
- Lo correcto es descartar el cambio (`git restore linux macos windows`) o simplemente no incluirlos en el commit. Si Flutter los vuelve a marcar, se repite.

## Cómo subirlo al repositorio

Ninguna de las dos ramas existe todavía en GitHub, por eso el primer `push` lleva `-u origin <rama>`.

### API

```bash
cd "D:\PROYECTOS DEL TRABAJO\codigo\bancopanda-api"
git status
git add .
git commit -m "Agregar endpoints de cuentas, tarjetas y movimientos; datos de prueba con seed.sql"
git push -u origin feature/dasboard2
```

La rama local se llama `feature/dasboard2` (le falta una "h"). Para corregirlo antes del primer push: `git branch -m feature/dashboard2` y luego `git push -u origin feature/dashboard2`.

### App

```bash
cd "D:\PROYECTOS DEL TRABAJO\codigo\bancopanda-app"
git restore linux macos windows
git add lib test
git commit -m "Conectar el dashboard al backend: cuentas, tarjetas y movimientos por tarjeta"
git push -u origin feature/dashboard2
```

### Después

En GitHub, abrir un Pull Request de cada rama hacia `develop`, como se hizo con `feature/profile`.

### Estas notas

La bóveda también es un repositorio (`PruebaOsidian`):

```bash
cd "D:\yuzu games\obsidiancarpeta\hola"
git add BancoPanda
git commit -m "Notas de BancoPanda: dashboard y movimientos de tarjetas"
git push
```

Solo sube la carpeta `BancoPanda`; los otros cambios pendientes de la bóveda quedan como están.
