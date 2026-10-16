# Control financiero (Spark Driver)

App web instalable (PWA) en un solo archivo: `index.html` (HTML, CSS y JS juntos), `sw.js` (funciona sin conexión), `manifest.webmanifest` e iconos. Los datos viven en `localStorage` del teléfono (clave `sd_tracker_v1`).

Datos guardados (todo en `sd_tracker_v1`): `shifts` (turnos), `fills` (llenadas), `personal` (gastos e ingresos, `type`), `apart` (apartados que el dueño marcó a mano: `day`, `fund` = tax/emer/veh, `amount`), `set.reserva` ($/milla) y `set.ap` (configuración de apartados; viene apagada). Cuentas: `acc.spark`/`acc.pers` = `{start, since}` (saldo inicial y desde cuándo cuenta), `moves` (transferencias `kind:'transfer'` y ajustes `kind:'adjust'`), y en los registros `acct` (cuenta con que se pagó) y `tip` en turnos (propina; ya está incluida en `earned`, solo cambia cuándo entra a la cuenta: al día siguiente). El saldo solo usa datos reales, nunca estimaciones. Categorías: `cats` (lista editable) y `cat` en cada gasto. Deudas: `debts` = `{id, name, kind:'card'|'affirm', start, since, limit?, min?, due?, apr?, plan:[{id,date,amount,paid?}]}`; una compra con tarjeta lleva `acct:'debt:<id>'` (sube la deuda y cuenta como gasto una vez); los pagos son `moves` `kind:'pay'` (bajan la deuda y salen del débito, no son gasto) y "Actualizar con el banco" crea `kind:'dadj'` (intereses y cargos). Pagos recurrentes: `recs` = `{id, name, amount, freq:'month'|'week'|'year', day|wday|md, acct?, cat?, start}`; nunca se registran solos: al confirmar se crea un gasto en `personal` con `rec` y `recDue` (el día del cobro). El calendario (`calEvents`) junta recurrentes, día de pago de tarjetas (desde su saldo inicial) y pagos de Affirm. Transferencias instantáneas desde Spark (One): `fee` y `feeMode` en el `move` (`'aparte'`: sale monto + comisión; `'resta'`: llega monto − comisión); `set.fee` = `{pct, min, mode, inst}` (1.75% con mínimo $0.25 por defecto, editable en Ajustes → Cuentas). `set.lastBackup` = fecha de la última copia guardada. `norm()` debe conservar cualquier campo nuevo o se pierde al cargar.

## Preferencias del dueño (tener en cuenta en todo cambio)
- Todo en **español**: textos, mensajes, commits y PRs.
- El dueño usa la app en **iPhone**, como app instalada en la pantalla de inicio.
- **El diseño visual actual le gusta**: colores verdes, letra Instrument Sans, tarjetas redondeadas, iconos de línea en el verde de la app. No cambiarlo sin que lo pida.
- **Las opciones e interacciones deben comportarse como en una app de iPhone**: pantallas nuevas con "‹ Atrás" en lugar de desplegables, gesto de atrás, sin efecto rebote, sin barra de scroll visible, selectores nativos.
- Mantener las pantallas principales (Trabajo y Personal) limpias; lo secundario va en el menú ☰ (grupos Historial y Ajustes).
- Antes de cambios grandes de diseño o de navegación, **proponer y discutir primero**. Los arreglos pequeños se pueden hacer directo.

## Cómo trabajar
- La lógica de cálculo está entre `//LOGIC-START` y `//LOGIC-END`; no tocarla salvo que el cambio lo pida.
- Probar con Playwright (Chromium) en modo claro y oscuro, a 390×844, antes de subir; revisar capturas. Las pruebas que recargan la página deben abrirla con un servidor web local (`python3 -m http.server`), no con `file://`: con `file://` el navegador de pruebas a veces pierde el `localStorage` al recargar.
- No romper los datos guardados: los cambios deben leer los datos que ya existen.
