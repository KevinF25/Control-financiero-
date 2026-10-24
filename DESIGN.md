# DESIGN.md — Mis cuentas

Cómo se ve y cómo se comporta la app. Sacado del código real (`index.html`). Toda pantalla o cambio nuevo debe seguirlo; si algo nuevo no cabe aquí, se propone primero al dueño.

## Idea general
- Una app de iPhone para llevar el dinero de un conductor de Spark: **clara, honesta y tranquila**. Los números son lo principal; el adorno, lo mínimo.
- **Al dueño le gusta este diseño.** Los colores verdes, la letra Instrument Sans, las tarjetas redondeadas y los iconos de línea no se cambian sin que él lo pida.
- Se comporta como una app nativa:
  - pantallas nuevas con ‹ y gesto de atrás;
  - hojas que suben desde abajo;
  - sin rebote ni barra de scroll;
  - selectores del iPhone.

## Colores
Siempre con variables CSS; nunca escribir un color directo en un componente.

| Variable | Claro | Oscuro | Para qué | Contraste (claro / oscuro) |
| --- | --- | --- | --- | --- |
| `--bg` | `#f2f5f3` | `#0f1917` | Fondo de la pantalla y de los campos | — |
| `--card` | `#ffffff` | `#17252a` | Tarjetas, hojas | — |
| `--text` | `#17302b` | `#e3eeea` | Texto principal | 14.1 / 13.3 sobre tarjeta |
| `--muted` | `#5a6d68` | `#8fa59f` | Texto secundario, fechas, notas | 5.5 / 6.0 |
| `--border` | `#d5ded9` | `#2b3f3b` | Líneas y bordes | — |
| `--accent` | `#1d6b58` | `#5fc4a5` | Verde de la app: botones principales, montos a favor, iconos | 6.4 / 7.4 |
| `--onaccent` | `#ffffff` | `#0f1917` | Texto sobre el verde | 6.4 / 8.5 |
| `--accbg` | `#e1efe9` | `#183a31` | Fondo verde claro (iconos, avisos, toque) | verde sobre él: 5.4 / 5.9 |
| `--bad` | `#b3261e` | `#ff8a80` | Negativos, deudas, borrar | 6.5 / 6.9 |
| `--hero` | `#185c4b` | `#1f4a3e` | Fondo de la tarjeta verde "Dinero disponible" (texto blanco 7.9 / 10.0; negativos `#ffd6d1`) | — |
| `--glass` | `rgba(242,245,243,.82)` | `rgba(15,25,23,.82)` | Barra de arriba translúcida | — |

Reglas:
- **Un color = un significado.**
  - Verde: dinero que tienes o acción principal.
  - Rojo: negativo, deuda o borrar.
  - Gris (`--muted`): información secundaria.
- **Honestidad:** los negativos van en rojo **y** con signo (−$). Nunca se suavizan.
- **Contraste:**
  - Texto normal: mínimo 4.5:1 en claro y en oscuro.
  - Texto grande o en negrita: mínimo 3:1.
  - Al agregar un color, medirlo en los dos modos.
- **Modo oscuro:** lo pone el iPhone. La app no tiene un interruptor propio de claro/oscuro.
- Numerito de la campanita: `#e5484d` con texto blanco.

## Letra
- **Instrument Sans** (Google Fonts, 400/500/600/700), con `system-ui` de respaldo.
- Los números llevan `font-variant-numeric: tabular-nums`, para que los montos queden alineados.

| Uso | Tamaño | Peso |
| --- | --- | --- |
| Monto grande (Dinero disponible, Te quedó por hora) | 44 px | 700, `letter-spacing:-.035em`; centavos al 55 % (`bigM`) |
| Título de pestaña (Trabajo/Personal), con la fecha arriba en 12 px mayúsculas | 30 px | 700, `letter-spacing:-.025em` |
| Título de pantalla interna | 22 px | 700 |
| Título de sección fuera de tarjeta (`.sech`) | 20 px | 700, 16 px arriba y 4 px abajo |
| Título de tarjeta (`h2`) | 17 px | 600 |
| Texto normal, renglones | 15–16 px | 400 |
| Montos en renglones | 15–16 px | 700 (`<b>`) |
| Texto secundario (`small`, `.muted`) | 13–15 px | 400, `--muted` |
| Etiquetas de campos (`label`) | 13 px | 400, `--muted` |
| Mínimo absoluto | 11 px | solo el numerito de la campanita |

- Los campos de texto miden **16 px o más**; si son menores, el iPhone hace zoom al escribir.
- Las mayúsculas van como en una oración ("Ver cuentas y transferencias"), no en tipo título.

## Espacios y formas
- Margen de la pantalla: 14 px a los lados.
- **Tarjeta (`.card`):**
  - fondo `--card`, radio 22 px, relleno 16 px;
  - 12 px de separación entre tarjetas;
  - sombra suave en claro (`--shadow`) y ninguna en oscuro.
- **Botón principal (`.btn`):**
  - fondo verde, texto `--onaccent`;
  - radio 14 px, alto mínimo 50 px, letra 17 px en 600.
- **Botón secundario (`.btn2`):**
  - borde `--border`, fondo de tarjeta;
  - radio 999 px (píldora) en tarjetas de resumen.
- **Campos (`input`, `select`):**
  - fondo `--bg`, borde `--border`;
  - radio 8 px, alto mínimo 42 px.
  - Las fechas llevan `appearance:none`.
- **Iconos (`.ic`):**
  - de línea, trazo 1.8–2, en el verde de la app;
  - dentro de un cuadrito de 36 px con radio 10 px y fondo `--accbg`.
  - Todos del mismo estilo; nunca emojis como iconos.
- **Toques:** todo lo que se toca mide **al menos 44 px de alto**. Si se ve más chico, se agranda el área de toque, no el dibujo.

## Componentes
- **Barra de arriba (`.top`):**
  - fija y translúcida (`--glass`);
  - título a la izquierda (con ‹ en pantallas internas);
  - campanita y ··· a la derecha.
- **Barra de abajo (`.tabs`):**
  - translúcida con desenfoque: Trabajo · (+) · Personal;
  - la pestaña activa va en verde y en 600;
  - (+) es un círculo verde de 52 px con sombra.
- **Renglones (`.row`, `.ev`, `.arow`):**
  - línea superior `--border`;
  - texto a la izquierda, con su `small` debajo;
  - monto en negrita a la derecha.
  - Si abren algo, llevan "›" y se toca el renglón entero.
  - En turnos, llenadas y gastos: deslizar a la izquierda muestra **Borrar** en rojo (`rowTap`).
- **Segmentos (Hoy / Semana / Mes):** botones juntos; el elegido va verde lleno y el resto con borde.
- **Interruptor (`.sw`):**
  - 51×31, verde cuando está encendido;
  - siempre en un renglón con su texto a la izquierda (`.swrow`);
  - nunca casillas sueltas.
- **Hoja desde abajo (`ask`):**
  - radio 22 px arriba y agarradera;
  - "Cancelar" a la izquierda y título al centro;
  - campo grande con borde verde y botón principal abajo;
  - teclado de números para montos.
- **Confirmar (`sure`):**
  - opciones desde abajo;
  - la acción va arriba (en rojo si borra) y "Cancelar" aparte y en negrita.
- **Avisos breves (`#msg`):** píldora verde clara sobre la barra de abajo. Solo dan información, nunca preguntan.
- **Nunca** se usan `alert()`, `confirm()` ni `prompt()` del navegador.

- Botón secundario de acción ("Evaluar oferta"): fondo `--accbg` y texto verde.
- Hoy/Semana/Mes: selector estilo iPhone (fondo gris, la opción elegida en blanco).

## Movimiento
- **Animaciones:**
  - cortas: unos 0.2 s al entrar a una pantalla y 0.25 s en las hojas;
  - con `prefers-reduced-motion` se quitan.
- **Gesto de atrás:** no se anima, porque Safari ya lo hace. Al regresar, la pantalla vuelve a donde estaba (`history.state.y`).
- **Sin rebote:** `overscroll-behavior:none`.

## Pantallas principales
- **Trabajo y Personal se mantienen limpias:** arriba, lo más importante (el dinero); abajo, lo secundario; lo demás va en el menú ···. **Sin accesos rápidos.**
- **Diseño moderno:**
  - "Dinero disponible" es la única tarjeta verde llena (`#acard.mhero`, fondo `--hero`, texto blanco). No repetir ese estilo en otras tarjetas.
  - Títulos de sección fuera de las tarjetas (`.sech`: "Resumen", "Próximos pagos" con "Ver todos"). Se esconden solos si su tarjeta está oculta.
  - Balance y Deuda son dos tarjetas separadas.
  - Estados como etiquetas (`.pvr small`): "Hoy" y "Registrado" en verde, "Pendiente" en gris, "Atrasado" en rojo.
  - La barra de abajo flota como píldora separada de los bordes (en modo Trabajo fijo solo queda el (+)).
- **Orden de Personal:**
  1. Dinero disponible (solo cuentas de uso diario; botón "Ver cuentas y transferencias").
  2. Balance del mes y Deuda total.
  3. Próximos pagos ("7 días"; máximo 3).
  4. Apartados (se esconde sin metas).

## Antes de subir un cambio visual
1. Capturas a 390×844 en claro y oscuro (Playwright).
2. Medir contraste y tamaño de toque (skill `apple-design`).
3. Que se vea como el resto de la app: mismos colores, radios, letra e iconos.
