# Control financiero (Spark Driver)

App web instalable (PWA) en un solo archivo: `index.html` (HTML, CSS y JS juntos), `sw.js` (funciona sin conexión), `manifest.webmanifest` e iconos. Los datos viven en `localStorage` del teléfono (clave `sd_tracker_v1`).

Datos guardados (todo en `sd_tracker_v1`): `shifts` (turnos), `fills` (llenadas), `personal` (gastos e ingresos, `type`), `apart` (apartados que el dueño marcó a mano: `day`, `fund` = tax/emer/veh, `amount`), `set.reserva` ($/milla) y `set.ap` (configuración de apartados; viene apagada). Cuentas: `acc.spark`/`acc.pers` = `{start, since}` (saldo inicial y desde cuándo cuenta), `moves` (transferencias `kind:'transfer'` y ajustes `kind:'adjust'`), y en los registros `acct` (cuenta con que se pagó) y `tip` en turnos (propina; ya está incluida en `earned`, solo cambia cuándo entra a la cuenta: al día siguiente). El saldo solo usa datos reales, nunca estimaciones. `norm()` debe conservar cualquier campo nuevo o se pierde al cargar.

## Preferencias del dueño (tener en cuenta en todo cambio)
- Todo en **español**: textos, mensajes, commits y PRs.
- El dueño usa la app en **iPhone**, como app instalada en la pantalla de inicio.
- **El diseño visual actual le gusta**: colores verdes, letra Instrument Sans, tarjetas redondeadas, iconos de línea en el verde de la app. No cambiarlo sin que lo pida.
- **Las opciones e interacciones deben comportarse como en una app de iPhone**: pantallas nuevas con "‹ Atrás" en lugar de desplegables, gesto de atrás, sin efecto rebote, sin barra de scroll visible, selectores nativos.
- Mantener las pantallas principales (Trabajo y Personal) limpias; lo secundario va en el menú ☰ (grupos Historial y Ajustes).
- Antes de cambios grandes de diseño o de navegación, **proponer y discutir primero**. Los arreglos pequeños se pueden hacer directo.

## Cómo trabajar
- La lógica de cálculo está entre `//LOGIC-START` y `//LOGIC-END`; no tocarla salvo que el cambio lo pida.
- Probar con Playwright (Chromium) en modo claro y oscuro, a 390×844, antes de subir; revisar capturas.
- No romper los datos guardados: los cambios deben leer los datos que ya existen.
