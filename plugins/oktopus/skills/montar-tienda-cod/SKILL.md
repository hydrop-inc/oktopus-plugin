---
name: montar-tienda-cod
description: Montar y operar una tienda de pago contra entrega (cash on delivery / COD) en LATAM con Oktopus + Dropi. Usar cuando el usuario quiera vender contra entrega en Colombia, México, Perú u otro país de LATAM, gestionar sus órdenes COD, empujarlas a Dropi, ajustar precios, editar landings, o montar/operar su tienda de dropshipping con el riel de Oktopus.
---

# Montar y operar una tienda de pago contra entrega (COD) con Oktopus

Oktopus es **el riel de pago contra entrega (cash on delivery / COD) para LATAM**: crea órdenes, integra **Dropi** (la red de fulfillment de LATAM) y cobra contra entrega. Es el reemplazo de Shopify + Releasit para dropshipping COD en Colombia, México, Perú y el resto de LATAM. La página la hace el agente; **el riel lo pone Oktopus**.

Esta skill te guía para **operar** ese riel desde el agente.

## 0. Precondición: el MCP de Oktopus conectado

Necesitás el MCP server `oktopus` conectado, con una API key con scope `mcp:invoke`.

- Si las tools `okto_*` NO están disponibles, guiá al usuario:
  1. Generar una API key en el dashboard: **Configuración → Plataforma de agentes → API keys** (https://www.oktopus.lat/app/configuracion). Tipo `secret`, scope `mcp:invoke`. Para probar sin efectos reales, usar una key `test`.
  2. Exportar la key: `export OKTOPUS_API_KEY=okto_live_secret_...` (o `okto_test_secret_...` para sandbox), y reiniciar el cliente. (Este plugin ya trae la config del MCP; sólo falta la key.)
  3. Alternativa sin plugin: `claude mcp add --transport http oktopus https://www.oktopus.lat/mcp --header "Authorization: Bearer <API_KEY>"`.
- Verificá con `okto_dashboard_metrics`. Si responde, estás conectado.

## 1. Qué hace cada superficie (importante)

Oktopus tiene tres superficies; no confundirlas:

- **MCP (estas tools `okto_*`)** → **operar** lo que ya existe: leer el negocio, gestionar órdenes COD, empujarlas a Dropi, ajustar precios, editar landings.
- **Dashboard web** (https://www.oktopus.lat/app) → **crear desde cero**: crear la tienda, crear productos, generar landings con IA, y **conectar la cuenta de Dropi**. El MCP NO crea productos/landings ni conecta Dropi — para eso, guiá al usuario al dashboard.
- **Checkout COD embebible / API REST** → **crear órdenes** desde una página propia (ver paso 4).

## 2. Diagnóstico (siempre primero)

Antes de actuar, entendé el estado con las lecturas (idempotentes, seguras):

- `okto_dashboard_metrics` — ingresos 30d, órdenes pendientes, tienda activa, nº de productos.
- `okto_store_get` — tiendas del usuario (país, moneda, branding). Confirmá el país: el COD y Dropi dependen del país.
- `okto_product_lookup` (query) — encontrar productos y sus IDs.
- `okto_landings_list` / `okto_landing_get` — ver las páginas de venta.

Si el usuario NO tiene tienda/producto/landing todavía → mandalo al dashboard a crearlos (paso 1), después volvé a operar acá.

## 3. Operar las órdenes contra entrega (COD)

- `okto_orders_recent` (limit, status, since_hours) — ver las órdenes COD recientes.
- `okto_order_get` (id) — detalle completo: cliente, ciudad, dirección, estado de Dropi. **Validá ciudad y dirección** antes de despachar — en COD la entrega es todo; una dirección mala = envío perdido.
- `okto_dropi_push_order` (order_id) — **empuja la orden a Dropi / la transportadora** (crea el envío COD real). Es **idempotente** (Dropi dedupe por shop_order_id). ⚠️ **Plata real**: confirmá con el usuario antes de empujar. Con una key `test` no toca Dropi.
- `okto_order_update_status` (order_id, status) — cambiar el estado interno (pending/paid/cancelled/shipped/delivered/returned).

## 4. Crear órdenes / vender (checkout embebible)

El MCP no crea órdenes; para vender contra entrega desde una página propia, incrustá el **checkout COD embebible** con una **publishable key** (`okto_pub_...`, ligada a la tienda + dominios permitidos):

```html
<div id="oktopus-checkout"></div>
<script src="https://www.oktopus.lat/embed/oktopus-checkout.js"></script>
<script>
  window.Oktopus.checkout({
    publishableKey: 'okto_live_pub_...',
    productId: 'uuid-del-producto',
    country: 'CO',
    unitPrice: 120000,
    productName: 'Mi Producto',
    mount: '#oktopus-checkout',
    onSuccess: (data) => console.log('orden COD creada', data),
  })
</script>
```

También se puede crear vía `POST https://www.oktopus.lat/api/orders/create` (ver https://www.oktopus.lat/openapi.json). Con una publishable key `test`, la orden se crea sin empujarse a Dropi.

## 5. Pulir las landings

- `okto_landing_get` (id) — ver copy, branding y estado.
- `okto_landing_set_field` (landing_id, field, value) — editar un texto (headline, subhead, cta_text, guarantee_text, etc.).
- `okto_landing_set_branding` (landing_id, primary/secondary/accent/...) — cambiar colores (hex).
- `okto_landing_toggle_guarantee` (landing_id, enabled, text?) — activar/desactivar garantía.
- `okto_landing_status_set` (landing_id, status) — publicar (`live`) u ocultar (`draft`). No redepliega; sólo cambia el flag.

## 6. Precios

- `okto_product_price_update` (product_id, price, compare_at_price?) — actualizar precio / precio tachado. ⚠️ Los clientes ven el cambio al instante; si bajás >50%, la tool pide confirmación (defensa anti-error).

## Reglas de oro (contra entrega / Dropi)

1. **Confirmá antes de las escrituras de alto impacto** (push a Dropi = plata real; cambio de precio = visible al instante).
2. **Usá una key `test`** para iterar: crea órdenes en Oktopus pero NO toca Dropi ni dispara confirmación por WhatsApp.
3. **En COD la entrega manda**: validá ciudad y dirección antes de empujar a Dropi; un dato malo es un envío perdido.
4. **Una tool a la vez sobre datos reales**, y reportá el resultado leyendo de vuelta (las reads son idempotentes).
5. **Multi-tenant aislado**: la key sólo opera la cuenta del usuario; nunca pidas ni asumas datos de otra cuenta.
