# 06 - Preguntas que te pueden hacer

Vuelve al [[00 - Guion de la exposición]].

## Arquitectura

**¿Por qué feature-first?**
Porque todo lo de una funcionalidad está junto (`features/transfers/`, `features/mobile_cash/`…). Es más fácil encontrar y cambiar cosas, y varias personas pueden trabajar en paralelo sin pisarse.

**¿Qué es MVVM y dónde se ve?**
Model (datos), View (pantalla) y ViewModel (lógica y estado). Ejemplo: `transfer_screen.dart` (View) solo dibuja; `transfer_view_model.dart` (ViewModel) decide en qué paso está, valida y llama al repositorio; `comprobante.dart` (Model) es la forma de los datos.

**¿Cómo se entera la pantalla de que cambió algo?**
El ViewModel extiende `ChangeNotifier` y llama a `notifyListeners()`; la pantalla lo escucha con `context.watch` (paquete `provider`) y se redibuja.

**¿Para qué el Repository si ya está el DataSource?**
Para que el ViewModel no dependa de HTTP. En los tests se reemplaza el DataSource por uno falso (`test/helpers/`) y el ViewModel funciona igual.

**¿Por qué la API está separada de la app?**
Para que las reglas del dinero estén en un servidor de confianza. Si la app hablara directo con la base, cualquiera podría cambiar saldos desde el celular.

## Dinero y seguridad

**¿Qué pasa si dos transferencias llegan al mismo tiempo?**
El API bloquea la cuenta (`SELECT … FOR UPDATE`); la segunda espera a que termine la primera y ve el saldo real. Se probó con 3 pedidos simultáneos.

**¿Y si el internet falla y el usuario toca "Transferir" otra vez?**
Cada intento lleva un `solicitudId`. Si llega repetido, el API devuelve el mismo comprobante y no cobra dos veces.

**¿Qué pasa si falla algo a la mitad de una transferencia?**
Todo está en una transacción: o se guarda todo o no se guarda nada.

**¿Cómo se guardan las contraseñas?**
Cifradas con BCrypt. Los códigos de efectivo móvil también.

**¿Dónde están las claves de la base de datos?**
No están en GitHub: en *user-secrets* (local) y en variables de entorno de Render (producción).

**¿Puedo ver los datos de otro usuario?**
No. Todos los endpoints filtran por el usuario del token JWT.

## Funcionalidades

**¿Las transferencias a otros bancos son reales?**
No: se descuenta de la cuenta y se guarda el comprobante, pero el envío al otro banco se simula. Dentro de Banco Panda sí llega el dinero a la otra cuenta.

**¿Pagar servicios descuenta dinero?**
Todavía no; es un prototipo visual con agua, luz e internet. El siguiente paso es un endpoint en el API que descuente y registre el movimiento.

**¿Llega un SMS con los códigos de efectivo móvil?**
No; se muestran en la app. Para SMS reales habría que conectar un proveedor como Twilio (tiene costo por mensaje).

**¿Qué pasa si un efectivo móvil vence?**
Queda registrado con vencimiento de 24 h. Devolver automáticamente el dinero al vencer es una mejora pendiente.

**¿Por qué se ve el número completo de la tarjeta?**
Lo pidió el diseño del proyecto. En un banco real no se guardaría en texto (norma PCI).

## Calidad

**¿Hay pruebas?**
En la app, 89 tests automáticos (`flutter test`) de ViewModels, validaciones y modelos. El API se probó con pedidos reales contra una base de prueba y con el archivo `.http`; falta un proyecto de tests automáticos del API.

**¿Cómo se actualiza la base de datos en producción?**
Con migraciones de EF Core: al hacer push a `main`, Render construye la API y al arrancar aplica las migraciones pendientes en Supabase.

**¿Qué mejorarías?**
1. Tests automáticos del API.
2. Pagar servicios conectado al API.
3. Devolver el dinero de efectivos móviles vencidos.
4. SMS o notificaciones push con los códigos.
5. Ver comprobantes anteriores desde el historial.
