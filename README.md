# Oktopus — plugin de Claude Code (riel COD para LATAM)

> ⭐ **¿Te sirve Oktopus?** Dejanos una estrella — nos ayuda a llegar a más dropshippers de LATAM y a ser el riel de contraentrega de referencia para agentes.

Conectá tu agente al **riel de pago contra entrega (cash on delivery / COD) de Oktopus** para LATAM. Operá tu tienda de dropshipping **Dropi** desde el agente: ver el negocio, gestionar órdenes contra entrega, empujarlas a Dropi, ajustar precios y editar landings. El reemplazo de Shopify + Releasit para COD.

> **Oktopus es el "ACP del contra entrega":** mientras el Agentic Commerce Protocol cobra con tarjeta/wallet en US/EU, Oktopus cobra **contra entrega** y crea la orden en **Dropi** para LATAM (Colombia, México, Perú, Ecuador, Chile, Paraguay, Argentina, Guatemala).

## Qué trae

- **MCP server `oktopus`** (Streamable HTTP, remoto) con 50+ tools para montar y operar el riel COD de punta a punta.
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

## Configuración: tu API key

El MCP usa tu API key de Oktopus vía la variable de entorno `OKTOPUS_API_KEY`:

```bash
export OKTOPUS_API_KEY=okto_live_secret_...
```

Generá la key en el dashboard: **Configuración → Plataforma de agentes → API keys** (https://www.oktopus.lat/app/configuracion), tipo `secret`, scope `mcp:invoke`. Para probar sin tocar Dropi real, usá una key `test` (`okto_test_secret_...`).

## Uso

Pedile a tu agente, por ejemplo:

> "Mostrame las órdenes de hoy y empujá las confirmadas a Dropi."
> "¿Cómo va el negocio este mes?"
> "Subí el precio del producto X y poné la garantía de 30 días en su landing."

Las skills se invocan solas cuando el contexto lo amerita, o explícitamente: `/oktopus:montar-tienda-cod` (operar tu tienda) y `/oktopus:conectar-mi-pagina` (conectar tu propia página / ejectar tu landing a código).

## Recursos

- Para agentes: https://www.oktopus.lat/para-agentes
- MCP endpoint: https://www.oktopus.lat/mcp
- OpenAPI: https://www.oktopus.lat/openapi.json
- llms.txt: https://www.oktopus.lat/llms.txt

---

Hecho por [Oktopus](https://www.oktopus.lat) by Hydrop · Colombia 🇨🇴
