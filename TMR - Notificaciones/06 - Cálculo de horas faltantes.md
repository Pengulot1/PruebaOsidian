# Cálculo de horas faltantes

Parte de [[00 - Índice Notificaciones]]. Dueño según el reparto: **Allan** (detección).

Archivos:
- `Features/Notificaciones/Domain/CalculadorHorasFaltantes.cs`
- `Features/Notificaciones/Application/Queries/ConsultaHorasFaltantes.cs`
- `Features/Notificaciones/Application/Handlers/DetectarHorasFaltantesHandler.cs`
- `Features/TimeReport/Services/CalculoHorasPeriodo.cs` (regla compartida con Seguimiento)

## La regla de negocio (`CalculoHorasPeriodo.Calcular`)

Es **la misma** que usa la tabla de Seguimiento, para que los números coincidan.

1. **Período real del empleado:** el rango pedido, recortado:
   - desde su `Fechaingreso`, si entró después del inicio;
   - hasta su `Fechaterminacion`, si salió antes del fin;
   - **nunca después de hoy** (hora de Ecuador).
2. **Días laborables:** de lunes a viernes, **sin feriados** activos (`TblTimeReportFeriados`).
3. **Jornada del día:** 8 h, o **6 h si el contrato es Pasantía** (código `PAS`). Se usa el contrato **vigente ese día** según el historial (`tbl_administracion_empleado_contrato`).
4. **Horas esperadas** = suma de la jornada de cada día laborable.
5. **Horas registradas** = horas de actividades activas en esos días.
6. **Horas faltantes** = `max(0, esperadas − registradas)`.

> [!note] Las horas extra compensan
> Como se compara el **total** del período, si alguien hizo 10 h un día y 6 h otro, no le falta nada.

`HoyEcuador()` = `UtcNow − 5 h`. Ecuador no tiene horario de verano.

## `CalculadorHorasFaltantes`

- `Calcular(empleado, inicio, fin, actividades, feriados, hoy, jornadas)`: llama a `CalculoHorasPeriodo.Calcular` y devuelve un `ResultadoHorasFaltantes` con `DebeHoras => HorasFaltantes > 0`.
- `PeriodoEnCurso(fecha)` (estático): si el día es ≤ 15 devuelve 1–15; si no, 16–último día del mes.

## `ConsultaHorasFaltantes` (lee la BD)

| Método | Qué hace | Lo usa |
|---|---|---|
| `CalcularAsync(inicio, fin, ids?)` | Carga empleados activos (todos o los ids), feriados, actividades y jornadas en **pocas consultas**, calcula cada uno y devuelve solo los que **deben horas**. Correo = `Emailcorporativo` o, si está vacío, `Persona.Email`. | Detector (flujos 1 y 2), `/pendientes` |
| `ObtenerPendientesAsync()` | Período en curso **hasta ayer** + último aviso manual de cada uno. | `GET /pendientes` |
| `UltimosAvisosManualesAsync(ids)` | `MAX(FechaEnvio)` de los avisos `Manual` + `Enviado` por empleado. | Detector (flujo 2), `/pendientes` |
| `NotificadosEnCorteAsync(fechaCorte)` | Ids con un aviso `Automatico` + `Enviado` en ese corte. | Detector (flujo 1) |

**Consume:** `ApplicationDbContext` → `TblAdministracionEmpleados` (+ Persona y TipoContrato), `TblTimeReportFeriados`, `TblTimeReportActividadDiaria`, `NotificacionEnvio`.

## `DetectarHorasFaltantesHandler`

Escucha **dos** eventos:

- `CorteDeHorasAlcanzado` (flujo 1) → todos los empleados; descarta a los ya notificados en el corte.
- `NotificacionManualSolicitada` (flujo 2) → solo los ids pedidos, hasta ayer; descarta a los que están dentro del umbral.

En ambos casos descarta a quien no tiene correo (con un warning en el log) y publica un `HorasFaltantesDetectadas` por cada empleado que queda.

Funciones auxiliares públicas, testeables sin BD:

- `FinManualEfectivo(finSolicitado, hoy)` → `min(finSolicitado, hoy − 1)`.
- `DentroDelUmbral(ultimo, ahora, umbralHoras)` → `true` si hubo aviso, el umbral es > 0 y pasó **menos** que el umbral.

Detalle del recorrido: [[04 - Flujo 1 - Corte automático con Quartz]] y [[05 - Flujo 2 - Aviso manual del líder]].
