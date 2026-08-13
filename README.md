# Encuesta posventa — Jabugo Almacén Gourmet

Sitio estático de una sola página: el cliente escanea un QR en la caja y
valora la compra. Campos: **estrellas** (obligatorio) y **teléfono**
(obligatorio); qué compró, qué mejorar y nombre son opcionales. Al enviar, la
respuesta se guarda **directo y en privado en una Google Sheet tuya** — no
pasa nada más.

**No hay ningún contacto con WhatsApp.** El cliente no recibe ninguna
respuesta, confirmación ni mensaje: la encuesta no abre WhatsApp, no manda
nada por WhatsApp, no queda ningún rastro ahí. Es un sitio 100% aparte.

No hay backend propio: todo corre en el navegador del cliente + un Google Apps
Script gratuito que escribe en tu planilla.

## Estructura

```
jabugo-posventa/
  index.html              -> la encuesta (esto es lo que se publica)
  apps-script/Code.gs      -> script que escribe en tu planilla de Google Sheets
  qr/generar-qr.html       -> herramienta offline para generar el QR imprimible
  qr/qrcode.lib.js          -> librería QR (offline, sin dependencias externas)
  _dev-server.ps1          -> solo para pruebas locales, no hace falta subirlo
```

Solo `index.html` necesita estar publicado en la web. `apps-script/` se despliega
aparte (en Google). `qr/` es una herramienta de uso interno, no hace falta
publicarla. `_dev-server.ps1` se usó únicamente para probar el sitio en este
entorno de desarrollo; se puede borrar o ignorar sin problema.

---

## Paso 1 — Google Sheets (obligatorio)

A diferencia de una versión anterior de este proyecto, **esto ya no es un
respaldo opcional: es el único lugar donde queda la información.** Si no se
configura, la encuesta no tiene forma de guardar nada.

Se usa la planilla que ya armaste: **"Postventa – Almacén Gourmet Jabugo"**
(`https://docs.google.com/spreadsheets/d/1zPF8i7yyR6hD-TMR_w7MPXquxgbYENGxqPQ6kOy5sKE`),
con las columnas: Fecha, Cliente, Teléfono / WhatsApp, Producto(s)
comprado(s), Estrellas (1-5), Qué le gustó (emoción / experiencia), Qué se
puede mejorar, Comentario completo, Estado. El script ya está escrito para
esa planilla puntual (el ID está fijo en `Code.gs`).

> La versión actual del formulario solo pide estrellas (obligatorio),
> teléfono (obligatorio), productos, "qué podemos mejorar" y nombre
> (opcionales) — así que las columnas **Qué le gustó** y **Comentario
> completo** van a quedar vacías en las respuestas nuevas. Quedaron en la
> planilla por si en algún momento querés volver a pedirlas; no hace falta
> tocar nada si no.

1. **Borrá la fila de ejemplo** ("Juan Pérez... EJEMPLO - borrar esta fila")
   antes de usar la planilla en serio — si no, va a distorsionar el promedio
   y las estadísticas.
2. Abrí la planilla → menú **Extensiones → Apps Script**.
3. Borrá el contenido del editor y pegá el contenido completo de
   [`apps-script/Code.gs`](apps-script/Code.gs).
4. Guardá el proyecto (ícono de disquete).
5. **Implementar → Nueva implementación**.
   - Tipo: **Aplicación web**.
   - Ejecutar como: **Yo** (tu cuenta de Google).
   - Quién tiene acceso: **Cualquier usuario**. (Esto es solo para que la
     página pueda *enviar* datos — nadie puede *leer* la planilla sin que vos
     se la compartas. La planilla en sí sigue siendo privada.)
6. Autorizá los permisos que pida Google (es tu propio script, sobre tu
   propia planilla).
7. Copiá la **URL de la aplicación web** que te da al terminar (termina en
   `/exec`).
8. Abrí `index.html`, buscá esta línea cerca del principio del `<script>`:

   ```js
   const GOOGLE_SHEETS_WEBAPP_URL = "";
   ```

   y pegá la URL ahí adentro:

   ```js
   const GOOGLE_SHEETS_WEBAPP_URL = "https://script.google.com/macros/s/AKfycb.../exec";
   ```

9. Para confirmar que quedó bien conectado, pegá esa misma URL en la barra del
   navegador: tiene que mostrar el texto "Jabugo posventa — Apps Script activo...".
10. Mandá una valoración de prueba desde la encuesta y confirmá que aparece la
    fila en la planilla.

Al conectarse, el script crea automáticamente dos pestañas nuevas además de la
tuya:

- **Detalle productos** (oculta): una fila por cada producto mencionado en
  cada valoración — es la que permite calcular el promedio por producto. No
  hace falta tocarla nunca a mano.
- **Resumen**: se crea una sola vez, con fórmulas que se actualizan solas —
  promedio general de estrellas, cantidad de reseñas por cantidad de
  estrellas, y el ranking de productos de mejor a peor valorados. La podés
  mirar en cualquier momento, no hace falta pedirme nada.

> Nota de privacidad: la planilla es tuya, privada, y solo vos (o quien
> invites) tiene acceso. Ningún dato se comparte con terceros ni con el
> cliente que la completó.

---

## Paso 2 — Hostear el sitio

Cualquiera de las dos opciones sirve. Ambas son gratis y no requieren backend.

### Opción A: GitHub Pages

1. Creá un repositorio nuevo en GitHub (puede ser público o privado).
2. Subí el contenido de esta carpeta (al menos `index.html`; podés subir todo
   `jabugo-posventa/` tal cual).
3. En el repo: **Settings → Pages**.
4. En "Build and deployment" elegí **Deploy from a branch**, rama `main`,
   carpeta `/ (root)` — o `/jabugo-posventa` si subiste la carpeta completa
   dentro de un repo más grande.
5. Guardá. GitHub te va a dar una URL del estilo:
   `https://tu-usuario.github.io/tu-repo/` (o `.../jabugo-posventa/` según
   dónde quedó `index.html`).
6. Puede tardar 1–2 minutos en estar disponible la primera vez.

### Opción B: Netlify

1. Entrá a [app.netlify.com/drop](https://app.netlify.com/drop).
2. Arrastrá la carpeta `jabugo-posventa` (o al menos `index.html`) a la zona de
   drop.
3. Netlify te da al instante una URL del estilo
   `https://algun-nombre-random.netlify.app/`.
4. Opcional: desde el panel de Netlify podés cambiar el subdominio por uno más
   corto (Site settings → Change site name) o conectar un dominio propio.

---

## Paso 3 — Generar el QR final

1. Con el sitio ya publicado, copiá la URL final (la de GitHub Pages o Netlify).
2. Abrí `qr/generar-qr.html` en tu navegador (doble clic al archivo, no hace
   falta subirlo a ningún lado — funciona 100% offline).
3. Pegá la URL, ajustá el tamaño físico (6 cm anda bien para mostrador) y el
   texto debajo del QR.
4. Tocá **Generar QR**.
5. Descargá el **PNG en alta resolución** o usá **Imprimir / Guardar como PDF**
   para obtener un PDF listo para imprimir al tamaño elegido.

Antes de mandarlo a imprimir: escaneá el QR generado con un par de celulares
distintos (iOS y Android) para confirmar que abre la encuesta correctamente.

---

## Qué revisar antes de publicar

- [ ] Se borró la fila de ejemplo de la planilla.
- [ ] `GOOGLE_SHEETS_WEBAPP_URL` apunta al Apps Script ya desplegado (sin esto
      la encuesta no puede guardar nada).
- [ ] Se probó el flujo completo en un celular real (iOS y Android): elegir
      estrellas, tocar productos, escribir los comentarios, enviar, y
      confirmar que **no pasa nada en WhatsApp** y que la fila aparece en la
      planilla.
- [ ] La pestaña **Resumen** muestra los números esperados después de la
      prueba.
- [ ] El QR final apunta a la URL definitiva (no a `https://jabugo.example.com/`
      ni a `localhost`).

---

## Dónde queda guardada la información

- **Google Sheets** (planilla "Postventa – Almacén Gourmet Jabugo"): único
  destino de los datos. Privada, tuya, con las pestañas `Valoraciones`
  (respuesta por respuesta), `Detalle productos` (auxiliar, oculta) y
  `Resumen` (estadísticas en vivo).
- **`localStorage` del celular del cliente**: si por algún motivo falla el
  envío (sin señal, etc.), la respuesta queda guardada ahí y el sitio la
  reintenta solo la próxima vez que ese mismo celular abra la página. No es
  accesible para vos — es solo para no perder la respuesta si hay un
  problema de red puntual.
- **WhatsApp**: no interviene en nada. No se abre, no se manda ningún
  mensaje, no queda ningún rastro de la encuesta ahí.

El campo de teléfono que completa el cliente (columna **Teléfono /
WhatsApp**) queda guardado solo en la planilla, para que vos lo puedas
contactar con promos o novedades (fidelización) — es opcional, el cliente
puede dejarlo en blanco.
