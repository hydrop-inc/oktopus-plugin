---
name: crear-pagina-persuasiva
description: Crear una página de venta contra entrega de alta conversión con la plantilla Premium Persuasiva de Oktopus, poniendo tú los textos y las imágenes (con tus propios modelos). Es el método con el que salieron páginas que vendieron desde el primer día: promesa con el resultado, carrusel de 10 imágenes que funciona como embudo, sellos tipo tarjeta, fotos de uso debajo de los sellos, "¿Te pasa esto?", antes y después deslizable, preguntas frecuentes de objeciones reales y botón de WhatsApp. Usar cuando el usuario diga "créame una página que venda", "una página persuasiva", "una landing agresiva", "rehaz esta página para que convierta", "la página de este producto", o pase un producto para lanzar con anuncios. Para una página sencilla con la Premium Editorial ver crear-pagina-premium; para HTML libre, crear-mi-pagina.
---

# Crea una página Persuasiva que venda

**Tú pones los textos y las imágenes; Oktopus arma la página** con su checkout contra entrega, los combos, el pixel y la carga rápida. No escribes HTML. Gasta un cupo de landing por página; ni imágenes ni IA de la cuenta.

La idea de fondo: **la página continúa el anuncio**. Los mismos dolores, el mismo mensaje y el mismo estilo, pero más directa y más completa, porque ahí la persona ya decidió escuchar. Cada bloque tiene que tener una intención; si un bloque no acerca a la compra, sobra.

## 0. Precondición

- El MCP de Oktopus conectado, con una API key con `mcp:invoke` y `landings:write`. Verifica con `okto_dashboard_metrics`.
- Un generador de imágenes propio que acepte fotos de referencia.
- Una terminal con `curl` para subir las imágenes.

## 1. El molde, una vez por sesión

Llama a **`okto_premium_playbook`** y lee completa la sección `persuasiva`: los campos, sus límites de caracteres, lo que se rechaza (`honestidad`) y un ejemplo de llamada. **El molde manda sobre esta skill**: si algo cambió, sigue el molde.

## 2. Antes de escribir

1. **Producto y precio.** `okto_product_lookup` / `okto_products_list` → `product_id`. El precio y los combos salen de **`okto_price_recommend`** (costo + tienda). Nunca inventes el precio. Confirma con el usuario el precio final y los combos (por ejemplo, 1 unidad y 2 unidades con el precio tachado de dos sueltas).
2. **Fotos reales del producto.** `okto_product_images_get` → descárgalas. Son la referencia de forma, color, etiqueta y **tamaño**. No se publican tal cual.
3. **El mensaje.** Si el usuario ya tiene anuncios, usa sus mismos dolores y su misma frase. Si no, define con él: los 3-5 dolores reales (emocionales: miedo, pena, pareja, frustración, no entender), el mecanismo (por qué pasa) y el villano (lo que la gente hace mal o el remedio casero que no sirve).
4. **La competencia.** Abre 2 o 3 páginas de quienes venden algo parecido y anota qué beneficios y objeciones tocan. Si ellos hablan del mal olor o del ardor y tú no, te falta.

## 3. Los textos

Respeta los límites del molde. Español neutro, de tú.

- **Promesa** (`headline`): es el gancho, con el resultado concreto y los dolores por su nombre. Ejemplo de forma: "Elimina [causa] que provoca [dolor 1], [dolor 2] y [dolor 3]". Nada genérico como "cuidado de calidad".
- **Subtítulo**: quita la culpa ("no es tu culpa", "casi nunca es lo que imaginas"), explica en una frase por qué pasa y presenta el producto.
- **Beneficios**: cada uno es un dolor resuelto, y uno más de discreción o confianza ("Nadie se entera", "Pagas al recibir").
- **"¿Te pasa esto?"** (`teIdentificas`): situaciones concretas en segunda persona, las que dan pena decir ("Te pica más de noche", "Te da miedo que tu pareja lo note"), y un cierre que quita la culpa.
- **Preguntas frecuentes** (máx. 8): primero las objeciones vergonzosas y reales, después las prácticas (cómo se usa, cuánto dura, envío y pago, garantía).
- **Cierre**: un titular corto y emocional, y la fórmula "Si quieres [resultado], [producto] es tu mejor opción".
- **Palabras directas.** Si es un producto íntimo o de adultos, nombra las cosas como son (vagina, vulva, pene, hongo), con respeto y nunca vulgar. Esconder las palabras confunde, y una mente confundida no compra.
- Escribe "Envío sin costo", no "envío gratis". No inventes estudios, reseñas, plazos ni escasez. Las reseñas solo pueden ser reales e importadas con su enlace; si no hay, la página va sin estrellas. "Agotado" o "Quedan pocas unidades" solo si es verdad.

## 4. Las imágenes

Todas verticales 3:4. Si tu modelo no genera 3:4, lleva cada imagen a 960×1280 con márgenes del color del fondo; **nunca recortes**. Prompts en inglés y todo texto visible en español exacto, entre comillas; revisa la ñ y las tildes una por una. Sin logos ni marcas de agua.

**El carrusel es un embudo de 10 imágenes, en este orden:**

1. **Lo que hace el producto, de un vistazo**: 4 filas, cada una con una miniatura realista del dolor a la izquierda y el beneficio en letra grande a la derecha; el producto pequeño. Sin envío ni pago en esta imagen.
2. El problema principal: antes y después.
3. Un dolor con emoción: una persona reaccionando.
4. Otro dolor.
5. El mecanismo: por qué pasa.
6. Cómo actúa el producto: 3 pasos, cada uno con su ingrediente activo.
7. El villano: los remedios caseros que no funcionan, frente al producto.
8. La oferta: alguien recibiendo una caja sin marca en la puerta; paga al recibir.
9. Prueba de uso, sin caras.
10. La caja discreta, al abrirla.

**Fotos de uso** (`persuasiva.fotos`, 3 a 5): fotos de celular, luz natural, casa real, **sin caras**, mostrando a alguien usando o recibiendo el producto. Van justo debajo de los sellos. El producto idéntico a la referencia y **de su tamaño real**: un frasco de 30 ml mide unos 9 cm y cabe en la palma. Tapa cerrada, sin boquillas ni chorros inventados.

**Historia y comparador:** una sola historia (`historias`) con un momento real de uso, el de la vida de la persona (no "en tu rutina"). La segunda historia la reemplaza el **comparador** antes/después deslizable. La imagen de "después" se genera a partir de la de "antes": mismo encuadre, misma luz y misma piel, solo sin el problema. Para anatomía íntima femenina usa ilustración vectorial educativa, estilo folleto de salud, nunca fotos.

Para agregar algo a una imagen ya hecha, regénerala desde cero con el elemento en el prompt; editarla casi siempre la daña.

## 5. Subir y crear

1. `okto_premium_image_upload` → `upload_url` (sirve una hora). Una petición por imagen:
   ```bash
   curl -sS -X POST --data-binary @01.png -H "Content-Type: image/png" "$UPLOAD_URL"
   ```
   Guarda la `url` de cada respuesta. Si dice `wrong_ratio`, vuelve a ajustar a 3:4.
2. `okto_landing_create_premium` con `template: "premium-persuasiva"`, `product_id`, `copy` (con `guarantee` si el dueño la ofrece de verdad), `images.carousel` (las 10), `packs` y `persuasiva`:
   - `estiloSellos: "tarjeta"` con garantía, pago al recibir y caja discreta.
   - `fotosPosicion: "tras_sellos"`, `tipografia: "impacto"` y `whatsapp: {}` (el botón usa el WhatsApp de la tienda).
   - `teIdentificas`, `comparador`, una sola `historias`, `franja` de 4 datos, `ingredientes` (con estudio solo si es real y con un resumen honesto) y `tiempo`.

   Si responde `ok:false`, lee `problems[]`, corrige exactamente eso y vuelve a llamar. Para corregir una página existente: la misma llamada con `replace_landing_id`.
3. La página queda en **borrador**. Publica con `okto_landing_publish` solo si el usuario lo pidió; si no, entrégale `studio_url`. Espera a que `okto_landing_get` diga `live` y dé la URL pública.

## 6. El cierre por WhatsApp

Los pedidos contra entrega se confirman por WhatsApp. Si la tienda tiene el asistente de confirmación, dale la misma información de la página. Con `okto_wa_agent_configure` (instrucciones): que su objetivo sea que el cliente confirme; que responda las dudas con los beneficios y las objeciones de la página y termine cada mensaje pidiendo la confirmación; que ofrezca el combo de 2; y que no invente nada que no diga la página. Si el asistente no sabe qué hace el producto, ante una pregunta como "¿qué cura?" va a responder mal y se pierde la venta.

## 7. Verificación

Abre la URL pública a 390 px de ancho y revisa:
- La promesa se lee primero.
- Las 10 imágenes del carrusel cuentan la historia en orden.
- Los sellos tarjeta y las fotos de uso están debajo de los combos.
- El comparador se desliza con el dedo.
- El botón de compra abre el formulario y el de WhatsApp abre el chat de la tienda.

Los pedidos de una página publicada son reales: para probar, haz uno y cancélalo con `okto_order_update_status`.
