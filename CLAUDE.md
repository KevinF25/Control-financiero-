# CLAUDE.md — Mis cuentas (Spark Driver)

App personal de finanzas para un conductor de Walmart Spark. El dueño la usa en iPhone, instalada en la pantalla de inicio.
## Qué leer según la tarea (para ahorrar tokens)
| Tarea | Leer |
| --- | --- |
| Pendientes y estado actual | `PROJECT_STATE.md` (sección 4) |
| Datos, cálculos, copia de seguridad | `DATOS.md`, solo la sección que toque |
| Diseño, colores, pantallas | `DESIGN.md` |
| Cambio de código | Esta página + `grep -n "nombre" index.html`; leer solo ese tramo, nunca el archivo completo (237 KB) |

## Comandos
No hay `npm` ni proceso de compilación: se edita `index.html` y listo.
```bash
python3 -m http.server 8794          # servir la app (desde la raíz)
# abrir http://localhost:8794/index.html
node prueba.js                       # pruebas con Playwright (global; Chromium ya instalado)
```
- Las pruebas son scripts propios de Playwright, fuera del repo (en el scratchpad). Hacerlas a **390×844**, en **claro y oscuro**, con la copia real del dueño en `localStorage`, y revisar las capturas.
- Usar siempre el servidor local, nunca `file://`: con `file://` se pierde el `localStorage` al recargar.
- Cargar la página y comprobar que no haya `pageerror`: es la revisión de sintaxis.
- Al publicar, la versión nueva llega sola (`sw.js` intenta primero la red). El dueño solo cierra y abre la app.

## Tecnologías
- **PWA de un solo archivo:** `index.html` (HTML + CSS + JS, sin frameworks ni librerías), `sw.js`, `manifest.webmanifest` e iconos.
- **Datos:** `localStorage`, clave `sd_tracker_v1`. Copia de seguridad en JSON (importar une con `mergeIn`).
- **Letra:** Instrument Sans (Google Fonts). Modo claro y oscuro según el iPhone.
- **Informe de Spark (.xlsx):** lector propio (ZIP + `DecompressionStream` + `DOMParser`).
- **Futuro (no empezado):** SimpleFIN (banco, solo lectura), Supabase (nube + inicio de sesión) y Vercel (funciones).

## Reglas de código
- **Leer antes de cambiar:** mirar el código cercano y copiar su estilo (nombres cortos, líneas largas, poca densidad de comentarios). Los comentarios van en español.
- **Comentarios:** dentro de una línea con más código, usar `/* … */`, nunca `//`, porque `//` corta el resto de la línea.
- **No romper datos guardados:** `norm()` debe conservar todo campo nuevo, y los cambios deben leer los datos viejos.
- **Lógica de cálculo:** va entre `//LOGIC-START` y `//LOGIC-END`; no tocarla salvo que el cambio lo pida.
- **Ventanas:** nunca `prompt()`, `confirm()` ni `alert()`. Usar `await ask({t,q,v,num,hint,ok,neg})` y `await sure({t,m,ok,red})`.
- **Navegación:** `go(v,true)` abre una pantalla nueva con ‹ y gesto de atrás. Las listas usan `rowTap` y `evTap`.
- **Estilos:**
  - Colores solo con variables CSS (`--accent`, `--muted`, `--bad`…).
  - Contraste de 4.5:1 o más.
  - Botones de 44 pt de alto o más.
  - Probar los dos modos.
- **Números honestos:** el saldo usa solo datos reales y los negativos van en rojo con signo.
- **Cambios quirúrgicos:**
  - El código mínimo para lo pedido; no "mejorar" código vecino, solo avisar.
  - Si el pedido no está claro, preguntar o decir qué se va a suponer.
  - Terminar con prueba y decir qué se probó.

## Preferencias del dueño
- **Español:** todo en español (textos, mensajes, commits y PRs).
- **Diseño actual:** le gusta (verdes, Instrument Sans, tarjetas redondeadas, iconos de línea). No cambiarlo sin que lo pida.
- **Como app de iPhone:**
  - pantallas con "‹ Atrás" y gesto de atrás;
  - hojas desde abajo y selectores nativos;
  - sin rebote ni barra de scroll.
- **Pantallas principales limpias:** Trabajo y Personal. Lo secundario va en el menú ···. Sin "Accesos rápidos".
- **Cambios grandes:** proponer primero con maquetas o capturas. Los arreglos pequeños se pueden hacer directo.

## Flujo con git
- **Rama:** `claude/code-review-design-improvements-8dxp7k`, actualizada desde `main`.
- **Cada cambio:** commit en español → push → PR → unir (el dueño lo autorizó).
- **Al terminar algo importante:** actualizar `PROJECT_STATE.md` (y `DATOS.md` si cambian datos o cálculos).

## Seguridad (obligatorio, para el banco y la nube)
- **Claves:** nunca en el código ni en GitHub; van en las variables de entorno de Vercel. La clave de SimpleFIN solo la usa la función de Vercel.
- **Supabase:**
  - RLS en todas las tablas;
  - `service_role` nunca en la app;
  - inicio de sesión obligatorio;
  - protección de contraseñas filtradas.
- **Tablas nuevas en `public`:** desde el 30 oct 2026, en la misma migración, `grant select, insert, update, delete on table public.<tabla> to authenticated;` (nunca `anon`) + RLS + políticas.
- **Banco:** solo lectura; la app nunca mueve dinero.
- **Antes de terminar:** revisar el código y el historial de git buscando claves expuestas, y contarle al dueño qué se revisó.
