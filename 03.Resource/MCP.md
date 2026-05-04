# MCP (Model Context Protocol)

## ¿Qué es MCP?

Protocolo abierto desarrollado por **Anthropic** que permite conectar asistentes de IA con herramientas y fuentes de datos externas de manera estandarizada.

## Problema que resuelve

- Cada tool/plugin tiene su propia API → integración manual para cada caso
- MCP unifica la comunicación: un cliente, múltiples servidores
- El modelo puede usar tools de forma dinámica sin hardcodear cada integración

## Arquitectura

```
┌─────────────┐     ┌──────────────┐     ┌─────────────────┐
│   Cliente   │────▶│   MCP Host   │────▶│  MCP Servers    │
│  (Cursor,  │     │ (-tu app de  │     │ (herramientas   │
│  Claude,    │     │  IA)         │     │  externas)      │
│  etc.)      │     │              │     │                 │
└─────────────┘     └──────────────┘     └─────────────────┘
```

## Componentes

### 1. Client
- Tu aplicación de IA (Cursor, Claude Desktop, etc.)
- Inicia conexiones con servidores MCP

### 2. Host
- El entorno donde corre el cliente
- Gestiona el ciclo de vida de las conexiones

### 3. Server
- Implementa el protocolo MCP
- Expone capabilities: tools, resources, prompts
- Ejemplos: filesystem, GitHub, databases, etc.

## Capabilities

### Tools
Funciones que el modelo puede invocar:
- `tools/list` → qué tools disponibles
- `tools/call` → ejecutar una tool

### Resources
Datos que el servidor puede compartir:
- `resources/list` → qué recursos hay
- `resources/read` → obtener contenido

### Prompts
Plantillas predefinidas para el modelo:
- `prompts/list` → prompts disponibles
- `prompts/get` → obtener un prompt específico

## Ejemplo de uso

```javascript
// Cliente MCP se conecta a un servidor
const client = new MCPClient({
  command: 'npx',
  args: ['-y', '@modelcontextprotocol/server-filesystem', '/ruta/proyecto']
});

// El modelo puede ahora:
// - Leer archivos del sistema
// - Escribir archivos
// - Listar directorios
// Sin necesidad de integrate cada tool manualmente
```

## Servers Populares

- `@modelcontextprotocol/server-filesystem` - Acceso al filesystem
- `@modelcontextprotocol/server-github` - GitHub API
- `@modelcontextprotocol/server-brave-search` - Búsqueda web
- `@modelcontextprotocol/server-postgres` - Base de datos
- `sequin/mcp-server` - Integración con SQL databases

## En resumen

MCP = **API universal para herramientas de IA**
- Mismo protocolo para cualquier tool
- El modelo descubre capabilities dinámicamente
- Separación clara entre cliente y servidor