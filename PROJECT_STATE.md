# PROJECT_STATE.md — Mis cuentas (Control financiero · Spark Driver)

Estado al **24 de octubre de 2026**. Último PR unido: #67 (23 oct).
Reglas de trabajo en `CLAUDE.md`. Datos y cálculos en `DATOS.md`. Diseño en `DESIGN.md`.

---

## 1. Qué es y cómo está

- App personal de finanzas para un conductor de **Walmart Spark Driver**. El dueño la usa en su **iPhone**, instalada en la pantalla de inicio.
- Lleva turnos, gasolina, propinas, cuentas, deudas, metas de ahorro, pagos recurrentes y gastos personales. Dice cuánto le queda de verdad después del carro.
- **Estado:** funciona y se usa a diario con datos reales. Todo está en `main`.
- **Flujo:** rama `claude/code-review-design-improvements-8dxp7k`. Cada cambio: commit → push → PR → unir (autorizado por el dueño).

## 2. Arquitectura

| Parte | Cómo está hecho |
| --- | --- |
| App | PWA de un solo archivo: `index.html` (~1,350 líneas; HTML + CSS + JS), sin frameworks ni librerías |
| Sin conexión | `sw.js`: intenta la red primero y, si no hay, usa la copia guardada. No hace falta cambiar versión al publicar |
| Instalación | `manifest.webmanifest` + `icon-180/192/512.png` |
| Datos | `localStorage`, clave `sd_tracker_v1`. Sin servidor ni nube |
| Copia de seguridad | Exportar/importar JSON; importar une con `mergeIn` |
| Informe de Spark | Lector propio de `.xlsx` (ZIP + `DecompressionStream` + `DOMParser`) |
| Letra | Instrument Sans (Google Fonts) |
| Pruebas | Playwright (Chromium), 390×844, claro y oscuro, con servidor local (`python3 -m http.server`) y la copia real del dueño (no está en el repo) |
| Diseño | Skill `apple-design` (`.claude/skills/apple-design`) para auditar; `DESIGN.md` con colores, letra y componentes |
| Repositorio | GitHub `KevinF25/Control-financiero-` |

## 3. Lo que ya funciona

**Trabajo (Spark)**
- Turnos: empezar y terminar con el odómetro; al terminar se piden las tres cifras de Spark (total, propinas confirmadas y por confirmar).
- Propinas: entran a la cuenta 24 h después de la mitad del turno; mientras, "por llegar".
- Turno guardado: lo ganado, lo que te quedó, "Aparta hoy" (metas y reserva del carro).
- Llenadas de gasolina (MPG y costo por milla), reserva del carro por milla y estadísticas Hoy/Semana/Mes.
- Meta diaria automática con barra de avance, y evaluar oferta ($/milla, $/hora, vuelta al Walmart).
- Informe de Spark (.xlsx): importa viajes, propinas e incentivos con vista previa y deshacer. Nunca toca millas, horas ni odómetro.
- Mejores días y horas (solo marca "mejor" con datos suficientes).

**Tu dinero**
- Cuentas: Spark, Personal, Ahorro y las que se agreguen. Ajustar al banco, transferir (con comisión instantánea de One) y editar datos.
- Dinero disponible: solo cuentas de uso diario.
- Deudas: tarjetas y planes a cuotas (Affirm, Apple cuotas), pagos, "Actualizar con el banco", siguiente paso según APR, simulador, evolución e intereses. Patrimonio.
- Metas y apartados: por % de lo ganado, $ por semana o "lo que pueda". Cuenta propia opcional. Bolsillos de OnePay. "Usé…" y "Pagué taxes".
- Calendario de cobros: pagos recurrentes, cuotas, fechas de tarjetas y sueldo ("Me llegó ✓").
- Gastos personales: categorías con icono, gráfica, formas de pago que se pueden apagar.
- Balance del mes: solo datos reales.

**Avisos y navegación**
- Campanita con numerito: pagos a deudas, cobros próximos o atrasados, copia de seguridad, tarjetas, propinas, apartados y avisos propios. Cada aviso tiene "Hecho ✓" y "Recordar mañana".
- Menú ··· por temas (Tu dinero, Trabajo (Spark), Ajustes). Barra de abajo flotante: Trabajo · (+) · Personal.
- Navegación de iPhone: ‹ Atrás y gesto de atrás, hojas desde abajo, nada de `prompt`/`confirm`/`alert`.

**Diseño**
- Modo claro y oscuro. Contraste medido: 0 fallas en 21 pantallas.
- Diseño moderno en Trabajo y Personal (ver `DESIGN.md`).

## 4. Pendientes

**Por revisar con el dueño**
- [ ] **Contador del turno** (en prueba): si no sirve, quitarlo (`active.lv`, "La acepté ✓", "Llevas en este turno"). Si sirve, quitar "prueba" de `DATOS.md`.
- [ ] **En su iPhone real:** que la barra flotante no choque con la rayita de inicio; que las hojas queden encima del teclado; que el gesto de atrás no brinque.
- [ ] **Raya vertical** que marcó en una captura junto a "Apartados". No quedó claro qué quería: preguntar.
- [ ] **Fechas en "Próximos pagos":** salen como "vie, oct 23" (formato del teléfono), no "vie, 23 de oct". Ajustar si le molesta.

**Ideas propuestas, sin aceptar**
- [ ] Skill propia "probar la app" (las pruebas de siempre en un paso).
- [ ] Seguir el texto grande del iPhone (solo si lo usa).
- [ ] Que la propina entre poco a poco al saldo de Spark, como llega en realidad.
- [ ] Autosave para los bolsillos.
- [ ] Esconder cuentas en Transferir.
- [ ] Pantalla "Taxes del año".
- [ ] Anotar solo el interés de la cuenta de ahorro (APY); hoy se hace a mano con "Ajustar al banco".

**Proyecto grande (el dueño dijo "para el futuro")**
- [ ] Conexión con el banco por **SimpleFIN**, solo lectura, desde una función de Vercel.
- [ ] Nube con **Supabase**: inicio de sesión obligatorio, RLS, GRANT solo para `authenticated` y protección de contraseñas filtradas.
- [ ] Revisar claves expuestas antes de cada entrega (reglas en `CLAUDE.md`).

**Lo que el dueño hace en su app (no es código)**
- [ ] Volver a guardar el turno del 20 oct (Editar → Guardar) para corregir la propina del 19 oct (+$12).
- [ ] Crear las cuentas de los bolsillos (Taxes, Colchón) y ligarlas a sus metas; poner a Emergencias el objetivo de $500.
- [ ] Marcar "Apple Card ahorro" como cuenta de ahorro, si es ahorro.
- [ ] Apagar en Formas de pago las tarjetas que ya no usa.
- [ ] Hacer copia de seguridad seguido (la campanita avisa a los 7 días).

## 5. Cómo seguir en una sesión nueva

1. Leer `CLAUDE.md`. Según el cambio, `DATOS.md` y `DESIGN.md`.
2. Trabajar en la rama `claude/code-review-design-improvements-8dxp7k`, actualizada desde `main`.
3. Probar con Playwright (claro y oscuro, 390×844), revisar capturas, buscar claves expuestas, hacer un PR en español y unirlo.
4. Al terminar algo importante, actualizar este archivo.
