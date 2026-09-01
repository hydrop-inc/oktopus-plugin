---
name: crear-mi-pagina
description: Crear una página de venta a medida para un producto del usuario (el agente escribe el HTML con el diseño que quiera) y subirla a Oktopus, que la hostea en <slug>.mipedido.lat con el checkout de pago contra entrega (COD) ya cableado — precio protegido, Dropi, WhatsApp, pixel + CAPI. Usar cuando el usuario diga "hacele una página a este producto", "hacele una página espectacular", "creá una landing y subila a Oktopus", "quiero mi propia página hosteada en Oktopus", "diseñá la página como vos quieras y publicala", "subí este HTML a Oktopus", "armá la landing de X con el checkout", o quiera reemplazar una landing subida por una versión nueva. NO usar si quiere hostear la página en su propio dominio (ver conectar-mi-pagina) ni si quiere la landing generada por la IA de Oktopus (okto_landing_generate_full).
---

# Creá la página y subila a Oktopus

La idea: **vos diseñás, Oktopus hostea y cobra.** Escribís un solo archivo HTML con el diseño que quieras, lo subís con `okto_landing_upload_html`, lo mirás en el preview, corregís, y lo publicás en `https://<slug>.mipedido.lat`. El checkout contra entrega lo cablea Oktopus: precio protegido en el servidor, orden a Dropi, confirmación por WhatsApp, Purchase por CAPI. No acuñás keys, no pegás snippets, no tocás el pixel.

No hay revisor humano ni IA en el medio: **lo que subís se publica tal cual pasa el sanitizador.** Por eso el contrato del paso 1 se cumple sí o sí.

## 0. Precondición: el MCP de Oktopus conectado

Necesitás las tools `okto_*` con una API key `secret` con scope `mcp:invoke` (+ `landings:write`, que viene por default).

- Si las tools no están: generar la key en **Configuración → Plataforma de agentes → API keys** (https://www.oktopus.lat/app/configuracion), `export OKTOPUS_API_KEY=okto_live_secret_...` y reiniciar el cliente.
- Verificá con `okto_dashboard_metrics`.
- Vas a necesitar también una skill de navegador (`agent-browser`) para mirar el preview a 390px. Si no la tenés, pedile al usuario que abra el `preview_url` en el celular y te cuente qué ve.

## 1. Playbook primero — leé el contrato

Llamá **`okto_page_playbook`** y leé TODO, en especial la clave **`subir_html_a_oktopus`**: es el contrato que tu HTML tiene que cumplir (reglas duras con su consecuencia, esqueleto mínimo, cómo va el checkout, qué JavaScript se permite, de dónde salen las imágenes, qué hace Oktopus con el pixel, el flujo exacto de tools y la verificación). También `estructura_recomendada`, `gatillos_mentales`, `confianza_cod` y `pais_y_moneda` (te dice si escribís con tuteo o voseo y en qué moneda se formatean los precios).

**No escribas una línea de HTML antes de leer esto.** Lo que viola una regla dura se quita (`removed[]`), se rechaza (error) o te avisa (`warnings[]`).

## 2. Producto

El checkout apunta a un producto de Oktopus.

- `okto_product_lookup({ query })` / `okto_products_list` → `product_id`, precio, precio tachado, moneda, tienda.
- ¿No existe? `okto_product_create`, o importalo de Dropi: `okto_dropi_products_search` + `okto_dropi_import_product` (trae fotos, precio y datos).
- Verificá: **activo**, con **precio** cargado, **sin variantes** (color/talla: el checkout embebido todavía no las elige; para eso usá `okto_landing_generate_full`), y de qué **tienda** es. `okto_store_get` → país, moneda y WhatsApp (si no tiene WhatsApp, decile que lo cargue: la confirmación por WhatsApp es parte de la confianza COD).
- `okto_dropi_coverage_get({ store_id })` → qué departamentos tienen contra entrega. Usalo en la página ("envío contra entrega a todo Colombia" o "a estas zonas").

Si el producto no es tuyo, está inactivo o tiene variantes, la tool te lo rechaza (`product_not_found` / `product_inactive` / `product_has_variants`): arreglalo antes, no lo fuerces.

## 3. Imágenes

- `okto_product_images_get({ product_id, purpose: "web" })` → `images[{ url, role }]` (hero, lifestyle, beforeafter, social, pack, cod, …) y `uploaded_images` (las fotos originales de Dropi). Usá esas URLs tal cual: son públicas y ya están hosteadas.
- Si el banco está vacío: `okto_product_images_generate({ product_id, set: "basic" })` genera las 6 básicas (tarda unos minutos; `set: "triggers"` son 10 con gatillos mentales). **Por el conector OAuth esta tool no aparece** (scope `ai:generate`): si no la tenés, usá `uploaded_images`, o pedile al usuario que las genere desde el dashboard (Producto → Imágenes IA) y volvé a llamar `okto_product_images_get`.
- Nunca base64, nunca fotos hotlinkeadas de Amazon/AliExpress/otra tienda. Si el usuario tiene fotos propias, tienen que estar en una URL https (su Drive público no sirve; un bucket o su propio sitio sí).

## 4. Escribí el HTML

**Brief de diseño (una pregunta, no diez):** si el usuario dijo "como vos quieras", decidí vos: elegí un estilo (premium minimal, colorido/juvenil, clínico/confiable, etc.) según el producto y decilo en una línea antes de arrancar. Si no lo dijo, preguntá UNA vez: estilo/colores de marca y si hay packs (2x/3x) y precios.

Después escribí la página, siguiendo:

- **Estructura** de `estructura_recomendada`: hero → problema → beneficios → cómo funciona → prueba social → antes/después → packs → garantía → FAQ → urgencia + checkout. Mínimo 3 gatillos mentales.
- **Reglas duras** de `subir_html_a_oktopus.reglas_duras`. Las que más se olvidan: un solo archivo con `<!doctype html>`, `<html lang="es">` y `<meta name="viewport">`; CSS en `<style>`; fuentes solo Google Fonts por `<link>`; imágenes por URL https; **`<div id="oktopus-checkout"></div>` en la sección de compra** y todos los CTAs con `href="#oktopus-checkout"`; **precios solo con `data-price="NUMERO"`** (el número crudo del producto o del pack, sin símbolo); **nada de pixel ni trackers**; `<script>` inline y `onclick` sí, `<script src>` solo de la allowlist; sin `fetch`; sin `<iframe>` (YouTube no: usá `<video>` mp4 o imagen); sin redirecciones; sin inputs de contraseña o tarjeta; máximo 512 KB; mobile-first a 390px.
- **Copy** en el español del país (`pais_y_moneda.registro`), 2ª persona, frases cortas, sin claims prohibidos (curas, garantías que la tienda no da, marcas ajenas, registros sanitarios inventados, contadores falsos).
- **Packs**: si hay, las tarjetas muestran el precio TOTAL del pack con `data-price`, marcás uno como "Más elegido", y decís "elegí tu pack en el formulario". El formulario muestra los packs como opciones; el servidor cobra el elegido.
- **Animaciones**: CSS o JS inline liviano. Nada de GSAP/framer-motion.

**Guardalo en disco** como `landing-<producto>.html` antes de subir (el usuario lo quiere tener; y vos lo vas a editar varias veces). Abrilo local a 390px si podés y revisá overflow antes del primer upload.

## 5. Subila

```
okto_landing_upload_html({
  product_id: "uuid-del-producto",
  html: "<contenido del archivo>",
  name: "Faja Reductora — página premium",   // opcional
  packs: [                                     // opcional; price = TOTAL del pack
    { name: "1 unidad", quantity: 1, price: 89900, compare_price: 129900 },
    { name: "Pack 2", quantity: 2, price: 159900, compare_price: 259800, popular: true },
    { name: "Pack 3", quantity: 3, price: 219900, compare_price: 389700 }
  ]
})
// → { ok, mode, landing_id, slug, status: "draft", preview_url, removed[], warnings[], packs, pixel, notes[] }
```

Lo primero que leés: **`removed[]`** (lo que el sanitizador quitó, con el motivo: scripts de otros hosts, iframes, trackers, widget pegado a mano) y **`warnings[]`** (precios que no coinciden, falta viewport, base64, `fetch` detectado, sin mount). Si hay algo ahí que no esperabas, corregí el archivo y **re-subí con `replace_landing_id: landing_id`**: se pisa el HTML, se conservan `landing_id` y `slug`. No crees una landing nueva por cada corrección.

Si la tool rechaza (`html_too_large`, `html_not_document`, `html_invalid`, `packs_invalid`, `plan_limit`…), el `note` te dice qué arreglar. `packs_invalid` trae el piso exacto.

## 6. Mirala a 390px

Con `agent-browser`: abrí `preview_url`, viewport 390×844, screenshot de página completa. Chequeá:

1. Hero: promesa, imagen, precio formateado (si ves "89900" pelado, falta `data-price`) y CTA visibles sin scrollear.
2. Sin scroll horizontal: `document.documentElement.scrollWidth === document.documentElement.clientWidth`. Probalo también a 320×658 (el público usa gama de entrada).
3. El **formulario del checkout está renderizado dentro de tu sección de compra** (nombre, teléfono, departamento, ciudad, dirección, cantidad o packs, botón). Si aparece una sección genérica al final, te faltó el `<div id="oktopus-checkout">`.
4. Botones de 44px o más, texto legible, imágenes cargadas (ninguna rota).
5. Consola: sin errores, salvo bloqueos de CSP de cosas que sabés que no deben cargar.

En el preview el formulario **no crea pedidos** (la landing es draft) — eso se prueba en el paso 8. Si el `preview_url` venció, `okto_landing_get({ id })` te da uno nuevo.

Corregí en el archivo, re-subí con `replace_landing_id`, volvé a mirar. Iterá hasta que quede impecable; mostrale el screenshot al usuario.

## 7. Publicá

Antes de publicar, dos cosas:

- **Pixel**: `okto_pixel_get({ store_id })`. Si no hay, `okto_pixel_set({ store_id, pixel_id })` AHORA — el pixel se hornea en el deploy; si lo cargás después, hay que republicar.
- **Confirmá con el usuario**: publicar deja la página accesible en internet con su producto y sus precios. Mostrale el preview y preguntá "¿la publico?".

Después:

```
okto_landing_publish({ landing_id })
// → { ok, request_id, landing_id, previous_status, message }
```

Esperá (el deploy pasa por la cola de Vercel; puede tardar varios minutos). Consultá `okto_landing_get({ id: landing_id })` hasta que `status` sea `"live"` y `public_url` tenga valor (`https://<slug>.mipedido.lat`). **No vuelvas a llamar publish** mientras esperás. Abrí `public_url` en el navegador y repetí el checklist del paso 6 sobre la página real.

## 8. Verificá que vende

**No hay modo test para pedidos desde una landing:** toda orden desde la página pública es real. La prueba se hace con una orden real que después se cancela.

1. Pedile al usuario **sus propios datos** (nombre, teléfono, ciudad, dirección) o que haga él el pedido desde `public_url`. **Nunca inventes datos de terceros** ni uses un teléfono que no sea del usuario: la tienda puede mandar WhatsApp y Dropi puede generar una guía.
2. Hacé el pedido desde la página. Tiene que aparecer "¡Pedido confirmado! #NNNN".
3. `okto_orders_recent({ limit: 5 })` → la orden está, con el total correcto (precio del producto o del pack elegido).
4. Cancelala: `okto_order_update_status({ order_id, status: "cancelled" })`. Si la tienda empuja directo a Dropi (sin confirmación por WhatsApp), la orden llegó a Dropi: decile al usuario que la cancele también ahí.

Alternativa que prueba el riel pero **no la página**: `okto_order_create_cod({ product_id, ..., test_mode: true })` crea una orden sandbox sin tocar Dropi ni WhatsApp. Sirve para validar precio/packs del servidor; no reemplaza el pedido desde la página.

## 9. Pixel

Con la página live, verificá la medición:

- `okto_pixel_get({ store_id })` → `configured: true`. Si no, `okto_pixel_set` y **republicá**.
- Meta Pixel Helper sobre `public_url`: al cargar, PageView + ViewContent (al pasar el 50 % de la página); al tocar el formulario, InitiateCheckout; al confirmar el pedido de prueba, Purchase con `eventID` = id de la orden.
- En Events Manager la Purchase aparece una sola vez (navegador + servidor deduplicados). La CAPI la dispara Oktopus; vos no hacés nada.

Si el playbook dijo que la CAPI no está lista (`pixel_y_capi.capi_purchase_serverside.estado_en_tu_cuenta`), guiá al usuario a conectar Meta en Oktopus antes de pautar.

## Reglas de oro

1. **Contrato primero**: leé `subir_html_a_oktopus` antes de escribir HTML. Lo que lo viola se quita o se rechaza sin preguntar.
2. **Un archivo, guardado en disco, iterado con `replace_landing_id`**: nunca una landing nueva por corrección.
3. **`removed[]` y `warnings[]` se leen siempre**; si algo se quitó y no lo esperabas, corregí antes de seguir.
4. **Mirala a 390px antes de publicar** y mostrale el screenshot al usuario. Mobile es el 97% del tráfico.
5. **Confirmá antes de publicar** y antes del pedido de prueba; el pedido es real, con datos del usuario, y se cancela.
6. **El pixel va antes de publicar** (se hornea en el deploy). Nunca lo pegues en el HTML.
7. **Multi-tenant aislado**: la key opera solo la cuenta del usuario; el producto y la landing tienen que ser suyos.
