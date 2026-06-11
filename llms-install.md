# Instalar el MCP server de Oktopus (para agentes como Cline)

Oktopus es un **MCP server remoto** (Streamable HTTP) — no se clona ni se corre local. Para "instalarlo" solo hay que **agregar la URL del server** a la configuración MCP del agente, con autenticación.

- **Endpoint MCP:** `https://www.oktopus.lat/api/mcp`
- **Transporte:** Streamable HTTP (remoto)
- **Autenticación:** API key (Bearer) **o** OAuth 2.1 + PKCE

## Opción A — API key (la más simple, recomendada para setup automático)

1. El usuario crea una cuenta en https://www.oktopus.lat y copia su API key desde **Configuración → API / Para agentes** (formato `okto_live_secret_...`).
2. Agregá este server remoto a la configuración MCP de Cline (`cline_mcp_settings.json`):

```json
{
  "mcpServers": {
    "oktopus": {
      "type": "streamableHttp",
      "url": "https://www.oktopus.lat/api/mcp",
      "headers": {
        "Authorization": "Bearer okto_live_secret_REEMPLAZAR"
      },
      "disabled": false
    }
  }
}
```

3. Reemplazá `okto_live_secret_REEMPLAZAR` por la API key del usuario. Guardá y reconectá.

> Pedile la API key al usuario antes de escribir el archivo. No inventes ni dejes la key de ejemplo.

## Opción B — OAuth 2.1 (consentimiento interactivo, sin pegar la key)

El server expone los metadatos OAuth estándar y Dynamic Client Registration:

- Protected resource metadata: `https://www.oktopus.lat/.well-known/oauth-protected-resource`
- Authorization server metadata: `https://www.oktopus.lat/.well-known/oauth-authorization-server`

Si el cliente MCP soporta OAuth para servers remotos, agregá el server solo con la `url` (sin `headers`) y el cliente disparará el flujo de autorización (PKCE S256) en el navegador.

```json
{
  "mcpServers": {
    "oktopus": {
      "type": "streamableHttp",
      "url": "https://www.oktopus.lat/api/mcp",
      "disabled": false
    }
  }
}
```

## Verificar la conexión

Una vez conectado, listá las tools: deberías ver ~47 herramientas `okto_*` (métricas, productos, landings, órdenes COD, Dropi, WhatsApp, checkout). Prueba rápida:

> "Mostrame las métricas de mi negocio y las órdenes contra entrega de hoy."

Si responde con datos de la cuenta, la conexión quedó lista.

## Notas

- Las tools de generación de imágenes/video con IA quedan **excluidas** cuando se conecta vía OAuth (por política de seguridad); con API key están disponibles según el plan.
- Documentación completa para agentes: https://www.oktopus.lat/para-agentes y https://www.oktopus.lat/docs
- El MCP **nunca** expone borrar cuenta/tienda, rotar secretos ni operaciones de tarjeta.
