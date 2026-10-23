# PROJECT_STATE.md — Mis cuentas (Control financiero · Spark Driver)

Estado del proyecto al **23 de octubre de 2026** (65 PRs unidos a `main`, del 12 al 23 de octubre).
Este archivo resume todo el historial de conversación para empezar sesiones nuevas sin repetirlo.
Reglas de trabajo en `CLAUDE.md`, detalle de datos y cálculos en `DATOS.md` y diseño en `DESIGN.md`.

---

## 1. Resumen del estado actual

- **Qué es:** una app personal de finanzas para un repartidor de **Walmart Spark Driver**. El dueño la usa en su **iPhone**, instalada en la pantalla de inicio.
- **Qué hace:**
  - Lleva sus turnos, gasolina, propinas, cuentas, deudas, metas de ahorro, pagos recurrentes y gastos personales.
  - Le dice cuánto le queda de verdad después del carro.
- **Estado:** funciona y se usa a diario con datos reales.
  - Todo lo construido está en `main`.
  - Rama de trabajo: `claude/code-review-design-improvements-8dxp7k`.
  - Flujo: cada cambio se sube en un PR que se une enseguida, porque el dueño lo autorizó ("hazlo como tú creas mejor").
- **Últimos cambios (22–23 oct):**
  - Revisión de diseño con las guías de Apple.
  - Hojas que suben desde abajo en vez de las ventanas del navegador.
  - Listas como en iPhone.
  - Diseño moderno en Trabajo y Personal.
  - Contador "lo que llevas en el turno", **en prueba**.
- **Preferencias del dueño:**
  - Todo en español.
  - Le gusta el diseño verde (letra Instrument Sans, tarjetas redondeadas, iconos de línea).
  - Quiere que todo se comporte como una app de iPhone.
  - Pide ver maquetas o imágenes antes de cambios grandes.
  - No quiere perder datos nunca.
  - No quiere "Accesos rápidos" en Personal.

## 2. Arquitectura y tecnologías

| Parte | Cómo está hecho |
| --- | --- |
| App | **PWA de un solo archivo**: `index.html` (HTML + CSS + JS, ~1,350 líneas largas), sin frameworks ni librerías externas |
| Sin conexión | `sw.js`: primero intenta la red y, si no hay, usa la copia guardada (no hace falta cambiar versión al publicar) |
| Instalación | `manifest.webmanifest` + iconos `icon-180/192/512.png` |
| Datos | `localStorage`, clave `sd_tracker_v1`. Cada teléfono tiene sus propios datos; no hay servidor ni nube todavía |
| Copia de seguridad | Exportar/importar JSON. Importar une la copia con los datos actuales (`mergeIn`) y muestra antes qué agrega |
| Letra | Instrument Sans (Google Fonts) |
| Informe de Spark | Lector propio de `.xlsx` (ZIP + `DecompressionStream` + `DOMParser`), sin librerías |
| Código | `norm()` normaliza y debe conservar todo campo nuevo. La lógica de cálculo va entre `//LOGIC-START` y `//LOGIC-END`. Navegación con `go(v, push)`, `history.pushState` y gesto de atrás |
| Pruebas | Playwright (Chromium) a 390×844, en claro y oscuro, con un servidor local (`python3 -m http.server`) y la copia real del dueño (no está en el repo). Siempre con capturas |
| Herramientas de diseño | Skill `apple-design` en `.claude/skills/apple-design` (guías de Apple, para auditar). `DESIGN.md` con colores, letra y componentes |
| Repositorio | GitHub `KevinF25/Control-financiero-`. Commits y PRs en español |

**Futuro planeado (no empezado):** conexión con el banco por **SimpleFIN** (solo lectura), nube en **Supabase** e inicio de sesión, con funciones en **Vercel**. Las reglas de seguridad ya están escritas en `CLAUDE.md`:
- claves solo en las variables de Vercel;
- RLS en todas las tablas;
- `service_role` nunca en la app;
- GRANT explícito solo para `authenticated` en cada tabla nueva desde el 30 oct 2026.

## 3. Lo que ya está construido y funciona

**Trabajo (Spark)**
- **Turnos:**
  - Empezar y terminar turno con el odómetro.
  - Terminar turno pide las tres cifras de Spark: total del día, propinas confirmadas y propinas por confirmar.
  - Las propinas se corrigen solas en el turno anterior (`tipConfirm`); Editar turno hace lo mismo.
- **Propinas:** entran a la cuenta 24 h después de la mitad del turno; mientras tanto se ven como "por llegar".
- **Turno guardado:** muestra lo ganado y lo que quedó, más "Aparta hoy" (metas y reserva del carro) con "Ya lo aparté ✓" o "Después".
- **Llenadas de gasolina:** MPG y costo por milla.
- **Reserva del carro:** $ por milla, guardada en la cuenta que el dueño elija.
- **Estadísticas Hoy/Semana/Mes:** te quedó por hora, ganado, gasolina, reserva, resultado, horas, millas y por milla.
- **Meta diaria automática:** sale de los gastos del mes, con barra de avance.
- **Evaluar oferta:** $/milla y $/hora, cuenta la vuelta al Walmart, compara con tus mínimos y tu promedio.
- **Contador del turno (en prueba):**
  - "La acepté ✓" en Evaluar oferta suma el pago a lo que llevas hoy.
  - También se puede poner a mano lo que dice Spark.
  - Es aproximado y no toca saldos.
- **Informe de Spark (.xlsx):**
  - Importa viajes, propinas e incentivos y corrige los turnos que cubre completos.
  - Nunca toca millas, horas ni odómetro.
  - Muestra una vista previa antes de guardar y se puede deshacer.
- **Mejores días y horas:** por día de la semana y por horario, a partir del informe; solo marca "mejor" si hay datos suficientes.

**Tu dinero**
- **Cuentas:**
  - Spark, Personal, Ahorro y las que el dueño agregue; cada una con su pantalla.
  - Ajustar al banco, Transferir (con la comisión instantánea de One) y Editar datos.
  - Se pueden marcar como cuentas de ahorro.
- **Dinero disponible:** solo cuentas de uso diario, con las propinas pendientes.
- **Deudas:**
  - Tarjetas y planes a cuotas (Affirm, Apple cuotas); "Agregar plan" crea todas las cuotas de una vez.
  - Pagos, "Actualizar con el banco" (registra intereses), siguiente paso según el APR, simulador de pago, evolución e intereses pagados.
  - Patrimonio.
- **Metas y apartados:**
  - Por % de lo ganado, $ por semana o "lo que pueda", con objetivo.
  - Cuenta propia opcional; los bolsillos de OnePay funcionan como cuentas.
  - "Usé…" con motivo, "Pagué taxes", y la meta vuelve a llenarse sola.
  - El % se calcula sobre lo que te quedó o sobre lo ganado.
- **Calendario de cobros:**
  - Pagos recurrentes (se confirman a mano, se pueden marcar pagados antes), cuotas, fechas de tarjetas y sueldo ("Me llegó ✓").
- **Gastos personales:**
  - Categorías con iconos, ordenadas con una migración automática; renombrar, unir y borrar.
  - Gráfica por categoría.
  - Formas de pago que se pueden apagar.
- **Balance del mes:** solo datos reales.
- **Mi trabajo:** Spark, trabajo fijo o los dos; ingresos fijos (sueldo).

**Avisos y navegación**
- **Campanita con numerito:**
  - Pagos semanales a deudas, cobros próximos o atrasados, copia de seguridad, actualizar tarjeta, propinas sin confirmar, apartados pendientes y avisos propios.
  - Cada aviso tiene "Hecho ✓" y "Recordar mañana".
- **Menú ··· por temas:** Tu dinero, Trabajo (Spark) y Ajustes.
- **Barra de abajo:** Trabajo · (+) · Personal, flotante.
- **Navegación como en iPhone:**
  - Pantallas con ‹ y gesto de atrás; al regresar, la pantalla vuelve a donde estaba (sin salto).
  - Hojas desde abajo (`ask`) y confirmaciones desde abajo (`sure`); nunca `prompt`, `confirm` ni `alert`.
  - En turnos, llenadas y gastos, tocar el renglón edita y deslizar a la izquierda borra (pregunta antes).
  - Botones de 44 pt o más.
  - Interruptores del iPhone y campos de fecha del ancho correcto.

**Diseño**
- Modo claro y oscuro, según lo que tenga el iPhone.
- Contraste medido: 0 fallas en 21 pantallas.
- **Diseño moderno** en Trabajo y Personal:
  - títulos grandes con la fecha;
  - tarjeta verde llena en "Dinero disponible", con los centavos más pequeños;
  - títulos de sección ("Resumen", "Próximos pagos · Ver todos");
  - estados como etiquetas (Hoy / Pendiente / Atrasado);
  - selector Hoy/Semana/Mes como el del iPhone.
- `DESIGN.md` documenta todo esto.

## 4. Tareas pendientes

**Para revisar con el dueño (recién hecho)**
- [ ] **Contador del turno:** el dueño lo está probando unos días. Si no sirve, quitarlo (`active.lv`, "La acepté ✓", el renglón "Llevas en este turno"); si sirve, dejarlo y quitar la palabra "prueba" de `DATOS.md`.
- [ ] **En el iPhone real:**
  - que la barra flotante no choque con la rayita de inicio;
  - que las hojas queden encima del teclado y que el teclado salga solo;
  - que el gesto de atrás no brinque.
- [ ] **Raya vertical del dueño:** marcó una raya junto a la flechita de "Apartados" en una captura y no quedó claro qué quería. Preguntar.
- [ ] **Fechas en "Próximos pagos":** en su iPhone salen como "vie, oct 23" (formato del teléfono) y no "vie, 23 de oct". Ajustar si le molesta.

**Ideas propuestas que el dueño no ha aceptado todavía**
- [ ] Crear una skill propia "probar la app": las pruebas de siempre en un paso.
- [ ] Que la app siga el **texto más grande** del iPhone. Solo si el dueño usa la letra grande.
- [ ] Que la propina entre poco a poco al saldo de Spark (como llegan en realidad).
- [ ] Opción de Autosave para los bolsillos.
- [ ] Esconder cuentas en Transferir.
- [ ] Pantalla "Taxes del año".
- [ ] Anotar solo el interés de la cuenta de ahorro (APY). Hoy se hace a mano con "Ajustar al banco".

**Proyecto grande para más adelante (el dueño dijo "déjalo para el futuro")**
- [ ] Conexión con el banco por **SimpleFIN**, solo lectura, desde una función de Vercel.
- [ ] Nube con **Supabase**: inicio de sesión obligatorio, RLS, GRANT solo para `authenticated` y protección de contraseñas filtradas.
- [ ] Revisar claves expuestas antes de cada entrega. Las reglas están en `CLAUDE.md`.

**Cosas que el dueño tiene que hacer en su app (no son código)**
- [ ] Volver a guardar el turno del 20 oct (Editar → Guardar) para corregir la propina del 19 oct (+$12).
- [ ] Crear las cuentas de los bolsillos (Taxes, Colchón) y ligarlas a sus metas; ponerle a Emergencias el objetivo de $500.
- [ ] Marcar "Apple Card ahorro" como cuenta de ahorro, si es ahorro.
- [ ] Apagar en Formas de pago las tarjetas que ya no usa.
- [ ] Hacer copia de seguridad seguido (la campanita avisa a los 7 días).

## Cómo seguir en una sesión nueva
1. Leer `PROJECT_STATE.md`, `CLAUDE.md` y, según el cambio, `DATOS.md` y `DESIGN.md`.
2. Trabajar en la rama `claude/code-review-design-improvements-8dxp7k`, actualizada desde `main`.
3. Probar con Playwright (claro y oscuro, 390×844), revisar las capturas, buscar claves expuestas, hacer un PR en español y unirlo.
4. Al terminar algo importante, actualizar este archivo.
