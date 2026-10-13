# Backend - cuentas, tarjetas y movimientos

Parte de [[BancoPanda]]. Proyecto: `bancopanda-api/BancoPanda.Api`.

## Endpoints

Todos piden `Authorization: Bearer <token>` y solo devuelven datos del usuario del token.

| Método | Ruta | Devuelve |
|---|---|---|
| GET | `/api/cuentas` | Cuentas con su saldo |
| GET | `/api/tarjetas` | Tarjetas + saldo de la cuenta a la que pertenecen |
| GET | `/api/tarjetas/{tarjetaId}/movimientos?limite=20` | Movimientos de la cuenta de esa tarjeta |
| GET | `/api/movimientos?limite=20` | Últimos movimientos de todas las cuentas |

`limite`: opcional, 20 por defecto, entre 1 y 50. Fuera de rango → `400`.

Ejemplo de movimiento:

```json
{
  "movimientoId": "58f4319d-87f5-440e-9455-65653f7a5797",
  "descripcion": "McDonalds",
  "monto": 9.75,
  "tipo": "Egreso",
  "categoria": "Comida",
  "fecha": "2026-10-13T09:34:57.188362Z",
  "cuentaId": "c1000000-0000-4000-8000-000000000001"
}
```

## Archivos nuevos

```
Common/ApiControllerBase.cs
Features/
├── Accounts/   AccountsController · AccountsService · AccountsDtos
├── Cards/      CardsController    · CardsService    · CardsDtos
└── Movements/  MovementsController · MovementsService · MovementsDtos
Database/seed.sql
```

Cada feature repite el patrón que ya tenían `Auth` y `Users`: **Controller** (HTTP) → **Service** (consulta y reglas) → **Dtos** (lo que entra y sale).

## Cómo funciona cada pieza

### `ApiControllerBase`

Tres ayudas que todo controlador protegido necesita:

- `TryGetUsuarioId` lee el claim `sub` del JWT. El id del usuario **nunca** viene en la URL: así nadie puede pedir datos de otro.
- `SesionInvalida()` responde `401` si el token no trae un id válido.
- `ARespuesta(resultado)` convierte un `Resultado<T>` en `200` o en `ProblemDetails` con el código del error.

`UsersController` tiene estas mismas tres funciones como privadas. No se movieron para no tocar perfil; queda como limpieza futura.

### `MovementsService`

Aquí está la lógica importante:

```csharp
var cuentaId = await db.Tarjetas
    .Where(t => t.TarjetaId == tarjetaId && t.Cuenta.UsuarioId == usuarioId)
    .Select(t => (Guid?)t.CuentaId)
    .SingleOrDefaultAsync(ct);

if (cuentaId is null)
    return Resultado<List<MovimientoResponse>>.Fallo("Tarjeta no encontrada", 404);
```

- El `Where` pregunta dos cosas a la vez: que la tarjeta exista **y** que sea del usuario. Si es de otra persona responde lo mismo que si no existiera (`404`), para no revelar que ese id existe.
- `(Guid?)` permite distinguir "no encontrada" (`null`) de un id real.
- Después, `MasRecientesAsync` ordena por fecha descendente, corta con `Take(limite)` y proyecta al DTO. Lo usan los dos listados, así el orden y la forma del JSON son idénticos.

Solo `ListarPorTarjetaAsync` devuelve `Resultado<T>` porque es el único que puede fallar. Los demás listados devuelven la lista directa: una lista vacía es una respuesta válida, no un error.

### `CardsController`

Tiene las dos rutas que empiezan con `/api/tarjetas`. La de movimientos delega en `MovementsService`: las consultas de movimientos viven en un solo sitio.

### `MovimientosRequest`

```csharp
[Range(1, 50, ErrorMessage = "El límite debe estar entre 1 y 50")]
public int Limite { get; set; } = 20;
```

Con `[ApiController]` la validación corre sola antes de entrar al método. El tope de 50 evita que alguien pida `limite=1000000`.

### `Program.cs`

- Se registran los tres servicios con `AddScoped`.
- `JsonStringEnumConverter` hace que los enums salgan como texto (`"Egreso"`) y no como número (`1`). Va dos veces: una para los controladores y otra para que el documento OpenAPI lo describa igual.

## Decisiones

- **Sin migración.** `Movimiento` sigue colgando de `Cuenta`. Si dos tarjetas comparten cuenta verán los mismos movimientos. Para tener consumos propios por tarjeta habría que agregar `TarjetaId` a `Movimiento`.
- **`TarjetaResponse` incluye `saldo` y `moneda`** de su cuenta, porque el diseño muestra el saldo en cada tarjeta. Evita que la app tenga que cruzar tarjetas con cuentas.
- **Proyección con `Select`**: se trae de la base solo lo que va al JSON, no entidades completas.
- **Montos siempre positivos**; `tipo` dice si suma o resta. Así ya lo guardaba el bono de bienvenida.

## Probarlo

Con la API corriendo (`dotnet run`), abrir `BancoPanda.Api/BancoPanda.Api.http` y ejecutar "Login con correo" y luego las peticiones nuevas del final.

Comprobado contra la base real: `200` con datos, `404` con tarjeta ajena o inexistente, `400` con `limite=0`, `401` sin token, y perfil sigue respondiendo igual.

Datos para probar: [[Datos de prueba (seed SQL)]]. Quién consume esto: [[Flutter - feature dashboard]].
