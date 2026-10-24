# DATOS.md — Cómo guarda y calcula la app

Detalle de los datos y de cada cálculo. Se guarda en `localStorage`, clave `sd_tracker_v1`, en el teléfono de cada persona (no hay nube). Mantenerlo al día cuando cambie un dato o una regla.

> **Regla de oro:** `norm()` debe conservar cualquier campo nuevo, o se pierde al cargar. Los cambios deben leer los datos que ya existen. El saldo usa solo datos reales, nunca estimaciones.

Reglas de trabajo en `CLAUDE.md`. Diseño en `DESIGN.md`. Estado y pendientes en `PROJECT_STATE.md`.

---

## 1. Cuentas y dinero

**Cuentas**
- `acc.spark`, `acc.pers`, `acc.ahorro` = `{start, since, name?}`: saldo inicial, desde cuándo cuenta y nombre que el dueño le puso.
- Cuentas agregadas: ids en `acc.ids` (`a_…`), cada una en `acc[id]`. Funcionan como Personal. Solo se borran si no tienen movimientos.
- `ACC` y `ACCK` se actualizan con `accSync`. Para recorrer las cuentas, usar `ACCK`.
- Spark siempre deposita en `spark`. La reserva del carro va a `RA()` (por defecto `ahorro`), aunque se renombren.
- **Ahorro** = cuenta de ahorro dentro de One donde el dueño guarda la reserva.
- **Cuentas de ahorro** (`isSav`): `ahorro`, `RA()`, cuentas de metas y las marcadas con `acc[k].sav` ("Es cuenta de ahorro"). No salen en "Dinero disponible" ni cuentan en él.
- Ajustes → Cuentas: lista. Cada cuenta tiene su pantalla con Ajustar al banco, Transferir y Editar datos (nombre, saldo de hoy y, en Spark, la comisión instantánea).
- Editar cuenta → "Aquí guardo la reserva del carro" cambia `set.resAcc`.

**Movimientos** (`moves`)
- `kind:'transfer'`: transferencias. Spark ↔ Ahorro no tiene comisión.
- `kind:'adjust'`: ajustes al banco.
- `kind:'pay'`: pago a una deuda. Baja la deuda y sale del débito. **No es gasto.**
- `kind:'dadj'`: intereses y cargos que registra "Actualizar con el banco" en deudas.
- Comisión instantánea de One (desde Spark): `fee` y `feeMode` en el move.
  - `'aparte'`: sale monto + comisión.
  - `'resta'`: llega monto − comisión.
  - `set.fee` = `{pct, min, mode, inst}`. Por defecto 1.75 % con mínimo $0.25. Se edita en Ajustes → Cuentas.
- En los registros, `acct` = cuenta con que se pagó.

**Dinero disponible** (Personal): solo cuentas de uso diario (Spark + Personal). Las propinas por llegar se muestran aparte como "+$X pendientes".

**Balance del mes** (Personal → "Balance de <mes>"), solo datos reales del mes:

`ganado + otros ingresos − gasolina pagada en llenadas − reserva del carro − gastos pagados con débito/efectivo − pagos a deudas (pay) − comisiones de transferencias`

- Reserva del carro: la calculada de los turnos anteriores a configurar el Ahorro, más lo transferido al Ahorro desde entonces. Sin Ahorro, solo la calculada.
- Las compras con tarjeta de crédito **no** restan aquí: restan cuando se paga la tarjeta. Se muestran en la nota.
- Los intereses (`dadj`) tampoco restan aquí.
- Balance y Disponible no tienen que coincidir.

**Ingresos fijos (sueldo)** — `incs` = `{id, name, amount (neto), freq:'week'|'2week'|'semi'|'month', next, acct}`
- Salen en el calendario como dinero que entra (`type:'inc'`). No cuentan en los totales de cobros.
- Nunca se registran solos. "Me llegó ✓" crea un ingreso en `personal` con `inc` e `incDue`.
- La meta diaria resta el sueldo de lo que necesita al mes.

---

## 2. Turnos, propinas y Spark

**Turno** (`shifts`)
- Se guarda al terminar. Cada turno nuevo guarda también `sT` (total del día) y `sC` (propinas confirmadas).
- `tip` = propina de ese día. Ya está incluida en `earned`; solo cambia cuándo entra a la cuenta.
- `tp:1` = propina por confirmar.
- `spk=1` = el informe de Spark confirmó el turno completo. `spk=2` = la base ya viene del informe, la propina sigue por llegar.

**Terminar turno** pide lo que muestra Spark en Ganancias del día:
- **T** = total del día.
- **C** = propinas confirmadas (son del turno anterior).
- **P** = propinas no confirmadas (las de hoy).
- Resultado: `earned = T − C + P`, `tip = P`, `tp:1`.
- C corrige la propina del último turno con `tp` (`tipConfirm`). En el historial, "Confirmar" hace lo mismo a mano.

**Editar turno** pide los mismos tres números y recalcula la ganancia.
- Si C no coincide con la propina del turno anterior (y ese turno no lo confirmó el informe, `spk=1`), corrige también ese turno con `tipConfirm`, igual que al terminar.
- Turnos viejos sin `sT`: si no se pone el total, se conserva lo ganado sin propinas.
- Turnos viejos sin `tp` no se tocan.

**Propinas:**
- Entran a la cuenta 24 h después de la mitad del turno. Spark confirma cada propina 24 h después de la entrega.
- Mientras tanto se ven como "por llegar".
- `tipPrev()` devuelve el turno anterior solo si su propina sigue pendiente.
- Aviso `tipAsk`: propinas con `tp` de hace más de 48 h.

**Informe de Spark (.xlsx)** — Menú → Trabajo (Spark) → Informe de Spark (se esconde en modo fijo)
- Lector propio: ZIP + `DecompressionStream` + `DOMParser`, sin librerías.
- Lee la hoja "Transacciones" (encabezados por nombre; fila con "ID del viaje"; ignora el pie). Comprueba que la suma cuadre con "Ganancias totales" de la hoja "Resumen". Si no cuadra, no importa nada.
- `spk` = `{rows:[{k,i,t,y,a}], files:[{name,at,from,to,n,add}], undo}`:
  - `k` = huella (id|hora|tipo|monto|n.º de aparición), para no duplicar.
  - `t` = hora local `YYYY-MM-DDTHH:MM`.
  - `y` = tipo: `v` viaje, `p` propina, `i` incentivo, `m` multiplicador, `a` ajuste, `o` otro.
- **Asignación a turnos:**
  - Propinas, incentivos y multiplicadores van a la hora de su viaje (mismo ID, el más cercano antes, máximo 3 días).
  - Cada movimiento va al turno cuya hora lo contiene (±30 min). Si no, al turno del mismo día. Si no hay turno, "Ganancias sin turno", que solo cuentan para el total del año.
- **Qué corrige:**
  - Solo los turnos que el informe cubre completos.
  - Si las propinas aún no se publican (viaje + 24 h después del corte), cambia solo la base y la propina sigue por llegar (`spk=2`).
  - Con propinas completas: `earned = base + propinas`, `tip` = propinas, se quita `tp` y queda `spk=1`.
  - Turnos sin propina anotada y con propinas incompletas no se tocan.
- **Nunca** toca millas, horas ni odómetro.
- Vista previa antes de guardar (incluye el cambio en el saldo de Spark). "Deshacer última importación" restaura los turnos y quita los movimientos agregados.

**Turno guardado** (`tdone`) — se abre al terminar el turno (‹ vuelve a Trabajo)
- Muestra: Hoy ganaste, Te quedó (gasolina y reserva), millas y horas.
- **Aparta hoy:** cada meta pendiente ("10% de $X", lo que lleva apartado y cuánto le falta) y la Reserva carro (millas × $/mi, lo de antes y a qué cuenta).
- **"Ya lo aparté ✓":** marca las metas sin cuenta en `apart` y crea transferencias desde Spark para las metas con cuenta (`mt`) y la reserva (a `RA()`).
- **"Después":** lo deja en la campanita, que avisa "Pasa $X a <cuenta de la reserva>".
- Abajo: la meta del día y si la propina anterior cambió.

**Contador del turno — en prueba** (`active.lv`)
- `active.lv` = lo que llevas en el turno abierto (aprox.).
- "La acepté ✓" en Evaluar oferta (solo con turno abierto) le suma el pago. El renglón "Llevas en este turno" deja ponerlo a mano.
- La meta de hoy muestra "Llevas $X (aprox.)" = ganado hoy + `lv`.
- Al terminar el turno se usa el total real de Spark y `lv` desaparece con `active`.
- **Nunca** cuenta en saldos ni en cálculos.

---

## 3. Estadísticas, metas y ofertas

**Estadísticas de Trabajo** (Hoy / Semana / Mes)
- "Te quedó por hora (est.)", con "Ganado por hora · te quedó el X %".
- Cuadritos: Ganado, Gasolina (est.), Reserva, Resultado (est.), Horas, Millas y Por milla.
- Por milla = lo ganado ÷ millas del odómetro, incluye vueltas. Sirve para comparar con las ofertas.
- La gasolina estimada por milla solo se usa en Trabajo.
- Llenadas de gasolina: MPG y costo por milla.

**Reserva del carro**
- `set.reserva` = $ por milla.
- `set.resAcc` = cuenta donde va (por defecto `ahorro`). Usar `RA()`.
- `set.resSince` = desde cuándo cuenta en esa cuenta. Usar `raT0()`.
- Solo cuentan las transferencias a `RA()` sin `mt`.

**Meta diaria** — `set.goal` = `{days, need?}`
- `days` = días que trabaja por semana. `need` = lo que necesita al mes (opcional).
- Sin `need`, `goalNeed()` lo calcula: pagos recurrentes + mínimos de tarjetas + cuotas del mes + gasto normal (gastos con débito sin recurrentes, promedio de los últimos 30 días) + costo del carro (gasolina + reserva, % de lo ganado).
- El primer mes cuenta solo desde el día en que empezó a registrar.
- **Meta de hoy** = (meta del mes − ganado antes de hoy) ÷ días de trabajo que quedan.
- Se ve en Trabajo y en el aviso al terminar el turno, con barra de avance.

**Evaluar oferta** — `set.offer` = `{minMile, minHour, ret}`
- Calcula $/milla y $/hora, y compara con tus mínimos y tu promedio.
- `ret` = "Contar la vuelta al Walmart": suma millas de regreso (vacío = igual que las de ida) y ~2 min por milla de regreso.
- La calculadora no guarda ofertas.

**Mejores días y horas** (`rBest`; Menú → Trabajo (Spark); se esconde en modo fijo)
- Por día de la semana: lo ganado por día trabajado según el informe (o $/h de los turnos si no hay informe), y $/h de los turnos.
- Por horario (mañana, mediodía, tarde, noche, madrugada): lo ganado por hora con viajes (aproximado; no cuenta la espera).
- "Mejor" solo con 4+ días y 15+ viajes. Si no, dice "pocos datos".

**Metas y apartados** — `metas` = `{id, name, mode:'pct'|'week'|'free', val, target?, since, acct?}`
- `pct` = % de lo ganado (neto de cada turno). `week` = $ por semana. `free` = lo que pueda.
- Lo que falta apartar cuenta desde `since` (al crearla, o con "Empezar a contar desde hoy") y se detiene al llegar a `target`.
- `acct` opcional = cuenta donde se guarda. Entonces "Depositar" abre Transferir con "Es para", y la transferencia lleva `mt` = id de la meta, que cuenta como aportado. Las cuentas de metas no entran en "Disponible hoy".
- `set.apBase` = `'net'` (por defecto: % sobre lo que te quedó) o `'gross'` (sobre lo ganado). Se cambia en Metas → "Calcular el % sobre" (`apBaseOf`).
- Meta con `target` llena: su `since` pasa a hoy (`rAp`). Al usar dinero, solo vuelve a pedir el % de lo nuevo.
- Bolsillos de OnePay: cada uno es una cuenta de ahorro ligada a su meta.
- `set.ap` y los fondos tax/emer/veh ya no se usan.

`apart` = `{day, fund (id de la meta), amount, note?}`
- Positivo = aparté. Negativo = usé. `note` = motivo.
- **"Usé…":** si la meta tiene cuenta propia, crea una transferencia de esa cuenta a la primera cuenta de uso diario (no Spark), con `mt` y `note`. `mtDep` guarda la cuenta como retiro.
- **"Pagué taxes"** (metas con "impuesto" o "tax" en el nombre): anota el gasto desde la cuenta de la meta y un `apart` negativo.

---

## 4. Deudas

`debts` = `{id, name, kind:'card'|'affirm', start, since, limit?, min?, due?, apr?, upd?, plan:[{id, date, amount, paid?, name?, k?, of?}]}`

- `upd` = última vez que se tocó "Actualizar con el banco", aunque ya coincidiera. Quita el recordatorio de esa tarjeta.
- **Compra con tarjeta:** `acct:'debt:<id>'`. Sube la deuda y cuenta como gasto una sola vez.
- **Pago:** `moves` con `kind:'pay'`.
- **Nuevo gasto:** las deudas salen como categorías "Pago a <deuda>" (valor `pay:<id>`). Eso crea un `pay` en vez de un gasto. Si hay una cuota de Affirm sin pagar con ese monto, la marca pagada.
- **"Actualizar con el banco"** crea `dadj` (intereses y cargos).
- **"Agregar plan"** crea de una vez todos los pagos que faltan de un plan de Affirm, cada 2 semanas o cada mes. `name` = qué se compró. `k` de `of` = número de pago.
- `kind:'affirm'` sirve para cualquier plan a cuotas (Affirm, cuotas de Apple Card). Apple Card lleva solo el saldo con interés; "Apple cuotas" va aparte.
- **Pantalla de Deudas:**
  - "Debo en total", con el patrimonio (todas las cuentas − deudas) debajo.
  - Siguiente paso según el APR (`coach`).
  - Evolución del total (`debtTotAt`) e intereses pagados (`dadj`).
  - En cada tarjeta con APR, simulador de pago (`simPay`).
- Personal: "Deuda total" = "N deudas registradas".

---

## 5. Calendario, pagos recurrentes y avisos

**Pagos recurrentes** — `recs` = `{id, name, amount, freq:'month'|'week'|'year', day|wday|md, acct?, cat?, start}`
- Nunca se registran solos. Al confirmar, se crea un gasto en `personal` con `rec` y `recDue` (el día del cobro).

**Calendario** (`calEvents`) junta:
- Recurrentes.
- Día de pago de tarjetas (desde su saldo inicial).
- Pagos de Affirm.
- Sueldos (`incs`).
- "Hecho ✓" = `evAct`. Tocar un renglón abre ese día, donde se confirma (`evTap`).

**Próximos pagos** (Personal): 7 días, máximo 3, solo lo pendiente.
- Lo ya registrado se quita al instante y sigue en el calendario con ✓.
- Cada renglón: icono por tipo, monto y estado (Registrado / Pendiente / Hoy / Atrasado).

**Campanita de avisos** (arriba a la derecha, con numerito rojo; pantalla `avisos`). `avList()` los arma desde los datos:
- **Pago semanal a deuda:** `avisos.dpays` = `{id, debt, amount, wday, since, acct?}`. "Hecho ✓" abre Pagar con el monto y la última cuenta usada (se guarda en `acct`). Se quita sola si hay un `pay` a esa deuda con ese monto desde el día anterior.
- **Cobros, cuotas y sueldo** a 3 días o menos, y atrasados. "Hecho ✓" = `evAct`, igual que el calendario.
- **Copia de seguridad:** 7 o más días desde `set.lastBackup`.
- **Actualizar tarjeta:** `updDue`.
- **Propinas** sin confirmar (`tipAsk`).
- **Apartados** pendientes ($1 o más).
- **Avisos propios:** `avisos.custom` = `{id, title, date, rep:'once'|'week'|'month'}`. "Hecho ✓" los marca en `avisos.done`.
- **"Recordar mañana"** = `avisos.snooze[clave] = mañana`.
- Los avisos automáticos se quitan solos cuando ya está hecho.
- Las tarjetas "Actualiza el saldo" (`#updcard`) y "Copia de seguridad" (`#bkcard`) están ocultas en Personal: viven en la campanita.

---

## 6. Gastos, categorías y formas de pago

- `personal` = gastos e ingresos (`type`). Campos de gasto: `cat`, `acct`, `rec`, `recDue`. Campos de ingreso: `inc`, `incDue`.
- Tocar un gasto lo edita. Deslizar a la izquierda muestra Borrar (pregunta antes), con `rowTap`.

**Categorías**
- `cats` = lista editable. `cat` en cada gasto.
- Lista recomendada (`CATS0`): Comida y súper, Restaurantes, Vivienda, Servicios, Carro, Salud, Compras, Familia y envíos, Diversión, Cuidado personal, Impuestos y trámites, Otros. Además, "Pagos de deudas" para pagos viejos anotados como gasto.
- Cada una tiene icono de línea (`CICO`, `catIco`). Las propias llevan una etiqueta. Se ven en Categorías y en la gráfica de Gastos.
- `catMigrate()` corre una sola vez (`set.catV = 2`):
  - Pasa tiendas, servicios y formas de pago a su categoría (`CMAP`: Walmart/Costco → Comida y súper; Spectrum/Netflix/teléfono → Servicios; Zelle → Familia y envíos).
  - Las categorías con nombre de deuda pasan a "Pagos de deudas".
  - Las notas de recreación pasan a Diversión.
  - Las propias que no se reconocen se quedan al final.
- En Ajustes → Categorías:
  - **Renombrar:** cambia `cat` en `personal`, `recs` y `acc.lastCat`. Si el nombre ya existe, se unen.
  - **Borrar:** los gastos quedan sin `cat`.
  - No se permite el nombre de una deuda.

**Formas de pago** (Ajustes → Formas de pago, `rPagos`)
- `set.payOff` = claves apagadas (una cuenta, o `debt:<id>`).
- `payOpts` no las muestra en "Pagado con", salvo que sea el valor ya guardado del registro.
- No borra nada.

---

## 7. Copia de seguridad

- Exportar e importar en JSON.
- Importar usa `mergeIn`: une la copia con los datos actuales y muestra antes qué agrega.
- `set.lastBackup` = fecha de la última copia guardada.

## 8. Modo "Mi trabajo" (`set.job`)

- `'spark'` (por defecto): la app igual que siempre.
- `'fijo'`: esconde la pestaña Trabajo, las opciones solo de Spark del menú (grupo `#grpw`/`#secw`: Informe de Spark, Mejores días, etc.) y las cuentas sin configurar. **No borra nada.**
- `'ambos'`: Spark y trabajo fijo.
- Los ingresos fijos (sueldo) son independientes de este ajuste.
- Ajustes → Mi trabajo.

---

## 9. Código y navegación (para cambios)

- Lógica de cálculo entre `//LOGIC-START` y `//LOGIC-END`. No tocarla salvo que el cambio lo pida.
- **Navegación:** `go(v, push)` usa `history.pushState`.
  - `go(v, true)` guarda en `history.state.y` cuánto habías bajado en la pantalla anterior. Al regresar (gesto o ‹) se restaura, sin salto.
  - La animación de regreso (`anim('back')`) solo corre si `btnBack` es true (botón ‹ o borrar deuda). Con el gesto de atrás, Safari ya la hace.
- **Ventanas:** `ask({t, q, v, num, hint, ok, neg})` y `sure({t, m, ok, red})`. Nunca `prompt`, `confirm` ni `alert`.
- **Listas:** `rowTap(contenedor, k)` en turnos, llenadas y gastos. `evTap` en calendario e ingresos fijos.
- Campos de fecha con `appearance:none`, para que Safari no los haga más anchos.
- **IDs de Personal:** `#acard` (Dinero disponible), `#dcard` (Balance y Deuda), `#calgo2` ("Ver todos"), `#psumd` (abre el resumen del mes), `#psum` (oculta, solo por compatibilidad).
- **Otros IDs:** `#updcard`, `#bkcard`, `#grpw`/`#secw` (grupo Trabajo (Spark) del menú).
