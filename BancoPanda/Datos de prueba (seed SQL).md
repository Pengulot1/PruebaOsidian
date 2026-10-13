# Datos de prueba (seed SQL)

Parte de [[BancoPanda]]. Archivo: `bancopanda-api/Database/seed.sql`.

## Para qué

Un usuario recién registrado solo tiene el bono de bienvenida. El taller permite cargar registros de prueba **en la base de datos** (no listas fijas en Flutter) para demostrar cuentas y movimientos.

## Cómo ejecutarlo

Desde la raíz de `bancopanda-api`:

```bash
psql -U bancopanda_user -d bancopanda -f Database/seed.sql
```

O abrir el archivo en pgAdmin / DBeaver y ejecutarlo completo.

En esta PC `psql` no está en el PATH; está en `C:\Program Files\PostgreSQL\17\bin\psql.exe`.

## Qué crea

| Usuario | Cédula | Cuentas | Tarjetas | Movimientos |
|---|---|---|---|---|
| `andrea.barroso@bancopanda.test` | `0912345678` | Ahorros `5000900001` ($12,456.80) · Corriente `5000900002` ($2,283.61) | Visa Débito 4821 · Mastercard Débito 7730 | 18 + 8 |
| `carlos.mendoza@bancopanda.test` | `0923456781` | Ahorros `5000900003` ($1,095.85) | Visa Débito 1596 · Mastercard Crédito 3068 (inactiva) | 8 |

Contraseña de ambos: la misma de los ejemplos del archivo `.http` (está escrita en la cabecera del script y en el README del API).

## Cómo está hecho

- **Transacción** (`BEGIN` … `COMMIT`): si algo falla no queda nada a medias.
- **Se puede repetir**: empieza borrando los dos usuarios sembrados (ids fijos `a1000000-…`). Como las llaves foráneas tienen `ON DELETE CASCADE`, se van también sus cuentas, tarjetas y movimientos. Los usuarios registrados desde la app no se tocan.
- **Fechas relativas**: cada movimiento es `now() - interval '2 hours'`, `'3 days'`, etc. Al volver a ejecutarlo siempre hay movimientos de "Hoy" y "Ayer".
- **Saldo calculado**: las cuentas se insertan en 0 y al final un `UPDATE` suma ingresos y resta egresos. El saldo siempre cuadra con los movimientos.
- **Enums como texto** (`'Egreso'`, `'Comida'`, `'Visa'`): así los guarda EF Core por el `HasConversion<string>()` de `AppDbContext`.
- `SET client_encoding = 'UTF8'` para que las tildes no se dañen al correrlo con `psql` en Windows.

## Y el hasheo de contraseñas

No se comentó ni se cambió nada. El API sigue hasheando con BCrypt al registrar (`AuthService`) y al cambiar la contraseña (`UsersService`).

El seed inserta usuarios directo por SQL, sin pasar por el API, así que trae el hash **ya calculado** con la misma librería (`BCrypt.Net-Next`). Por eso el login de los usuarios sembrados funciona con el código tal cual.

Para generar otro hash: un proyecto de consola con el paquete `BCrypt.Net-Next` y `BCrypt.Net.BCrypt.HashPassword("...")`.

## Si choca con datos existentes

Si ya hay un usuario real con esos correos, cédulas o números de cuenta, el script falla por índice único y no cambia nada. Se arregla cambiando esos valores en el script.
