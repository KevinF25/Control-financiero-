# Mis cuentas (Spark Driver)

App personal de finanzas para un conductor de Walmart Spark. La usa en iPhone, instalada en la pantalla de inicio.

**Estado:** la app actual (`index.html`) se va a reemplazar por una app nueva en el servidor del dueño. De la actual solo se reutiliza la lógica de cálculo, documentada en `DATOS.md`. No empezar la app nueva hasta que el dueño lo pida.

## Reglas
- **Todo en español:** textos de la app, mensajes, commits y PRs.
- **Números honestos:** los saldos usan solo datos reales, nunca estimaciones. Los negativos van en rojo y con signo (−$).
- **No perder datos:** cualquier cambio debe leer los datos que ya existen. Antes de cambiar cómo se guardan, hacer una copia.
- **Git:** no hacer commit, push ni merge sin que el dueño lo pida.
- **Cambios grandes:** proponerlos primero, con maquetas si son visuales.
- **Al terminar:** probar y decir qué se probó.

## Seguridad
- **Claves:** nunca en el código ni en GitHub; van en variables de entorno del servidor.
- **Banco:** solo lectura. La app nunca mueve dinero.
- **Datos en el servidor:** inicio de sesión obligatorio, y cada usuario solo puede ver sus propios datos.
- **Antes de subir algo:** revisar que no haya claves expuestas.

## Diseño (al dueño le gusta; no cambiarlo sin que lo pida)
- **Estilo:** clara, honesta y tranquila. Los números son lo principal.
- **Letra:** Instrument Sans (400/500/600/700), con números tabulares.
- **Forma:** tarjetas redondeadas (22 px) e iconos de línea.
- **Como app de iPhone:**
  - pantallas con "‹ Atrás" y gesto de atrás;
  - hojas que suben desde abajo;
  - selectores nativos;
  - sin rebote ni barra de scroll.
- **Pantallas principales limpias:** Trabajo y Personal. Lo secundario va en el menú.
- **Modo claro y oscuro:** según el iPhone. El contraste debe ser de 4.5:1 o más, y los botones de 44 pt de alto o más.

| Color | Claro | Oscuro | Para qué |
| --- | --- | --- | --- |
| Fondo | `#f2f5f3` | `#0f1917` | Pantalla y campos |
| Tarjeta | `#ffffff` | `#17252a` | Tarjetas y hojas |
| Texto | `#17302b` | `#e3eeea` | Texto principal |
| Secundario | `#5a6d68` | `#8fa59f` | Fechas, notas |
| Borde | `#d5ded9` | `#2b3f3b` | Líneas |
| Verde | `#1d6b58` | `#5fc4a5` | Dinero a favor, acción principal |
| Verde suave | `#e1efe9` | `#183a31` | Fondo de iconos y avisos |
| Rojo | `#b3261e` | `#ff8a80` | Negativos, deudas, borrar |
| Tarjeta principal | `#185c4b` | `#1f4a3e` | "Dinero disponible", con texto blanco |

**Un color = un significado:** verde es dinero o acción principal, rojo es negativo o borrar, gris es información secundaria.
