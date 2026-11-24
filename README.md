# Ejercicios en silla

App web de un solo archivo ([ejercicios-en-silla.html](ejercicios-en-silla.html)) para que dos personas mayores hagan, desde el celular, un plan de ejercicios en silla de 28 días. El progreso de cada persona se guarda en el propio celular.

- Sin servidor, sin dependencias externas y sin conexión a internet: todo (HTML, CSS, JS e imágenes) está dentro del `.html`.
- Pensada para pantallas de celular: letra grande, botones grandes y un solo paso por pantalla.

> Este material no reemplaza la indicación de un médico o kinesiólogo. Los ejercicios marcados con aviso se hacen acompañados.

---

## Cómo se distribuye y se usa

1. Se manda el archivo `ejercicios-en-silla.html` al celular (por WhatsApp, mail o cable).
2. Se descarga y se abre **siempre con el mismo navegador** (por ejemplo Chrome). En lo posible, conviene abrirlo siempre desde el mismo lugar (la carpeta de Descargas), no desde la vista previa de WhatsApp.
3. Se elige quién va a hacer ejercicio, se toca **Empezar el día N** y se siguen los pasos. Al tocar **Terminé** en el último paso, el día queda marcado.

### Dónde se guarda el progreso (importante)

El progreso se guarda en el `localStorage` del navegador, bajo la clave `ejercicios-silla-v1`. Eso significa que:

- Queda **solo en ese celular y en ese navegador**. Si se abre el archivo con otra app u otro navegador, el progreso no aparece. Tampoco es seguro que aparezca si se abre una copia del archivo desde otra ubicación.
- Se pierde si se borran los datos del navegador, si se reinstala o si se cambia de teléfono.
- Mandar una versión nueva del `.html` **no borra** el progreso, siempre que se abra con el mismo navegador y se mantenga la misma clave (ver [Cosas que no hay que romper](#cosas-que-no-hay-que-romper)).

Si el navegador no permite guardar (por ejemplo, en modo incógnito), la app lo avisa en la pantalla inicial y deja hacer los ejercicios igual, pero sin guardar el progreso.

### Copia de seguridad

Desde la pantalla de inicio de cualquiera de las dos personas, abajo de todo, está **Copia de seguridad del progreso**. Esa pantalla tiene:

| Botón | Qué hace |
|---|---|
| **Copiar el texto de la copia** | Copia al portapapeles un texto JSON con el progreso de **las dos personas**. La idea es pegarlo en un chat de WhatsApp con uno mismo o en una nota. Si el navegador no deja copiar, selecciona el texto para copiarlo a mano. |
| **Descargar copia como archivo** | Descarga `ejercicios-progreso-AAAA-MM-DD.json`. Solo aparece si la página no está dentro de un iframe. |
| **Recuperar** | Toma el texto pegado en el segundo cuadro y lo guarda como progreso. **Reemplaza por completo** el progreso actual, no lo combina. |

Recomendaciones:

- Hacer una copia cada tanto (por ejemplo, al terminar cada semana) y siempre **antes de cambiar de celular o de borrar datos del navegador**.
- Para pasar el progreso a un celular nuevo: copiar el texto en el celular viejo, abrir el `.html` en el nuevo, ir a Copia de seguridad, pegarlo y tocar **Recuperar**.
- Al recuperar, solo se valida que el texto sea un objeto JSON. No se revisa su estructura, así que conviene pegar el texto completo y sin modificar.

---

## Cómo está armado

Todo vive en [ejercicios-en-silla.html](ejercicios-en-silla.html), en este orden:

| Parte | Qué contiene |
|---|---|
| `<head>` / `<style>` | Metadatos para celular (`viewport`, `theme-color`, modo app) y todo el CSS. Los colores del módulo actual se aplican con las variables `--mod` y `--modc`. |
| `MODULOS` | Los 4 módulos (A, B, C, D) con nombre, colores, texto de introducción, `aviso` opcional y 6 ejercicios cada uno. |
| `CALENTAMIENTO` | Lista de movimientos del calentamiento (igual todos los días). |
| `BICI_MIN` | Minutos de bici fija por día: matriz de 4 semanas × 7 días. |
| `IMGS` | Las 24 imágenes de los ejercicios (tomadas del manual de la Universidad de Georgia), en JPEG y codificadas en base64. **Es una sola línea de ~730 KB.** |
| `PERSONAS` | Las dos personas (`id`, `nombre`, `color` del botón). |
| `KEY` | La clave de `localStorage`. |
| Guardado | `loadAll`, `saveAll`, `prog` y `marcarDia`. |
| Plan | `modDeDia`, `kDeDia`, `dosisTexto`, `pasosSesion` y `diaSugerido`. |
| Vistas | Un `render()` que según `st.vista` dibuja una de estas pantallas: `quien` → `inicio` → `previa` → `sesion` → `fin`, más `copia`. |
| Temporizador | Cuenta regresiva para la bici fija (`arrancarTimer` y `pararTimer`). Usa `Date.now()` para no desfasarse y vibra al terminar si el celular lo permite. |

No hay frameworks: cada vista arma su HTML con template strings y asigna los `onclick` a mano. Los textos que vienen de datos pasan por `esc()`.

### El plan de 28 días

- **Semana = módulo.** Los días 1–7 son del Módulo A, los 8–14 del B, los 15–21 del C y los 22–28 del D (`modDeDia`).
- Cada sesión tiene: **bici fija → calentamiento → los 6 ejercicios del módulo → vuelta a la calma** (`pasosSesion`).
- **La dosis sube dentro de la semana** según el día de la semana `k` (de 0 a 6, calculado por `kDeDia`):

  | `k` (día de la semana) | Valor | Series (solo `reps`) |
  |---|---|---|
  | 0–2 | `lo` | 1 |
  | 3 | punto medio entre `lo` y `hi` | 1 |
  | 4 | punto medio entre `lo` y `hi` | 2 |
  | 5–6 | `hi` | 2 |

- Tipos de dosis (`ejercicio.dosis.tipo`):
  - `reps`: repeticiones. Con `lado: true` agrega "con cada lado" y `nota` se muestra entre paréntesis.
  - `seg`: sostener N segundos, `veces` veces (con `lado: true`, de cada lado).
  - `dur`: duración libre en segundos.
- **Día sugerido:** el primer día del 1 al 28 que todavía no se hizo. Igual se puede tocar cualquier día del calendario para hacerlo o repetirlo.
- Los ejercicios con `ayuda: true` y los módulos con `aviso` muestran un cartel amarillo para hacerlos acompañados.

### Formato de los datos guardados

```json
{
  "mingo": {
    "dias": {
      "1": { "veces": 2, "fechas": ["2026-11-20", "2026-11-22"] },
      "2": { "veces": 1, "fechas": ["2026-11-23"] }
    }
  },
  "choly": { "dias": {} }
}
```

- La clave de primer nivel es el `id` de `PERSONAS`.
- Un día aparece en `dias` solo si se completó al menos una vez.
- `fechas` guarda como máximo las últimas 20 fechas (formato `AAAA-MM-DD`, hora local).

Este mismo JSON es el que exporta e importa la pantalla de Copia de seguridad.

---

## Cómo actualizarlo

Después de cualquier cambio, conviene abrir el archivo en Chrome de escritorio con las herramientas de desarrollo en modo celular (`F12` → ícono de dispositivo) y recorrer al menos un día completo. Después se vuelve a mandar el `.html` al celular.

> **Cuidado con la línea de `IMGS`:** pesa unos 730 KB en una sola línea. Algunos editores se ponen lentos o la cortan al guardar. Conviene editar las demás partes sin seleccionar esa línea y no aplicar formateadores automáticos (Prettier, etc.) al archivo.

### Cambiar textos o dosis de un ejercicio

Editar el objeto correspondiente dentro de `MODULOS`. Por ejemplo:

```js
{ nombre: "Elevación de talones", img: 16, dosis: { tipo: "reps", lo: 8, hi: 15 },
  pasos: ["...", "..."],
  variante: "Más difícil: de pie, tomado del respaldo de la silla." },
```

`variante: ""` hace que no se muestre la sección de variante.

### Agregar o reemplazar una imagen

Las imágenes se referencian por índice (`img: 0` … `img: 23`) dentro del array `IMGS`. Para generar el base64 de una imagen nueva en PowerShell:

```powershell
$b64 = [Convert]::ToBase64String([IO.File]::ReadAllBytes("C:\ruta\imagen.jpg"))
"data:image/jpeg;base64,$b64" | Set-Clipboard
```

Después se pega como un elemento nuevo al final de `IMGS` (o reemplazando uno existente) y se usa ese índice en `img`. Conviene achicar antes la imagen (unos 800 px de ancho y JPEG con calidad media) para que el archivo no crezca demasiado.

### Cambiar la bici fija o el calentamiento

- `BICI_MIN[semana][k]` = minutos. Tiene que seguir siendo 4 × 7.
- `CALENTAMIENTO` es una lista de textos libres.

### Cambiar o agregar personas

Editar `PERSONAS`. El `id` es la clave con la que se guarda el progreso: **si se cambia el `id` de una persona existente, su progreso deja de aparecer** (sigue guardado bajo el `id` viejo). El `nombre` y el `color` se pueden cambiar sin problema. Si se agregan personas, también hay que actualizar el texto de `vCopia()`, que nombra a "Mingo y Choly".

### Cosas que no hay que romper

- **`KEY = "ejercicios-silla-v1"`**: si se cambia, el celular deja de ver todo el progreso anterior. Solo conviene cambiarla si el formato de datos cambia de forma incompatible, y en ese caso hay que migrar los datos viejos al cargar.
- **Cantidad de días y módulos**: `diaSugerido`, el calendario y los textos asumen 28 días = 4 módulos × 7 días. Para cambiar eso hay que tocar `modDeDia`, `diaSugerido`, `vInicio`, `vFin` y `BICI_MIN`.
- **Cantidad de ejercicios por módulo**: puede variar sin problema, porque la sesión se arma recorriendo `m.ejercicios`.

---

## Archivos del repositorio

| Archivo | Versionado | Descripción |
|---|---|---|
| `ejercicios-en-silla.html` | Sí | La app. |
| `README.md` | Sí | Este documento. |
| `.gitignore` | Sí | Excluye los PDF y la carpeta de imágenes. |
| `*.pdf` | No | Material de referencia (ver Fuentes). |
| `Tai chi imagenes/` | No | Imágenes de referencia de tai chi en silla. La app no las usa. |

---

## Fuentes

Los ejercicios y los módulos A–D se basan principalmente en el manual de la Universidad de Georgia. Las 24 imágenes de los ejercicios también salen de ese manual (ilustraciones de Krysia Haag). El resto del material se usó como referencia para el calentamiento, la vuelta a la calma y las recomendaciones de seguridad. Los PDF que están en la carpeta local son versiones en español de estos documentos.

1. Bell, M., Sellers, T. y Porter, K. N. *Senior Chair Exercises* [Ejercicios en silla para adultos mayores]. University of Georgia, Department of Foods and Nutrition. Ilustraciones de Krysia Haag; revisión de Bree Marsh. Contacto: Mary Ann Johnson, PhD. Adaptado a su vez de:
   - National Institute on Aging. *Exercise: A Guide from the National Institute on Aging*, 2001.
   - Tufts University y Centers for Disease Control and Prevention. *Growing Stronger: Strength Training for Older Adults*, 2002.
   - Archivo original: `Senior Chair Exercises.pdf`. Archivo local: `Ejercicios en silla Universidad de Georgia.pdf`.
2. Health Service Executive (Irlanda). *Easy Exercises: A Chair-Based Programme for Older People* [Ejercicios fáciles: un programa en silla para adultos mayores]. Publicado en enero de 2008, revisado en enero de 2010. Código de pedido HPM00496. <https://www.healthinfo.ie>
   - Archivo original: `easy-chair-based-exercises.pdf`. Archivo local: `Ejercicios faciles en silla.pdf`.
3. NHS (National Health Service, Reino Unido). *Exercises for Older People – Sitting* [Ejercicios para personas mayores: sentado]. <https://www.nhs.uk/exercises-for-older-people>
   - Archivo original: `NHS-exercises-for-older-people.pdf`. Archivo local: `NHS Ejercicios para personas mayores.pdf`.
4. Lifeline Canada. *Chair and Seated Exercises for Seniors* [Ejercicios en silla y sentados para adultos mayores]. Cita las Guías Canadienses de Actividad Física de la Sociedad Canadiense de Fisiología del Ejercicio. <https://www.lifeline.ca/en/resources/chair-exercises-for-seniors/>
   - Archivo original: `Chair-and-Seated-Exercises-For-Seniors.pdf`. Archivo local: `Ejercicios en silla para adultos mayores.pdf`.
