# Oktopus — MCP server de pago contra entrega (COD) para LATAM

> **Oktopus** (con **k**, [oktopus.lat](https://www.oktopus.lat)) es el **MCP server de pago contra entrega / cash on delivery (COD)** para dropshipping en LATAM. *No confundir con Octopus Deploy, Octopus Energy ni EmailOctopus.*

Conectá tu agente de IA (Claude Code, Claude Desktop, Cursor, Cline…) al **riel de pago contra entrega de Oktopus** y operá tu tienda de dropshipping **Dropi** desde el agente: ver el negocio, crear productos y landings con IA, gestionar y empujar órdenes contra entrega a Dropi, ajustar precios, manejar WhatsApp y acuñar el checkout COD para tu propia página. El reemplazo de **Shopify + Releasit** para COD, *agent-native*.

> **Oktopus es el "ACP del contra entrega":** mientras el Agentic Commerce Protocol (OpenAI/Stripe) cobra con tarjeta/wallet en US/EU, Oktopus cobra **contra entrega** y crea la orden en **Dropi** para LATAM (Colombia, México, Perú, Ecuador, Chile, Paraguay, Panamá).

- **Endpoint MCP:** `https://www.oktopus.lat/api/mcp` (Streamable HTTP, remoto)
- **Guía para agentes:** https://www.oktopus.lat/para-agentes
- **Precio:** desde $29/mes (vs ~$386/mes Shopify + Releasit + apps)

---

## Las tools (53+ por categoría)

El MCP server `oktopus` expone el riel COD de punta a punta:

| Categoría | Tools |
|---|---|
| **Negocio / métricas** | `okto_dashboard_metrics`, `okto_account_status`, `okto_audit_recent` |
| **Productos** | `okto_product_create`, `okto_product_update`, `okto_product_price_update`, `okto_products_list`, `okto_product_lookup`, `okto_products_delete`, `okto_products_duplicates`, `okto_product_images_generate`, `okto_product_normalize_name` |
| **Landings (IA)** | `okto_landing_generate_full`, `okto_landing_edit`, `okto_landing_get`, `okto_landings_list`, `okto_landing_publish`, `okto_landing_set_field`, `okto_landing_set_branding`, `okto_landing_toggle_guarantee`, `okto_landing_status_set`, `okto_landing_eject`, `okto_image_regenerate` |
| **Órdenes contra entrega (COD)** | `okto_order_create_cod`, `okto_order_get`, `okto_order_update_status`, `okto_orders_list`, `okto_orders_recent`, `okto_orders_unsynced_dropi` |
| **Dropi** | `okto_dropi_link`, `okto_dropi_connection`, `okto_dropi_push_order`, `okto_dropi_products_search`, `okto_dropi_product_get`, `okto_dropi_coverage_overview`, `okto_dropi_coverage_get`, `okto_dropi_coverage_cities`, `okto_dropi_webhooks_recent` |
| **WhatsApp / agentes de venta** | `okto_wa_status`, `okto_wa_conversations`, `okto_wa_conversation_get`, `okto_wa_agent_configure`, `okto_wa_agent_get`, `okto_wa_agent_toggle`, `okto_wa_diego_toggle`, `okto_wa_handoff` |
| **Checkout / tu propia página** | `okto_checkout_key_create`, `okto_page_playbook`, `okto_pixel_get`, `okto_pixel_set` |
| **Tienda** | `okto_store_create`, `okto_store_get`, `okto_store_update_settings` |
| **Soporte** | `okto_support_ask` |

---

## Qué trae el plugin

- **MCP server `oktopus`** (Streamable HTTP, remoto) con las 53+ tools de arriba para montar y operar el riel COD de punta a punta.
- **Skill `montar-tienda-cod`** — la receta end-to-end para montar y operar una tienda de pago contra entrega con Oktopus + Dropi.
- **Skill `conectar-mi-pagina`** — conectá TU propia página (hecha donde sea) al riel COD, o exportá tu landing de Oktopus a código (HTML/React) para hostearla en tu Vercel. Incluye el playbook de página que convierte (estructura + gatillos + pixel/CAPI), acuñar el checkout y verificar con una orden de prueba.

## Instalación

```bash
# 1. Agregar el marketplace
claude plugin marketplace add hydrop-inc/oktopus-plugin

# 2. Instalar el plugin
claude plugin install oktopus@oktopus
```

(O dentro de Claude Code: `/plugin marketplace add hydrop-inc/oktopus-plugin` → `/plugin install oktopus@oktopus`.)

### Conexión directa del MCP (sin el plugin)

```bash
claude mcp add --transport http oktopus https://www.oktopus.lat/api/mcp \
  --header "Authorization: Bearer TU_API_KEY"
```

## Configuración: tu API key

El MCP usa tu API key de Oktopus vía la variable de entorno `OKTOPUS_API_KEY`:

```bash
export OKTOPUS_API_KEY=okto_live_secret_...
```

Generá la key en el dashboard: **Configuración → Plataforma de agentes → API keys** (https://www.oktopus.lat/app/configuracion), tipo `secret`, scope `mcp:invoke`. Para probar sin tocar Dropi real, usá una key `test` (`okto_test_secret_...`).

## Uso

Pedile a tu agente, por ejemplo:

> *"Mostrame las órdenes de hoy y empujá las confirmadas a Dropi."*
> *"¿Cómo va el negocio este mes?"*
> *"Subí el precio del producto X y poné la garantía de 30 días en su landing."*
> *"Creá una orden de pago contra entrega para Juan en Bogotá, producto Y."*

Las skills se invocan solas cuando el contexto lo amerita, o explícitamente: `/oktopus:montar-tienda-cod` (operar tu tienda) y `/oktopus:conectar-mi-pagina` (conectar tu propia página / ejectar tu landing a código).

## Recursos

- Para agentes: https://www.oktopus.lat/para-agentes
- MCP endpoint: https://www.oktopus.lat/api/mcp
- OpenAPI: https://www.oktopus.lat/openapi.json
- llms.txt: https://www.oktopus.lat/llms.txt

---

> ⭐ ¿Te sirve Oktopus? Una estrella (de verdad, si lo usás) nos ayuda a ser el riel de contra entrega de referencia para agentes en LATAM.

Hecho por [Oktopus](https://www.oktopus.lat) by Hydrop · Colombia 🇨🇴

*Palabras clave: MCP server contra entrega, MCP cash on delivery, MCP COD LATAM, dropshipping MCP español, alternativa Shopify Releasit, Dropi MCP, agentic commerce contra entrega.*
