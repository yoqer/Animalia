# 🧠 Investigación: Sistemas de Memoria en Asistentes IA
## Crawbot, Garras, OpenClaw y Extracción de Datos para Entrenamiento

---

## 📋 Tabla de Contenidos

1. [Introducción](#introducción)
2. [OpenClaw - Arquitectura General](#openclaw---arquitectura-general)
3. [Garras - El Asistente Local](#garras---el-asistente-local)
4. [Sistemas de Memoria](#sistemas-de-memoria)
5. [Extracción de Datos](#extracción-de-datos)
6. [Entrenamiento con Datos Ordenados](#entrenamiento-con-datos-ordenados)
7. [Despliegue en Docker](#despliegue-en-docker)
8. [Integración con Animalia](#integración-con-animalia)

---

## Introducción

### ¿Qué es OpenClaw?

**OpenClaw** es un asistente de IA personal que se ejecuta en dispositivos locales. Características principales:

- **Multi-canal**: WhatsApp, Telegram, Slack, Discord, Google Chat, Signal, iMessage, Microsoft Teams, Matrix, Zalo
- **Local-first**: Control plane en Gateway WebSocket local
- **Multi-agente**: Enrutamiento de múltiples agentes aislados
- **Voz**: Soporte para macOS/iOS/Android con ElevenLabs
- **Canvas en Vivo**: Interfaz visual controlada por el agente

### ¿Qué es Garras?

**Garras** (basado en OpenClaw) es el asistente local más "simpático" para uso completo local y refuerzo online de modelos.

- **Repositorio**: GitHub.com/Trompetilla/Garras
- **Descripción**: "El Cangrejo Abierto más Simpático"
- **Enfoque**: Uso local + refuerzo online
- **Distribución**: ZIP completo para instalación local (`2-ClawAbierto_Local.zip`)

---

## OpenClaw - Arquitectura General

### Componentes Principales

```
┌─────────────────────────────────────────────────────────────┐
│                    OPENCLAW ARCHITECTURE                     │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  Canales de Entrada (Channels)                              │
│  ├─ WhatsApp (Baileys)                                      │
│  ├─ Telegram (grammY)                                       │
│  ├─ Slack (Bolt)                                            │
│  ├─ Discord (discord.js)                                    │
│  ├─ Google Chat                                             │
│  ├─ Signal                                                  │
│  ├─ iMessage (BlueBubbles/Legacy)                           │
│  ├─ Microsoft Teams                                         │
│  ├─ Matrix                                                  │
│  └─ WebChat                                                 │
│                    ↓                                          │
│  ┌──────────────────────────────────────────────────────┐   │
│  │  Gateway (Control Plane - WebSocket)                │   │
│  │  - Sessions management                              │   │
│  │  - Presence tracking                                │   │
│  │  - Config management                                │   │
│  │  - Cron jobs                                         │   │
│  │  - Webhooks                                          │   │
│  │  - Control UI                                        │   │
│  │  - Canvas host                                       │   │
│  └──────────────────────────────────────────────────────┘   │
│                    ↓                                          │
│  Clientes Conectados                                         │
│  ├─ Pi Agent Runtime (RPC mode)                             │
│  ├─ CLI (openclaw ...)                                      │
│  ├─ WebChat UI                                              │
│  ├─ macOS App (menu bar)                                    │
│  └─ iOS/Android Nodes                                       │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

### Instalación Recomendada

```bash
# Requisitos
Node ≥ 22

# Instalación global
npm install -g openclaw@latest
# o
pnpm add -g openclaw@latest

# Configuración automática (recomendado)
openclaw onboard --install-daemon

# Esto instala el daemon del Gateway (launchd/systemd)
```

### Inicio Rápido

```bash
# Iniciar Gateway
openclaw gateway --port 18789 --verbose

# Enviar mensaje
openclaw message send --to +1234567890 --message "Hola desde OpenClaw"

# Hablar con el asistente
openclaw agent --message "Checklist de envío" --thinking high
```

---

## Garras - El Asistente Local

### Características Principales

**Garras** es una versión mejorada de OpenClaw enfocada en:

1. **Uso Local Completo**: Funciona sin conexión a internet
2. **Refuerzo Online**: Puede conectarse para mejorar modelos
3. **Interfaz Amigable**: Diseño "simpático" para usuarios
4. **Modelos Locales**: Soporte para Ollama, LMstudio
5. **Entrenamiento Local**: Fine-tuning en máquina local

### Estructura de Directorios

```
Garras/
├── .agent/                    # Configuración de agentes
├── src/                       # Código fuente TypeScript
│   ├── gateway/              # Control plane WebSocket
│   ├── agents/               # Runtime de agentes
│   ├── channels/             # Integraciones de canales
│   ├── tools/                # Herramientas disponibles
│   ├── memory/               # Sistema de memoria
│   └── training/             # Módulo de entrenamiento
├── docker/                    # Configuraciones Docker
│   ├── Dockerfile
│   ├── Dockerfile.sandbox
│   └── docker-compose.yml
├── docs/                      # Documentación
├── MANUAL_LOCAL.md           # Manual para uso local
├── AGENTS.md                 # Documentación de agentes
├── CLAUDE.md                 # Integración Claude
└── package.json              # Dependencias Node
```

### Archivos Clave para Despliegue

```
Garras/
├── docker-compose.yml        # Orquestación Docker
├── docker-setup.sh           # Script de configuración
├── .env.example              # Variables de entorno
├── package.json              # Dependencias
├── pnpm-workspace.yaml       # Workspace de pnpm
└── openclaw.mjs              # Punto de entrada
```

---

## Sistemas de Memoria

### Tipos de Memoria en OpenClaw/Garras

#### 1. **Session Memory** (Memoria de Sesión)

```typescript
// Almacenamiento: En memoria + persistencia opcional
interface Session {
  id: string;
  userId: string;
  channelId: string;
  messages: Message[];
  metadata: Record<string, any>;
  createdAt: Date;
  updatedAt: Date;
  expiresAt?: Date;
}

// Ubicación de almacenamiento:
// - Memoria local: ~/.openclaw/sessions/
// - Base de datos: SQLite/PostgreSQL (opcional)
// - Redis: Para caché distribuido (opcional)
```

#### 2. **User Memory** (Memoria de Usuario)

```typescript
// Perfil y preferencias del usuario
interface UserProfile {
  id: string;
  name: string;
  email?: string;
  preferences: {
    language: string;
    timezone: string;
    modelPreference: string;
    channels: string[];
  };
  conversationHistory: ConversationEntry[];
  customTools: Tool[];
  skills: Skill[];
}

// Ubicación:
// ~/.openclaw/users/
// ~/.openclaw/profiles/
```

#### 3. **Conversation Memory** (Memoria de Conversación)

```typescript
// Historial de conversaciones
interface ConversationEntry {
  id: string;
  timestamp: Date;
  channel: string;
  sender: string;
  message: string;
  response: string;
  context: {
    tools_used: string[];
    model: string;
    thinking_time: number;
  };
  embedding?: number[]; // Para búsqueda semántica
}

// Almacenamiento:
// ~/.openclaw/conversations/
// Base de datos: Búsqueda por embedding
```

#### 4. **Tool Memory** (Memoria de Herramientas)

```typescript
// Estado de herramientas y resultados
interface ToolMemory {
  toolId: string;
  lastUsed: Date;
  executionCount: number;
  successRate: number;
  cachedResults: Map<string, any>;
  configuration: Record<string, any>;
}

// Ubicación:
// ~/.openclaw/tools/
```

#### 5. **Training Data Memory** (Memoria de Datos de Entrenamiento)

```typescript
// Datos para fine-tuning
interface TrainingDataset {
  id: string;
  name: string;
  description: string;
  entries: TrainingEntry[];
  metadata: {
    created: Date;
    updated: Date;
    version: string;
    format: 'jsonl' | 'csv' | 'parquet';
  };
  statistics: {
    totalTokens: number;
    averageLength: number;
    categories: Record<string, number>;
  };
}

interface TrainingEntry {
  prompt: string;
  completion: string;
  category?: string;
  weight?: number;
  metadata?: Record<string, any>;
}

// Ubicación:
// ~/.openclaw/training/
// ~/.openclaw/datasets/
```

### Ubicaciones de Almacenamiento

```
~/.openclaw/
├── gateway/
│   ├── config.json           # Configuración del Gateway
│   ├── sessions/             # Sesiones activas
│   │   ├── session_1.json
│   │   └── session_2.json
│   └── webhooks/             # Configuración de webhooks
├── agents/
│   ├── agent_1/
│   │   ├── config.json
│   │   ├── memory.json
│   │   └── tools.json
│   └── agent_2/
├── users/
│   ├── user_1/
│   │   ├── profile.json
│   │   ├── preferences.json
│   │   └── history.jsonl
│   └── user_2/
├── conversations/
│   ├── 2024-02/
│   │   ├── conv_001.jsonl
│   │   └── conv_002.jsonl
│   └── 2024-03/
├── training/
│   ├── dataset_1/
│   │   ├── data.jsonl
│   │   ├── metadata.json
│   │   └── embeddings.bin
│   └── dataset_2/
├── tools/
│   ├── browser/
│   │   ├── cache/
│   │   └── profiles/
│   └── canvas/
├── models/
│   ├── local/
│   │   ├── ollama/
│   │   └── lmstudio/
│   └── cache/
└── logs/
    ├── gateway.log
    ├── agents.log
    └── errors.log
```

---

## Extracción de Datos

### Métodos de Extracción

#### 1. **Exportar Sesiones**

```bash
# CLI de OpenClaw
openclaw export sessions --output sessions.json

# O manualmente
cp -r ~/.openclaw/sessions/ ./backup/sessions/
```

#### 2. **Exportar Conversaciones**

```bash
# Extraer todas las conversaciones
openclaw export conversations \
  --format jsonl \
  --output conversations.jsonl \
  --start-date 2024-01-01 \
  --end-date 2024-02-25

# Resultado: JSONL con formato
{
  "id": "conv_001",
  "timestamp": "2024-02-25T10:30:00Z",
  "channel": "telegram",
  "sender": "user_123",
  "message": "¿Cuál es el clima?",
  "response": "El clima es soleado...",
  "context": {
    "tools_used": ["weather_api"],
    "model": "claude-3-opus",
    "thinking_time": 1.2
  }
}
```

#### 3. **Exportar Datos de Entrenamiento**

```bash
# Extraer dataset de entrenamiento
openclaw export training \
  --dataset dataset_1 \
  --format jsonl \
  --output training_data.jsonl

# Resultado: JSONL con prompts y completions
{
  "prompt": "¿Cuál es la capital de Francia?",
  "completion": "La capital de Francia es París.",
  "category": "geography",
  "weight": 1.0
}
```

#### 4. **Extraer Memoria de Usuario**

```bash
# Exportar perfil de usuario
openclaw export user \
  --user-id user_123 \
  --output user_profile.json

# Resultado JSON
{
  "id": "user_123",
  "name": "Juan",
  "preferences": {
    "language": "es",
    "model": "claude-3-opus"
  },
  "conversationHistory": [...]
}
```

#### 5. **Script Python para Extracción Completa**

```python
#!/usr/bin/env python3
"""
Extractor de datos de OpenClaw/Garras
Extrae sesiones, conversaciones, usuarios y datos de entrenamiento
"""

import json
import os
from pathlib import Path
from datetime import datetime
from typing import List, Dict, Any

class GarrasDataExtractor:
    def __init__(self, openclaw_home: str = None):
        self.openclaw_home = openclaw_home or os.path.expanduser("~/.openclaw")
        self.base_path = Path(self.openclaw_home)
    
    def extract_sessions(self) -> List[Dict[str, Any]]:
        """Extrae todas las sesiones"""
        sessions = []
        sessions_path = self.base_path / "gateway" / "sessions"
        
        if sessions_path.exists():
            for session_file in sessions_path.glob("*.json"):
                with open(session_file, 'r') as f:
                    sessions.append(json.load(f))
        
        return sessions
    
    def extract_conversations(self, start_date: str = None, end_date: str = None) -> List[Dict]:
        """Extrae conversaciones con filtro de fecha"""
        conversations = []
        conv_path = self.base_path / "conversations"
        
        if conv_path.exists():
            for month_dir in conv_path.iterdir():
                if month_dir.is_dir():
                    for conv_file in month_dir.glob("*.jsonl"):
                        with open(conv_file, 'r') as f:
                            for line in f:
                                conv = json.loads(line)
                                # Filtrar por fecha si es necesario
                                if self._check_date_range(conv.get("timestamp"), start_date, end_date):
                                    conversations.append(conv)
        
        return conversations
    
    def extract_training_data(self, dataset_name: str = None) -> List[Dict]:
        """Extrae datos de entrenamiento"""
        training_data = []
        training_path = self.base_path / "training"
        
        if training_path.exists():
            for dataset_dir in training_path.iterdir():
                if dataset_dir.is_dir():
                    if dataset_name and dataset_dir.name != dataset_name:
                        continue
                    
                    data_file = dataset_dir / "data.jsonl"
                    if data_file.exists():
                        with open(data_file, 'r') as f:
                            for line in f:
                                training_data.append(json.loads(line))
        
        return training_data
    
    def extract_user_profiles(self) -> List[Dict]:
        """Extrae perfiles de usuarios"""
        profiles = []
        users_path = self.base_path / "users"
        
        if users_path.exists():
            for user_dir in users_path.iterdir():
                if user_dir.is_dir():
                    profile_file = user_dir / "profile.json"
                    if profile_file.exists():
                        with open(profile_file, 'r') as f:
                            profiles.append(json.load(f))
        
        return profiles
    
    def export_all(self, output_dir: str = "./garras_export"):
        """Exporta todos los datos a archivos"""
        os.makedirs(output_dir, exist_ok=True)
        
        # Exportar sesiones
        sessions = self.extract_sessions()
        with open(f"{output_dir}/sessions.json", 'w') as f:
            json.dump(sessions, f, indent=2)
        print(f"✓ Sesiones exportadas: {len(sessions)}")
        
        # Exportar conversaciones
        conversations = self.extract_conversations()
        with open(f"{output_dir}/conversations.jsonl", 'w') as f:
            for conv in conversations:
                f.write(json.dumps(conv) + '\n')
        print(f"✓ Conversaciones exportadas: {len(conversations)}")
        
        # Exportar datos de entrenamiento
        training_data = self.extract_training_data()
        with open(f"{output_dir}/training_data.jsonl", 'w') as f:
            for entry in training_data:
                f.write(json.dumps(entry) + '\n')
        print(f"✓ Datos de entrenamiento exportados: {len(training_data)}")
        
        # Exportar perfiles de usuario
        profiles = self.extract_user_profiles()
        with open(f"{output_dir}/user_profiles.json", 'w') as f:
            json.dump(profiles, f, indent=2)
        print(f"✓ Perfiles de usuario exportados: {len(profiles)}")
    
    def _check_date_range(self, timestamp: str, start_date: str, end_date: str) -> bool:
        """Verifica si timestamp está en rango de fechas"""
        if not start_date and not end_date:
            return True
        
        try:
            ts = datetime.fromisoformat(timestamp.replace('Z', '+00:00'))
            if start_date:
                start = datetime.fromisoformat(start_date)
                if ts < start:
                    return False
            if end_date:
                end = datetime.fromisoformat(end_date)
                if ts > end:
                    return False
            return True
        except:
            return True

# Uso
if __name__ == "__main__":
    extractor = GarrasDataExtractor()
    extractor.export_all()
```

---

## Entrenamiento con Datos Ordenados

### Estructura de Datos para Entrenamiento

```json
{
  "dataset_id": "dataset_comportamiento_animal",
  "name": "Comportamiento Animal - Dataset",
  "version": "1.0",
  "created": "2024-02-25",
  "entries": [
    {
      "id": "entry_001",
      "prompt": "¿Cuál es el comportamiento de un león cuando caza?",
      "completion": "El león es un depredador que caza en grupo...",
      "category": "comportamiento_depredador",
      "species": "Panthera leo",
      "weight": 1.0,
      "metadata": {
        "source": "wikipedia",
        "verified": true,
        "confidence": 0.95
      }
    },
    {
      "id": "entry_002",
      "prompt": "¿Cómo se comunican los delfines?",
      "completion": "Los delfines utilizan clicks y silbidos...",
      "category": "comunicacion_animal",
      "species": "Tursiops truncatus",
      "weight": 1.0,
      "metadata": {
        "source": "research_paper",
        "verified": true,
        "confidence": 0.98
      }
    }
  ],
  "statistics": {
    "total_entries": 2,
    "total_tokens": 450,
    "average_length": 225,
    "categories": {
      "comportamiento_depredador": 1,
      "comunicacion_animal": 1
    }
  }
}
```

### Fine-tuning Local con Ollama

```bash
# 1. Preparar datos en formato JSONL
cat > training_data.jsonl << 'EOF'
{"prompt": "¿Qué es un león?", "completion": "Un león es un felino grande..."}
{"prompt": "¿Cómo caza un león?", "completion": "El león caza en grupo..."}
EOF

# 2. Convertir a formato Ollama
python3 << 'PYTHON'
import json

with open('training_data.jsonl', 'r') as f:
    with open('modelfile', 'w') as out:
        out.write('FROM llama2\n\n')
        for line in f:
            data = json.loads(line)
            out.write(f'SYSTEM {data["prompt"]}\n')
            out.write(f'RESPONSE {data["completion"]}\n\n')
PYTHON

# 3. Crear modelo personalizado
ollama create animalia-model -f modelfile

# 4. Usar el modelo
ollama run animalia-model "¿Cómo caza un león?"
```

### Fine-tuning con LMstudio

```bash
# 1. Abrir LMstudio
# 2. Cargar modelo base (ej: llama-2-7b)
# 3. Ir a "Training" tab
# 4. Cargar training_data.jsonl
# 5. Configurar parámetros:
#    - Learning rate: 0.0001
#    - Epochs: 3
#    - Batch size: 8
# 6. Iniciar entrenamiento
# 7. Guardar modelo fine-tuned
```

### Script de Entrenamiento Automatizado

```python
#!/usr/bin/env python3
"""
Script de entrenamiento automatizado para Garras
Extrae datos, prepara dataset y entrena modelo local
"""

import json
import subprocess
from pathlib import Path
from typing import List, Dict

class GarrasTrainer:
    def __init__(self, model: str = "llama2", output_dir: str = "./models"):
        self.model = model
        self.output_dir = Path(output_dir)
        self.output_dir.mkdir(exist_ok=True)
    
    def prepare_training_data(self, conversations: List[Dict]) -> str:
        """Prepara datos de conversaciones para entrenamiento"""
        training_data = []
        
        for conv in conversations:
            entry = {
                "prompt": conv.get("message", ""),
                "completion": conv.get("response", ""),
                "metadata": {
                    "channel": conv.get("channel"),
                    "timestamp": conv.get("timestamp"),
                    "tools_used": conv.get("context", {}).get("tools_used", [])
                }
            }
            training_data.append(entry)
        
        # Guardar en JSONL
        output_file = self.output_dir / "training_data.jsonl"
        with open(output_file, 'w') as f:
            for entry in training_data:
                f.write(json.dumps(entry) + '\n')
        
        return str(output_file)
    
    def train_with_ollama(self, training_file: str):
        """Entrena modelo con Ollama"""
        print(f"🚀 Iniciando entrenamiento con Ollama...")
        
        # Crear Modelfile
        modelfile_path = self.output_dir / "Modelfile"
        with open(modelfile_path, 'w') as f:
            f.write(f'FROM {self.model}\n\n')
            
            with open(training_file, 'r') as tf:
                for line in tf:
                    data = json.loads(line)
                    prompt = data.get("prompt", "").replace('"', '\\"')
                    completion = data.get("completion", "").replace('"', '\\"')
                    f.write(f'SYSTEM "{prompt}"\n')
                    f.write(f'RESPONSE "{completion}"\n\n')
        
        # Crear modelo
        model_name = f"animalia-{self.model}-finetuned"
        result = subprocess.run(
            ["ollama", "create", model_name, "-f", str(modelfile_path)],
            capture_output=True,
            text=True
        )
        
        if result.returncode == 0:
            print(f"✓ Modelo creado: {model_name}")
            return model_name
        else:
            print(f"✗ Error: {result.stderr}")
            return None
    
    def train_with_lmstudio(self, training_file: str):
        """Entrena modelo con LMstudio (requiere interfaz manual)"""
        print(f"📚 Entrenamiento con LMstudio:")
        print(f"1. Abre LMstudio")
        print(f"2. Carga el modelo: {self.model}")
        print(f"3. Ve a Training tab")
        print(f"4. Carga archivo: {training_file}")
        print(f"5. Configura parámetros:")
        print(f"   - Learning rate: 0.0001")
        print(f"   - Epochs: 3")
        print(f"   - Batch size: 8")
        print(f"6. Inicia entrenamiento")
        print(f"7. Guarda modelo fine-tuned")

# Uso
if __name__ == "__main__":
    # Extraer datos
    from garras_extractor import GarrasDataExtractor
    extractor = GarrasDataExtractor()
    conversations = extractor.extract_conversations()
    
    # Entrenar
    trainer = GarrasTrainer(model="llama2")
    training_file = trainer.prepare_training_data(conversations)
    model_name = trainer.train_with_ollama(training_file)
    
    if model_name:
        print(f"✓ Entrenamiento completado: {model_name}")
```

---

## Despliegue en Docker

### Dockerfile para Garras

```dockerfile
# Dockerfile para Garras/OpenClaw
FROM node:22-alpine

WORKDIR /app

# Instalar dependencias del sistema
RUN apk add --no-cache \
    git \
    python3 \
    make \
    g++ \
    curl

# Copiar archivos del proyecto
COPY package.json pnpm-lock.yaml ./
COPY . .

# Instalar pnpm
RUN npm install -g pnpm

# Instalar dependencias
RUN pnpm install --frozen-lockfile

# Compilar proyecto
RUN pnpm build

# Exponer puertos
EXPOSE 18789 3000

# Variables de entorno
ENV NODE_ENV=production
ENV OPENCLAW_PORT=18789

# Comando de inicio
CMD ["pnpm", "start"]
```

### Docker Compose

```yaml
version: '3.8'

services:
  garras:
    build:
      context: .
      dockerfile: Dockerfile
    container_name: garras-assistant
    ports:
      - "18789:18789"
      - "3000:3000"
    environment:
      - NODE_ENV=production
      - OPENCLAW_PORT=18789
      - LOG_LEVEL=info
    volumes:
      - garras-data:/root/.openclaw
      - garras-models:/root/.models
    restart: unless-stopped
    networks:
      - garras-network

  redis:
    image: redis:7-alpine
    container_name: garras-redis
    ports:
      - "6379:6379"
    volumes:
      - redis-data:/data
    restart: unless-stopped
    networks:
      - garras-network

  postgres:
    image: postgres:15-alpine
    container_name: garras-postgres
    environment:
      - POSTGRES_DB=garras
      - POSTGRES_USER=garras
      - POSTGRES_PASSWORD=secure_password
    ports:
      - "5432:5432"
    volumes:
      - postgres-data:/var/lib/postgresql/data
    restart: unless-stopped
    networks:
      - garras-network

volumes:
  garras-data:
  garras-models:
  redis-data:
  postgres-data:

networks:
  garras-network:
    driver: bridge
```

### Script de Despliegue Docker

```bash
#!/bin/bash
# docker-setup.sh - Script de configuración Docker para Garras

set -e

echo "🐳 Configurando Garras en Docker..."

# 1. Crear directorio de datos
mkdir -p ./garras-data/{sessions,conversations,training,users}

# 2. Crear archivo .env
cat > .env << 'EOF'
NODE_ENV=production
OPENCLAW_PORT=18789
LOG_LEVEL=info
DATABASE_URL=postgresql://garras:secure_password@postgres:5432/garras
REDIS_URL=redis://redis:6379
EOF

# 3. Construir imagen
echo "🔨 Construyendo imagen Docker..."
docker-compose build

# 4. Iniciar servicios
echo "🚀 Iniciando servicios..."
docker-compose up -d

# 5. Esperar a que esté listo
echo "⏳ Esperando a que Garras esté listo..."
sleep 10

# 6. Verificar estado
echo "✓ Garras está corriendo en http://localhost:18789"
echo "✓ Redis disponible en localhost:6379"
echo "✓ PostgreSQL disponible en localhost:5432"

# 7. Ver logs
echo ""
echo "📋 Logs en tiempo real:"
docker-compose logs -f garras
```

### Uso de Docker

```bash
# Hacer script ejecutable
chmod +x docker-setup.sh

# Ejecutar setup
./docker-setup.sh

# Ver estado
docker-compose ps

# Ver logs
docker-compose logs -f garras

# Ejecutar comando en contenedor
docker-compose exec garras openclaw agent --message "Hola"

# Detener servicios
docker-compose down

# Detener y eliminar volúmenes
docker-compose down -v
```

---

## Integración con Animalia

### Conectar Garras con Animalia Web

```typescript
// server/integrations/garras-connector.ts
import axios from 'axios';

interface GarrasConfig {
  gatewayUrl: string;
  apiKey: string;
  agentId: string;
}

export class GarrasConnector {
  private config: GarrasConfig;
  private client: axios.AxiosInstance;

  constructor(config: GarrasConfig) {
    this.config = config;
    this.client = axios.create({
      baseURL: config.gatewayUrl,
      headers: {
        'Authorization': `Bearer ${config.apiKey}`,
        'Content-Type': 'application/json'
      }
    });
  }

  // Enviar mensaje a Garras
  async sendMessage(message: string, context?: Record<string, any>) {
    const response = await this.client.post('/api/agent/message', {
      agentId: this.config.agentId,
      message,
      context
    });
    return response.data;
  }

  // Obtener conversaciones
  async getConversations(limit: number = 100) {
    const response = await this.client.get('/api/conversations', {
      params: { limit }
    });
    return response.data;
  }

  // Exportar datos de entrenamiento
  async exportTrainingData(format: 'jsonl' | 'json' = 'jsonl') {
    const response = await this.client.get('/api/export/training', {
      params: { format }
    });
    return response.data;
  }

  // Entrenar modelo
  async startTraining(datasetId: string, config: Record<string, any>) {
    const response = await this.client.post('/api/training/start', {
      datasetId,
      config
    });
    return response.data;
  }
}

// Uso en Animalia
const garrasConnector = new GarrasConnector({
  gatewayUrl: 'http://localhost:18789',
  apiKey: 'your-api-key',
  agentId: 'animalia-agent'
});

// Sincronizar conversaciones
const conversations = await garrasConnector.getConversations();
```

### Tabla de Sincronización en Animalia

```sql
-- Tabla para sincronizar datos de Garras
CREATE TABLE garras_sync (
  id INT PRIMARY KEY AUTO_INCREMENT,
  garras_agent_id VARCHAR(255) NOT NULL,
  last_sync TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  sync_status ENUM('pending', 'syncing', 'completed', 'failed'),
  conversations_count INT DEFAULT 0,
  training_entries INT DEFAULT 0,
  metadata JSON,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  UNIQUE KEY unique_agent (garras_agent_id)
);

-- Tabla para almacenar conversaciones sincronizadas
CREATE TABLE garras_conversations (
  id INT PRIMARY KEY AUTO_INCREMENT,
  garras_sync_id INT NOT NULL,
  conversation_id VARCHAR(255) UNIQUE,
  channel VARCHAR(50),
  sender_id VARCHAR(255),
  message TEXT,
  response TEXT,
  context JSON,
  created_at TIMESTAMP,
  FOREIGN KEY (garras_sync_id) REFERENCES garras_sync(id) ON DELETE CASCADE
);

-- Tabla para datos de entrenamiento sincronizados
CREATE TABLE garras_training_data (
  id INT PRIMARY KEY AUTO_INCREMENT,
  garras_sync_id INT NOT NULL,
  prompt TEXT,
  completion TEXT,
  category VARCHAR(100),
  weight FLOAT DEFAULT 1.0,
  metadata JSON,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  FOREIGN KEY (garras_sync_id) REFERENCES garras_sync(id) ON DELETE CASCADE
);
```

---

## Conclusiones

### Puntos Clave

1. **OpenClaw/Garras** es una arquitectura local-first con soporte para múltiples canales
2. **Memoria** se almacena en `~/.openclaw/` con múltiples tipos (sesiones, conversaciones, usuarios, entrenamiento)
3. **Extracción** de datos es posible mediante CLI o scripts Python
4. **Entrenamiento** local es viable con Ollama o LMstudio
5. **Docker** permite despliegue reproducible y escalable
6. **Integración** con Animalia es directa mediante APIs
