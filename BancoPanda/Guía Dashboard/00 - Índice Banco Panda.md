---
tags: [bancopanda, indice]
creado: 2026-10-15
---
# 🐼 Banco Panda — Índice

Notas para entender el proyecto Banco Panda: qué hace, cómo está armado y, sobre todo, cómo funciona el **Dashboard** (la pantalla de inicio de la app).

El código está en `D:\PROYECTOS DEL TRABAJO\codigo\`:

| Carpeta | Qué es | Tecnología |
|---|---|---|
| `bancopanda-api` | El **backend**: guarda los datos y responde a la app | .NET 10 + PostgreSQL |
| `bancopanda-app` | La **app móvil** que usa el cliente | Flutter (Dart) |

## Orden de lectura recomendado

1. [[01 - Cómo funciona Banco Panda]] → explicación general, sin tecnicismos.
2. [[02 - API - Endpoints]] → todos los endpoints de la API, explicados uno por uno.
3. [[03 - App - Dashboard paso a paso]] → cómo la app arma el Dashboard, archivo por archivo.
4. [[04 - Dashboard - Qué endpoint alimenta cada parte]] → mapa visual de la pantalla con el endpoint de cada parte.

## Glosario rápido

- **Endpoint**: una "dirección" de la API a la que la app le pide algo. Ej: `GET /api/cuentas` = "dame mis cuentas".
- **JWT / token**: un "pase" que la API le da a la app al iniciar sesión. La app lo manda en cada petición para demostrar quién es.
- **DTO**: el "molde" de los datos que entran o salen de la API.
- **Entidad**: una tabla de la base de datos representada en código (Usuario, Cuenta, Tarjeta, Movimiento).
- **ViewModel**: en la app, la clase que tiene el estado de una pantalla (cargando, error, datos) y avisa a la vista cuando cambia.
- **Repository / DataSource**: en la app, las capas que hablan con la API.
