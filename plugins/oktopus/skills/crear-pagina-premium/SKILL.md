---
name: crear-pagina-premium
description: Crear la página de venta de un producto con la plantilla nativa Premium Editorial de Oktopus, poniendo tú los textos y las imágenes (generadas con tus propios modelos). Oktopus arma la página real con checkout de pago contra entrega, packs, pixel y la carga más rápida; no se escribe HTML y no se gasta el cupo de IA de la cuenta. Usar cuando el usuario diga "créale una página premium a este producto", "arma las páginas de este catálogo", "haz las landings de los productos de este proveedor", "crea la página con tus propias imágenes", "que quede como las premium de Oktopus", o pase un catálogo o una lista de productos para montar. NO usar si quiere un diseño libre escrito por el agente (ver crear-mi-pagina) ni si quiere hostear la página en su propio dominio (ver conectar-mi-pagina).
---

# Crea páginas Premium con tus propios textos e imágenes

La idea: **tú pones las piezas, Oktopus arma la página.** Escribes los textos, generas las 8 imágenes con tu modelo de imagen, las subes y llamas a `okto_landing_create_premium`. La página sale de la plantilla Premium Editorial real: mismo diseño, mismo checkout contra entrega, mismo pixel y la misma carga rápida que una hecha dentro de Oktopus. No escribes HTML.

Lo que gasta de la cuenta: solo un cupo de landing por página. Ni imágenes ni IA de Oktopus.

## 0. Precondición

- El MCP de Oktopus conectado con una API key con `mcp:invoke` y `landings:write`. Verifica con `okto_dashboard_metrics`.
- Un generador de imágenes propio (el de tu cliente o el que el usuario te indique) que acepte fotos de referencia. Sin fotos de referencia del producto real no generes: pídelas.
- Una terminal con `curl` para subir las imágenes.

## 1. El molde, una vez por sesión

Llama a **`okto_premium_playbook`** y léelo completo. Te dice:

- `acceso`: si esta cuenta puede usar la plantilla Premium. Si `disponible` es `false`, detente y dile al usuario el motivo.
- `imagenes`: las 8 posiciones (5 de carrusel y 3 de sección), qué muestra cada una, su estilo, y la medida: **960×1280, vertical 3:4**.
- `textos.campos`: cada texto de la página con su límite de caracteres.
- `honestidad`: lo que ningún texto ni imagen puede decir. Lo que lo incumple se rechaza.
- `ejemplo_de_llamada`: la forma exacta de los argumentos.

No memorices los límites de una sesión anterior: el molde es la fuente y puede cambiar.

## 2. Por cada producto

1. **Producto y precio.** `okto_product_lookup` / `okto_products_list` → `product_id`. Si no existe: `okto_product_create`, o `okto_dropi_products_search` + `okto_dropi_import_product`. Tiene que estar activo y con precio.
   - El precio de venta y los packs salen de **`okto_price_recommend`** (Precio Inteligente): le pasas el costo del producto y la tienda, y devuelve `recommended_price` y `packs` (1, 2 y 3 unidades). Crea el producto con ese `price` y con `cost`. **Nunca inventes el precio** con un multiplicador; si no conoces el costo, pregúntaselo al usuario.
2. **Referencia.** `okto_product_images_get({ product_id })` → `uploaded_images` son las fotos reales. Descárgalas: son la referencia de forma, color, piezas y etiquetas. No se publican tal cual.
3. **Textos.** Escribe `copy` dentro de los límites. Español neutro, de tú. Describe el producto y su uso; no prometas resultados. Escribe además un titular y un apoyo para cada imagen. Todo texto le habla al comprador: nunca menciones "la ficha", "el proveedor", "el catálogo" ni lo que te falta saber. Si un dato no está (por ejemplo, la dosis), remite a la etiqueta del producto.
4. **Imágenes.** Genera las 8 siguiendo la dirección de arte de cada posición, con las fotos reales como referencia y el titular y el apoyo escritos dentro de la imagen, grandes y legibles en un celular. Guárdalas como `01.png` … `08.png`.
5. **Medida.** Si tu modelo no genera 3:4, lleva cada imagen a 960×1280 agregando márgenes del color del fondo. **Nunca recortes**: se corta el texto.
6. **Subida.** `okto_premium_image_upload` → `upload_url` (sirve una hora para todas). Una petición por imagen:
   ```bash
   curl -sS -X POST --data-binary @01.png -H "Content-Type: image/png" "$UPLOAD_URL"
   ```
   Cada respuesta trae `{ "ok": true, "url": "…" }`. Guarda la `url` de cada una. Si responde `wrong_ratio`, vuelve al paso 5.
7. **Página.** `okto_landing_create_premium({ product_id, copy, images, packs })` con esas `url` y los `packs` de `okto_price_recommend`. Si responde `ok:false`, lee `problems[]`, corrige justo eso y vuelve a llamar.
8. **Publicar.** La página queda en **borrador**. Publica con `okto_landing_publish({ landing_id })` solo si el usuario te pidió publicar; si no, entrégale `studio_url` para que la revise. Tu cliente puede pedir aprobación para este paso (publicar cambia algo visible en internet): es normal. Después espera a que `okto_landing_get` diga `live` y dé la URL pública; si la tienda está recibiendo otra actualización, puede tardar unos minutos.

Para corregir una página que ya creaste: la misma llamada con `replace_landing_id`, y vuelve a publicar.

## 3. Un catálogo completo

- Trabaja **un producto a la vez**, de punta a punta. No generes las imágenes de todos antes de subir ninguna.
- Lleva una tabla: producto, `landing_id`, estado (borrador o publicada), URL, y lo que quedó pendiente.
- Si un producto no tiene fotos reales o no tiene precio, sáltalo y anótalo; no inventes.
- Si llegas al límite de landings del plan (`plan_limit`), detente y avisa: no reintentes.
- Al terminar, entrega la tabla. No digas que una página está publicada si no viste la URL pública en la respuesta de `okto_landing_publish`.

## 4. Lo que se rechaza

- Imágenes que no son 3:4, o que no se subieron con el enlace de subida.
- Reseñas, estrellas, "más vendido", precios o porcentajes en textos o imágenes, "envío gratis", promesas de salud.
- Hablar de garantía sin declararla en `copy.guarantee` (y solo se declara si el dueño la ofrece de verdad).
- Antes y después de un resultado en el cuerpo. El antes/ahora muestra la situación, no el resultado.

## 5. Verificación

Después de publicar, abre la URL pública a 390 px de ancho y revisa: la portada carga primero, los titulares dentro de las imágenes se leen, el botón de compra abre el formulario. Los pedidos de una landing publicada son reales: para probar el formulario, haz un pedido y cancélalo con `okto_order_update_status`.
