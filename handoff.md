# handoff.md — Cómo organizar un proyecto con Claude para gastar pocos tokens y no perder calidad

Guía para llevar a otro proyecto lo que funcionó en **Mis cuentas** (app personal de finanzas en una PWA de un solo archivo). Está escrita en general: lo específico de esa app aparece marcado como *ejemplo*.

No incluye datos personales, claves ni nombres de cuentas. Antes de copiarla, reemplazar los ejemplos por los del nuevo proyecto.

---

## 1. Idea central

Claude gasta tokens en tres cosas:
1. **Leer archivos grandes** completos (lo más caro).
2. **Explorar** para entender el proyecto cada vez que empieza una sesión.
3. **Conversaciones largas** con temas mezclados, porque cada mensaje reenvía todo el contexto.

La solución no es escribir menos, sino **dejar el conocimiento en archivos cortos y dirigidos**, y pedir a Claude que lea solo lo necesario.

---

## 2. Estructura mínima de archivos

| Archivo | Para qué | Tamaño objetivo |
| --- | --- | --- |
| `CLAUDE.md` | Reglas de trabajo, comandos, tabla "qué leer según la tarea" | 3–5 KB |
| `PROJECT_STATE.md` | Qué es, estado, pendientes, cómo seguir en una sesión nueva | 5–7 KB |
| `DATOS.md` (o `ARQUITECTURA.md`) | Modelo de datos, cálculos, reglas de negocio, por secciones | Lo que haga falta, pero **dividido en secciones** |
| `DESIGN.md` (si hay interfaz) | Colores, tipografía, componentes, comportamiento | 5–8 KB |

**Regla:** cada dato vive en un solo archivo. Si se repite, se quita de uno y se deja una referencia.

### 2.1 `CLAUDE.md` — plantilla

```markdown
# CLAUDE.md — <nombre del proyecto>

<Una o dos líneas: qué es y quién lo usa.>
Estado y pendientes: `PROJECT_STATE.md`. Datos y cálculos: `DATOS.md`. Diseño: `DESIGN.md`.

## Qué leer según la tarea
| Tarea | Leer |
| --- | --- |
| Pendientes y estado | `PROJECT_STATE.md` (sección de pendientes) |
| Datos o cálculos | `DATOS.md`, solo la sección que toque |
| Diseño | `DESIGN.md` |
| Cambio de código | Esta página + `grep -n "nombre" <archivo>`; leer solo ese tramo |

## Comandos
<cómo correr, cómo probar, cómo revisar sintaxis>

## Reglas de código
- Leer el código cercano antes de cambiar y copiar su estilo.
- Lo mínimo para lo pedido. No "mejorar" código vecino: avisar.
- Si el pedido no está claro, preguntar o decir qué se va a suponer.
- Terminar con prueba y decir qué se probó.
- <reglas propias: datos guardados, lógica protegida, nombres prohibidos, etc.>

## Preferencias de la persona
- <idioma, estilo visual que no se cambia sin pedirlo, cómo se quiere el comportamiento>

## Flujo con git
- Rama de trabajo, commits en el idioma del equipo, cómo se unen los cambios.

## Seguridad
- <claves fuera del código; qué se revisa antes de terminar>
```

### 2.2 `PROJECT_STATE.md` — plantilla

```markdown
# PROJECT_STATE.md — <nombre>

Estado al <fecha>. Último cambio unido: <PR/commit>.

## 1. Qué es y cómo está
## 2. Arquitectura (tabla corta: parte → cómo está hecha)
## 3. Lo que ya funciona (por áreas, una línea por función)
## 4. Pendientes
- [ ] Por revisar con la persona
- [ ] Ideas propuestas, sin aceptar
- [ ] Proyectos grandes (con la condición que puso la persona)
- [ ] Cosas que la persona hace fuera del código
## 5. Cómo seguir en una sesión nueva
1. Leer `CLAUDE.md`. Según el cambio, la sección de `DATOS.md` o `DESIGN.md`.
2. Trabajar en la rama de trabajo.
3. Probar, revisar, buscar claves expuestas, unir.
4. Al terminar algo importante, actualizar este archivo.
```

Mantener "Lo que ya funciona" en una línea por función. El detalle técnico va a `DATOS.md`, no aquí.

### 2.3 `DATOS.md` — cómo ordenarlo

- **Por secciones con viñetas**, nunca un párrafo largo. Un párrafo de 20 KB no se puede leer por partes.
- Una sección por área (por ejemplo: cuentas, turnos, metas, deudas, calendario, categorías, copia de seguridad, código).
- Cada campo guardado con su forma: `campo = {a, b, c?}`. Los opcionales llevan `?`.
- Cada regla con una nota de qué pasa con datos viejos.
- Una sección "Código y navegación" para quien cambia el código: funciones clave, IDs, reglas de la interfaz.
- Cuando cambie un dato o una regla, se actualiza en el mismo cambio.

---

## 3. Reglas que más ahorraron tokens

1. **Buscar antes de leer.** `grep -n` para encontrar la función o el ID. Leer después solo ese tramo (`offset` y `limit`).
2. **Una tarea por sesión.** Al cambiar de tema, sesión nueva. La nueva empieza leyendo `PROJECT_STATE.md`.
3. **Decir el punto exacto.** "En la función `X`, línea N" cuesta menos que "revisa cómo funciona".
4. **Cambios con `Edit`, no reescribir archivos.** Reescribir un archivo grande con `Write` gasta mucho en salida.
5. **Pruebas dirigidas.** Revisar la pantalla que cambió, en los dos modos, y revisar errores de carga. No repetir todo el conjunto de pruebas si nada más cambió.
6. **Agente de exploración para barridos amplios.** Devuelve solo la conclusión; no trae todos los archivos a la conversación principal.
7. **Archivos ignorados.** Usar `.claudeignore` para excluir dependencias, compilados, capturas, copias de seguridad, datos personales y secretos.
8. **Skills pesadas, solo si se usan.** Una skill con cientos de páginas de referencia puede costar más que todo el proyecto. Si no se usa a diario, quitarla y dejar en `DESIGN.md` las reglas que de verdad se aplican (contraste, tamaño de toque, etc.).
9. **Un solo lugar por dato.** Lo que se repite entre archivos se quita de uno.
10. **Mapa de código (opcional).** Un archivo corto generado con `grep` (función → línea, ID → línea), que se regenera al cambiar el código.

### Lo que no conviene recortar
- Reglas de seguridad y de datos guardados.
- Detalle de cálculos y de la forma de los datos cuando una sesión nueva debe poder cambiarlos sin romper nada.
- Las preferencias de la persona (estilo, idioma, comportamiento de la interfaz).

---

## 4. Pasos para aplicarlo en otro proyecto

1. **Inventario.** Listar archivos y tamaños (`wc -c`). Los archivos grandes que se leen completos son el primer objetivo.
2. **Crear `.claudeignore` y revisar `.gitignore`.** Excluir lo generado, los secretos y los datos personales.
3. **Escribir `CLAUDE.md`** con la plantilla de la sección 2.1. Incluir la tabla "qué leer según la tarea".
4. **Escribir `PROJECT_STATE.md`** con la plantilla de la sección 2.2. Pasar el historial de conversaciones a estado actual: lo que funciona y lo pendiente.
5. **Dividir los datos o la arquitectura** en secciones (sección 2.3). Quitar duplicados con `CLAUDE.md`.
6. **Verificar contra el código.** Buscar los nombres de funciones, campos e IDs que aparecen en la documentación. Lo que no exista, corregirlo.
7. **Revisar secretos** en el código y en el historial de git antes de subir.
8. **Medir.** Comparar el tamaño de los documentos antes y después, y el número de archivos grandes que una sesión nueva necesita leer.
9. **Probar una tarea real** en una sesión nueva. Si Claude necesita leer más de lo debido, el documento que falta es el que hay que mejorar.

---

## 5. Frase para arrancar una sesión nueva

> Lee `CLAUDE.md` y la sección de pendientes de `PROJECT_STATE.md`. Vamos a hacer solo <tarea>. Busca con `grep` antes de leer archivos grandes. Al terminar, di qué probaste.

---

## 6. Checklist rápido

- [ ] `CLAUDE.md` con tabla "qué leer según la tarea"
- [ ] `PROJECT_STATE.md` con pendientes y "cómo seguir"
- [ ] Datos y cálculos divididos en secciones, sin duplicados
- [ ] `.claudeignore` creado
- [ ] Skills y referencias pesadas que no se usan, quitadas
- [ ] Nombres de funciones, campos e IDs verificados contra el código
- [ ] Revisión de secretos hecha y contada
- [ ] Una tarea real probada en sesión nueva
