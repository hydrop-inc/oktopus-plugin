---
name: conectar-mi-pagina
description: Conectar la página del usuario (hecha donde sea — Claude Code, su Vercel, Framer, su propio React) al riel de pago contra entrega (COD) de Oktopus, o exportar una landing de Oktopus a código editable. Usar cuando el usuario quiera vender contra entrega desde SU propia página, obtener el snippet/checkout embebible para su sitio, "ejectar"/exportar su landing de Oktopus como código (HTML o React) para hostearla en su propio Vercel, o hacer que su página venda más. Cubre el playbook de página que convierte, acuñar la publishable key, cablear el checkout y verificar con una orden de prueba.
---

# Conectá tu propia página al riel COD de Oktopus

La idea: **Oktopus es el riel, no el host.** El usuario arma su página **donde quiera** y le enchufa el checkout contra entrega. Las órdenes entran igual que una landing nativa: anti-fraude de precio + push a **Dropi** + confirmación por **WhatsApp**. Él es dueño de su página; el riel lo pone Oktopus.

Hay **3 niveles de libertad**:
1. **Widget** — pega un snippet (`Oktopus.checkout({...})`), cero backend.
2. **Headless** — su propio formulario + `POST /api/orders/create` con su publishable key.
3. **Eject** — se lleva una landing de Oktopus como **código** (HTML y/o React) y la deploya en su Vercel.

## 0. Precondición: el MCP de Oktopus conectado

Necesitás las tools `okto_*` disponibles, con una API key `secret` con scope `mcp:invoke` (+ `stores:write` y `landings:write`, que vienen por default en las keys secret).

- Si las tools no están, guiá: generar la key en **Configuración → Plataforma de agentes → API keys** (https://www.oktopus.lat/app/configuracion), `export OKTOPUS_API_KEY=okto_live_secret_...` (o `okto_test_secret_...` para sandbox) y reiniciar el cliente.
- Verificá con `okto_dashboard_metrics`.

## 1. SIEMPRE empezá por el playbook

Antes de armar o ejectar NADA, llamá **`okto_page_playbook`**. Te devuelve, con los datos reales de la cuenta del usuario:
- la **estructura** que convierte (hero→problema→beneficios→cómo funciona→prueba social→comparación→packs con anclaje→garantía→FAQ→urgencia→checkout),
- los **gatillos mentales** a usar,
- el estado de **pixel + CAPI** y el snippet del Meta Pixel con su pixel id,
- confianza COD, mobile/velocidad, errores comunes, checklist.

**Construí la página siguiendo ese playbook.** No armes "cualquier" página.

## 2. Encontrá (o creá) el producto

El checkout apunta a un producto de Oktopus (ligado a Dropi).
- `okto_product_lookup` (query) / `okto_products_list` — encontrar el producto y su `product_id`.
- ¿No existe? `okto_product_create` (o importá del catálogo Dropi con `okto_dropi_products_search` + `okto_dropi_link`).
- Confirmá el país con `okto_store_get` — el COD y Dropi dependen del país.

## 3. Armá la página — elegí el camino

### Camino A — el usuario quiere SU diseño desde cero
Construí la página (HTML/React/lo que use) **siguiendo el playbook del paso 1**. Para el checkout, elegí:
- **Widget (rápido):** dejá un `<div id="oktopus-checkout"></div>` y el snippet del paso 4.
- **Headless (control total):** su propio form que hace `POST https://www.oktopus.lat/api/orders/create` con `Authorization: Bearer <publishable_key>` y body `{ product_id, nombre, telefono, departamento, ciudad, direccion, quantity, total_price }`. La tienda y el país salen de la llave; las ciudades válidas, de `GET /api/geo/cities?country=XX`.

### Camino B — el usuario ya tiene una landing de Oktopus y la quiere a su medida
Usá **`okto_landing_eject`** (`{ landing_id, domains, format }`, format = `html` | `react` | `both`). Devuelve un map de archivos `{ files: [{path, content}] }` → **escribí cada archivo a disco** (una escritura por archivo). El checkout ya viene cableado y la publishable key ya acuñada. El proyecto trae el README con cómo deployar + editar.

> Si el usuario aún no tiene landing y quiere una base: `okto_landing_generate_full` genera una con IA, después la ejectás.

## 4. Acuñá la publishable key + cableá el checkout

Si NO usaste eject (que ya acuña la key), acuñala vos:

```
okto_checkout_key_create({
  domains: ["mitienda.com"],   // ⚠️ la key SOLO funciona desde estos dominios
  product_id: "uuid-del-producto",
  environment: "test"          // empezá en test; pasá a "live" al final
})
// → { publishable_key, snippet, ... }
```

Pegá el `snippet` que devuelve (o usá la `publishable_key` en tu form headless). **El dominio importa**: si la página va a vivir en otro dominio, acuñá otra key o agregá el dominio.

La publishable key es **pública** (va en el JS del cliente) y de bajo riesgo: solo crea órdenes y solo desde sus dominios.

## 5. Pixel + CAPI (medición)

Del playbook: agregá el **Meta Pixel base** en la página (con el pixel id del usuario) y disparás `PageView`/`ViewContent`/`InitiateCheckout`. **La conversión `Purchase` la dispara Oktopus del lado del servidor (CAPI)** cuando se crea la orden — también desde la página externa — así que NO hace falta dispararla en el navegador. Si el playbook dice que la CAPI no está lista, guiá al usuario a conectar su pixel/Meta en Oktopus antes de pautar.

## 6. Verificá con una orden de PRUEBA (no te saltes esto)

Antes de poner la página en producción:
1. Acuñá / usá una key **`test`** (`environment: "test"`).
2. Hacé un pedido de prueba desde la página (o `okto_order_create_cod` con `test_mode: true`).
3. Confirmá que la orden aparece con `okto_orders_recent` — **sin** tocar Dropi ni disparar WhatsApp (eso es lo que hace el modo test).
4. Recién ahí, acuñá la key **`live`** y reemplazá la del snippet.

## Reglas de oro

1. **Playbook primero**, siempre — que la página tenga lo que convierte.
2. **Test antes que live**: la key `test` crea órdenes sin tocar Dropi/WhatsApp.
3. **El dominio manda**: la publishable key es domain-locked; si cambia el dominio, cambia la key.
4. **Confirmá antes de pasar a live** y antes de cualquier escritura de alto impacto.
5. **Multi-tenant aislado**: la key opera solo la cuenta del usuario; nunca asumas datos de otra cuenta.
6. La página es del usuario; **Oktopus es el riel** (órdenes + Dropi + WhatsApp).
