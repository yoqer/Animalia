# 📘 Manual del Orquestador de IA Multi-Proveedor

**Para Animalia Web Hosting - Integración de IA Flexible**

---

## 🎯 ¿Qué es el Orquestador de IA?

El **Orquestador de IA** es un sistema que permite seleccionar dinámicamente qué proveedor de IA usar para cada solicitud. Soporta:

- ✅ **OpenAI/GPT**: Modelos GPT-4, GPT-3.5 y compatibles con API de OpenAI.
- ✅ **Manus API**: Modelos de IA de Manus
- ✅ **LLM Local**: Modelos ejecutados localmente (Ollama, etc.)
- ✅ **Agentes Personalizados**: Clawbot, asistentes custom, etc.

**Ventajas**:
- Flexibilidad: Cambiar proveedor sin cambiar código
- Fallback automático: Si un proveedor falla, intenta otro
- MCP integrado: Protocolo Model Context Protocol
- A2A: Comunicación entre agentes
- API REST: Fácil de integrar

---

## 🚀 Configuración Rápida

### Paso 1: Configurar Proveedores

En tu archivo `.env`:

```bash
# OpenAI/GPT
OPENAI_API_KEY=sk-...

# Manus (ya configurado por defecto)
BUILT_IN_FORGE_API_KEY=tu_key
BUILT_IN_FORGE_API_URL=https://api.manus.im

# LLM Local (Ollama)
LOCAL_LLM_ENDPOINT=http://localhost:11434
```

### Paso 2: Usar en tu Aplicación

```typescript
import { aiOrchestrator } from './server/ai-orchestrator';

// Invocar GPT
const response = await aiOrchestrator.invoke({
  prompt: 'Analiza este patrón de comportamiento animal',
  config: {
    provider: 'openai',
    model: 'gpt-4',
    temperature: 0.7,
  },
});

console.log(response.content);
```

---

## 📚 Guía de Uso Completa

### 1. Invocar Proveedor Específico

```typescript
// Usar OpenAI
const response = await aiOrchestrator.invoke({
  prompt: 'Tu pregunta aquí',
  config: {
    provider: 'openai',
    model: 'gpt-4',
    temperature: 0.7,
    maxTokens: 2000,
  },
});
```

### 2. Invocar con Fallback Automático

Si el proveedor principal falla, intenta otros automáticamente:

```typescript
const response = await aiOrchestrator.invokeWithFallback(
  {
    prompt: 'Tu pregunta aquí',
    config: {
      provider: 'openai',
      model: 'gpt-4',
    },
  },
  ['manus', 'local'] // Fallback a Manus, luego Local
);
```

### 3. Usar Herramientas MCP

Las herramientas MCP permiten que la IA acceda a funcionalidades de Animalia:

```typescript
const response = await aiOrchestrator.invoke({
  prompt: 'Busca patrones de comportamiento de leones',
  config: {
    provider: 'openai',
  },
  tools: [
    {
      name: 'search_patterns',
      description: 'Buscar patrones de comportamiento',
      parameters: {
        type: 'object',
        properties: {
          query: { type: 'string' },
        },
      },
      handler: async (params) => {
        // Tu lógica aquí
      },
    },
  ],
});
```

---

## 🔌 Integración con API REST

### Endpoint: Invocar IA

```bash
POST /api/trpc/ai.invoke
Content-Type: application/json

{
  "prompt": "Analiza este comportamiento",
  "provider": "openai",
  "model": "gpt-4",
  "temperature": 0.7,
  "useMCPTools": true
}
```

**Respuesta**:
```json
{
  "success": true,
  "content": "Respuesta de la IA...",
  "provider": "openai",
  "model": "gpt-4",
  "tokensUsed": 150
}
```

### Endpoint: Invocar con Fallback

```bash
POST /api/trpc/ai.invokeWithFallback
Content-Type: application/json

{
  "prompt": "Tu pregunta",
  "primaryProvider": "openai",
  "fallbackProviders": ["manus", "local"]
}
```

### Endpoint: Listar Proveedores

```bash
GET /api/trpc/ai.listProviders
```

**Respuesta**:
```json
{
  "providers": [
    {
      "id": "openai",
      "name": "OpenAI GPT",
      "available": true,
      "models": ["gpt-4", "gpt-3.5-turbo"]
    },
    {
      "id": "manus",
      "name": "Manus API",
      "available": true,
      "models": ["manus-default"]
    },
    ...
  ]
}
```

### Endpoint: Probar Proveedor

```bash
POST /api/trpc/ai.testProvider
Content-Type: application/json

{
  "provider": "openai",
  "model": "gpt-4"
}
```

---

## 🤖 Registrar Agentes Personalizados

### Ejemplo: Registrar Clawbot

```typescript
import { aiOrchestrator } from './server/ai-orchestrator';

// Registrar agente
aiOrchestrator.registerAgent('clawbot-1', {
  id: 'clawbot-1',
  name: 'Clawbot',
  description: 'Agente orquestador local',
  config: {
    tools: ['search', 'analyze', 'execute'],
  },
  process: async (request) => {
    // Lógica del agente
    const result = await processWithClawbot(request.prompt);
    return {
      content: result,
      metadata: { agentId: 'clawbot-1' },
    };
  },
});

// Usar agente
const response = await aiOrchestrator.invoke({
  prompt: 'Analiza esto',
  config: {
    provider: 'agent',
    agentId: 'clawbot-1',
  },
});
```

### Registrar vía API

```bash
POST /api/trpc/ai.registerAgent
Content-Type: application/json

{
  "agentId": "mi-agente",
  "name": "Mi Agente",
  "description": "Descripción del agente",
  "config": {
    "tools": ["search", "analyze"],
    "model": "custom"
  }
}
```

---

## 🔄 Comunicación A2A (Agent-to-Agent)

Los agentes pueden comunicarse entre sí:

```typescript
// Agente 1 solicita ayuda a Agente 2
const response = await aiOrchestrator.invoke({
  prompt: 'Agente2, necesito que analices esto',
  config: {
    provider: 'agent',
    agentId: 'agente-2',
  },
  context: {
    requesterAgent: 'agente-1',
    requestType: 'analysis',
  },
});
```

---

## 🛠️ Herramientas MCP Disponibles

### 1. search_patterns

Buscar patrones de comportamiento animal.

```typescript
{
  name: 'search_patterns',
  parameters: {
    query: 'comportamiento de caza',
    limit: 10,
  },
}
```

### 2. get_knowledge

Obtener base de conocimiento.

```typescript
{
  name: 'get_knowledge',
  parameters: {
    category: 'comportamiento',
  },
}
```

### 3. request_retraining

Solicitar reentrenamiento del modelo.

```typescript
{
  name: 'request_retraining',
  parameters: {
    datasetId: 'dataset-123',
    epochs: 5,
  },
}
```

---

## 📊 Comparativa de Proveedores

| Aspecto | OpenAI | Manus | Local | Agente |
|--------|--------|-------|-------|--------|
| **Costo** | Pago | Incluido | Gratuito | Gratuito |
| **Latencia** | Media | Baja | Variable | Baja |
| **Calidad** | Excelente | Buena | Media | Variable |
| **Privacidad** | Remoto | Remoto | Local | Local |
| **Offline** | ❌ | ❌ | ✅ | ✅ |
| **Personalización** | Media | Media | Alta | Muy Alta |

---

## 🔐 Seguridad

### 1. Proteger Claves API

```bash
# .env (nunca compartir)
OPENAI_API_KEY=sk-... # Secreto
BUILT_IN_FORGE_API_KEY=... # Secreto
```

### 2. Validar Solicitudes

```typescript
// Solo usuarios autenticados
export const aiRouter = router({
  invoke: protectedProcedure
    .input(...)
    .mutation(async ({ ctx, input }) => {
      // ctx.user contiene usuario autenticado
      if (!ctx.user) throw new Error('No autenticado');
      // ...
    }),
});
```

### 3. Rate Limiting

```typescript
// Limitar solicitudes por usuario
const requestsPerMinute = 10;
// Implementar en middleware
```

---

## 🚨 Problemas Comunes

### "Proveedor no configurado"

**Causa**: Falta configurar la clave API

**Solución**:
```bash
# Agregar a .env
OPENAI_API_KEY=sk-...
```

### "Conexión rechazada a LLM Local"

**Causa**: Ollama no está ejecutándose

**Solución**:
```bash
# Instalar Ollama
# Descargar desde: https://ollama.ai

# Ejecutar
ollama serve

# En otra terminal
ollama pull llama2
```

### "Agente no encontrado"

**Causa**: Agente no registrado

**Solución**:
```typescript
// Registrar agente primero
aiOrchestrator.registerAgent('mi-agente', agentConfig);
```

---

## 📈 Optimización

### 1. Usar Fallback para Confiabilidad

```typescript
// Siempre tener fallback
await aiOrchestrator.invokeWithFallback(request, ['manus', 'local']);
```

### 2. Cachear Respuestas

```typescript
// Cachear respuestas frecuentes
const cache = new Map();
const key = `${prompt}-${provider}`;
if (cache.has(key)) return cache.get(key);
```

### 3. Usar LLM Local para Desarrollo

```typescript
// Desarrollo: usar local (gratuito)
// Producción: usar openai (mejor calidad)
const provider = process.env.NODE_ENV === 'production' ? 'openai' : 'local';
```

---

## 📞 Ejemplos Completos

### Ejemplo 1: Análisis de Comportamiento

```typescript
const response = await aiOrchestrator.invoke({
  prompt: `Analiza este comportamiento animal:
    Especie: León
    Comportamiento: Cazando en manada
    Contexto: Sabana africana`,
  config: {
    provider: 'openai',
    model: 'gpt-4',
    temperature: 0.5,
  },
  tools: [
    {
      name: 'search_patterns',
      // ...
    },
  ],
});
```

### Ejemplo 2: Reentrenamiento Automático

```typescript
const response = await aiOrchestrator.invoke({
  prompt: 'Necesito reentrenar el modelo con nuevos datos',
  config: {
    provider: 'agent',
    agentId: 'retraining-agent',
  },
  context: {
    datasetId: 'dataset-123',
    epochs: 5,
  },
});
```

### Ejemplo 3: Comunicación A2A

```typescript
// Agente de análisis solicita ayuda a agente de búsqueda
const response = await aiOrchestrator.invoke({
  prompt: 'Busca patrones similares a este comportamiento',
  config: {
    provider: 'agent',
    agentId: 'search-agent',
  },
  context: {
    requesterAgent: 'analysis-agent',
    pattern: patternData,
  },
});
```

---

## ✅ Checklist

- [ ] Configurar claves API en `.env`
- [ ] Probar cada proveedor con `testProvider`
- [ ] Implementar fallback automático
- [ ] Registrar agentes personalizados
- [ ] Configurar herramientas MCP
- [ ] Implementar caché de respuestas
- [ ] Agregar rate limiting
- [ ] Documentar proveedores personalizados

---

**¡Listo! El Orquestador de IA está configurado.** ✅

