# 📘 Manual del Panel de Administración

**Para Animalia Web Hosting - Gestión Avanzada de Proveedores e Integraciones**

---

## 🎯 ¿Qué es el Panel de Administración?

El **Panel de Administración** es una interfaz centralizada para gestionar:

1. **Proveedores de IA**: OpenAI, Gemini, Claude, Grok, Meta, Qwen, Deepseek, Perplexity, Copilot, Ollama, LMstudio
2. **Bases de Datos Externas**: Supabase, Database.com, Notion, Obsidian, NotebookLM
3. **Webhooks**: Automatización y eventos
4. **Caché**: Configuración de Redis
5. **Extracción de Datos**: Para entrenamiento de modelos

---

## 🚀 Acceso al Panel

### URL

```
https://tu-dominio.com/admin
```

### Requisitos

- ✅ Usuario autenticado
- ✅ Rol de administrador
- ✅ Acceso a internet

---

## 📋 Secciones del Panel

### 1️⃣ Proveedores de IA

#### Agregar Nuevo Proveedor

**Paso 1**: Ir a **Proveedores IA** → **Agregar Nuevo Proveedor**

**Paso 2**: Completar formulario

| Campo | Descripción | Ejemplo |
|-------|-------------|---------|
| **Nombre** | Nombre del proveedor | OpenAI, Gemini, Claude |
| **Clave API** | Token de autenticación | sk-... |
| **Endpoint** | URL personalizada (opcional) | https://api.custom.com |

**Paso 3**: Hacer clic en **Agregar Proveedor**

#### Proveedores Soportados

| Proveedor | Tipo | Configuración |
|-----------|------|---------------|
| **OpenAI/GPT** | Cloud | Clave API de OpenAI |
| **Google Gemini** | Cloud | Clave API de Google |
| **Claude** | Cloud | Clave API de Anthropic |
| **Grok** | Cloud | Clave API de xAI |
| **Meta Llama** | Cloud | Clave API de Meta |
| **Qwen** | Cloud | Clave API de Alibaba |
| **DeepSeek** | Cloud | Clave API de DeepSeek |
| **Perplexity** | Cloud | Clave API de Perplexity |
| **Copilot** | Cloud | Token de Microsoft |
| **Ollama** | Local | Endpoint local |
| **LMstudio** | Local | Endpoint local |

#### Obtener Claves API

**OpenAI**:
```
1. Ir a https://platform.openai.com/api-keys
2. Crear nueva clave
3. Copiar y pegar en el panel
```

**Google Gemini**:
```
1. Ir a https://makersuite.google.com/app/apikey
2. Crear clave API
3. Copiar y pegar en el panel
```

**Claude**:
```
1. Ir a https://console.anthropic.com/
2. Crear nueva clave
3. Copiar y pegar en el panel
```

**Ollama (Local)**:
```
1. Instalar Ollama: https://ollama.ai
2. Ejecutar: ollama serve
3. Endpoint: http://localhost:11434
```

**LMstudio (Local)**:
```
1. Descargar: https://lmstudio.ai
2. Ejecutar servidor
3. Endpoint: http://localhost:1234
```

#### Probar Proveedor

```
1. Hacer clic en "Probar" en la tarjeta del proveedor
2. Esperar respuesta
3. Ver latencia y estado
```

---

### 2️⃣ Bases de Datos

#### Agregar Nueva Base de Datos

**Paso 1**: Ir a **Bases de Datos** → **Agregar Nueva Base de Datos**

**Paso 2**: Completar formulario

| Campo | Descripción | Ejemplo |
|-------|-------------|---------|
| **Nombre** | Nombre de la BD | Mi Supabase |
| **Tipo** | Tipo de BD | supabase, notion, etc |
| **Credenciales** | Token/Contraseña | Token de autenticación |

**Paso 3**: Hacer clic en **Agregar Base de Datos**

#### Bases de Datos Soportadas

| BD | Tipo | Configuración |
|----|------|---------------|
| **Supabase** | PostgreSQL Cloud | URL + Clave API |
| **Notion** | Knowledge Base | Token de integración |
| **Obsidian** | Local/Sync | Token de sincronización |
| **NotebookLM** | AI Notes | Token de Google |
| **Database.com** | Salesforce | Token SOQL |

#### Obtener Credenciales

**Supabase**:
```
1. Ir a https://supabase.com
2. Crear proyecto
3. Ir a Settings → API
4. Copiar "anon public" key
```

**Notion**:
```
1. Ir a https://www.notion.so/my-integrations
2. Crear nueva integración
3. Copiar "Internal Integration Token"
```

**Obsidian**:
```
1. Instalar Obsidian: https://obsidian.md
2. Ir a Settings → Sync
3. Copiar token de sincronización
```

**NotebookLM**:
```
1. Ir a https://notebooklm.google.com
2. Obtener token de Google
3. Configurar acceso
```

#### Extraer Datos para Entrenamiento

```
1. Hacer clic en "Extraer Datos" en la BD
2. Especificar consulta (opcional)
3. Establecer límite de registros
4. Los datos se guardan para entrenamiento
```

---

### 3️⃣ Webhooks

#### Ver Webhooks Configurados

```
1. Ir a "Webhooks"
2. Ver lista de webhooks activos
3. Verificar URL y estado
```

#### Crear Nuevo Webhook

```
1. Hacer clic en "Nuevo Webhook"
2. Configurar:
   - Evento (sync, retraining, etc)
   - URL de destino
   - Método (POST, PUT, etc)
3. Guardar
```

---

## 🔄 Flujo de Extracción de Datos

### Paso 1: Conectar Base de Datos

```
Panel Admin → Bases de Datos → Agregar Nueva → Configurar
```

### Paso 2: Probar Conexión

```
Hacer clic en "Probar" → Verificar estado
```

### Paso 3: Extraer Datos

```
Hacer clic en "Extraer Datos" → Especificar consulta → Extraer
```

### Paso 4: Usar para Entrenamiento

```
Los datos se guardan automáticamente en la tabla "training_data"
Se pueden usar para fine-tuning del modelo
```

---

## 📊 Estadísticas del Panel

### Ver Estadísticas

```
Ir a "Estadísticas" → Ver:
- Número de proveedores configurados
- Número de BDs conectadas
- Webhooks activos
- Datos extraídos
- Última sincronización
```

---

## 🔐 Seguridad

### Proteger Claves API

✅ **Hacer**:
- Usar variables de entorno
- Cambiar claves regularmente
- Limitar permisos de claves
- Auditar acceso

❌ **No hacer**:
- Compartir claves en chat
- Guardar en archivos de texto
- Usar claves genéricas
- Dejar claves expiradas

### Roles y Permisos

```
Admin: Acceso completo
Editor: Puede agregar/editar proveedores
Viewer: Solo lectura
```

---

## 🚨 Problemas Comunes

### "Conexión rechazada"

**Causa**: Credenciales inválidas

**Solución**:
1. Verificar clave API
2. Verificar que no esté expirada
3. Verificar permisos en el proveedor
4. Probar en otra aplicación

### "Base de datos no encontrada"

**Causa**: Endpoint incorrecto

**Solución**:
1. Verificar URL/endpoint
2. Verificar que esté activa
3. Probar conexión manual

### "Extracción lenta"

**Causa**: Demasiados datos

**Solución**:
1. Reducir límite de registros
2. Usar filtros más específicos
3. Ejecutar en horarios de bajo uso

---

## 📈 Mejores Prácticas

### 1. Organizar Proveedores

```
Agrupar por tipo:
- Cloud: OpenAI, Gemini, Claude
- Local: Ollama, LMstudio
- Especializados: Perplexity, NotebookLM
```

### 2. Monitorear Uso

```
Revisar regularmente:
- Latencia de proveedores
- Uso de cuotas de API
- Errores de conexión
```

### 3. Rotación de Claves

```
Cada 3 meses:
- Generar nuevas claves
- Actualizar en panel
- Eliminar claves antiguas
```

### 4. Respaldo de Datos

```
Antes de extraer datos:
- Hacer respaldo de BD
- Verificar integridad
- Probar restauración
```

---

## 📞 Ejemplos de Uso

### Ejemplo 1: Usar Múltiples Proveedores

```
1. Agregar OpenAI (primario)
2. Agregar Gemini (fallback)
3. Agregar Ollama (local)
4. El sistema intenta en orden
```

### Ejemplo 2: Extraer Datos de Notion

```
1. Conectar Notion
2. Especificar base de datos
3. Extraer últimos 1000 registros
4. Usar para entrenar modelo
```

### Ejemplo 3: Sincronización Automática

```
1. Configurar webhook
2. Establecer evento "sync"
3. URL: https://tu-app.com/api/sync
4. Cada cambio dispara sincronización
```

---

## ✅ Checklist

- [ ] Panel de administración accesible
- [ ] Al menos 1 proveedor de IA configurado
- [ ] Al menos 1 BD externa conectada
- [ ] Prueba de conexión exitosa
- [ ] Extracción de datos funcionando
- [ ] Webhooks configurados
- [ ] Claves API protegidas
- [ ] Monitoreo en lugar

---

**¡Panel de Administración configurado y listo!** ✅

