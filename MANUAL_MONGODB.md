# 📘 Manual de Configuración - MongoDB

**Para Animalia Web Hosting con MongoDB (NoSQL)**

---

## 🎯 ¿Por qué MongoDB?

- ✅ **Flexible**: Sin esquema rígido
- ✅ **Escalable**: Ideal para datos no estructurados
- ✅ **Rápido**: Búsquedas rápidas
- ✅ **Nube**: MongoDB Atlas gratuito disponible

---

## 🚀 Despliegue Automático (Recomendado)

### Paso 1: Ejecutar script de despliegue

```bash
cd /ruta/a/animalia_web_hosting
./deploy-multi-db.sh mongodb producción
```

**¿Qué hace?**
- ✅ Instala dependencias de MongoDB
- ✅ Crea base de datos `animalia`
- ✅ Crea colecciones necesarias
- ✅ Configura índices
- ✅ Inicia el servidor

---

## 🔧 Despliegue Manual

### Opción A: MongoDB Local

#### Paso 1: Instalar MongoDB

**Windows**:
```bash
# Descargar desde: https://www.mongodb.com/try/download/community
# Ejecutar instalador
```

**Mac**:
```bash
brew tap mongodb/brew
brew install mongodb-community
brew services start mongodb-community
```

**Linux**:
```bash
sudo apt-get install -y mongodb
sudo systemctl start mongodb
```

#### Paso 2: Crear archivo .env

```bash
cat > .env << EOF
DATABASE_URL=mongodb://localhost:27017/animalia
DB_TYPE=mongodb
JWT_SECRET=tu_secreto_aleatorio_aqui
VITE_APP_ID=tu_app_id
OAUTH_SERVER_URL=https://api.manus.im
...
EOF
```

#### Paso 3: Crear base de datos y colecciones

```bash
# Conectar a MongoDB
mongosh

# Dentro de mongosh:
use animalia

# Crear colecciones
db.createCollection("users")
db.createCollection("animal_patterns")
db.createCollection("knowledge")
db.createCollection("conversations")
db.createCollection("retraining_requests")
db.createCollection("sync_history")

# Crear índices
db.users.createIndex({ openId: 1 }, { unique: true })
db.animal_patterns.createIndex({ animal_type: 1 })
db.sync_history.createIndex({ createdAt: -1 })

exit
```

---

### Opción B: MongoDB Atlas (Nube - Recomendado)

#### Paso 1: Crear cuenta en MongoDB Atlas

1. Ir a: https://www.mongodb.com/cloud/atlas
2. Crear cuenta gratuita
3. Crear cluster gratuito

#### Paso 2: Obtener conexión

En MongoDB Atlas:
1. Ir a "Clusters"
2. Hacer clic en "Connect"
3. Copiar URL de conexión

**Ejemplo**:
```
mongodb+srv://usuario:contraseña@cluster.mongodb.net/animalia?retryWrites=true&w=majority
```

#### Paso 3: Actualizar .env

```
DATABASE_URL=mongodb+srv://usuario:contraseña@cluster.mongodb.net/animalia?retryWrites=true&w=majority
DB_TYPE=mongodb
```

#### Paso 4: Instalar dependencias

```bash
pnpm install
pnpm add mongodb drizzle-orm
```

#### Paso 5: Ejecutar

```bash
pnpm run dev
```

---

## 📊 Verificar Instalación

### Conectar a MongoDB

```bash
# Local
mongosh

# Remoto (Atlas)
mongosh "mongodb+srv://usuario:contraseña@cluster.mongodb.net/animalia"
```

### Ver colecciones

```javascript
use animalia
show collections

// Deberías ver:
// animal_patterns
// conversations
// knowledge
// retraining_requests
// sync_history
// users
```

### Ver documentos

```javascript
db.users.find().limit(5)
db.animal_patterns.find().limit(5)
```

---

## 📈 Optimización para Producción

### 1. Crear índices

```javascript
// Índices para búsquedas rápidas
db.users.createIndex({ openId: 1 }, { unique: true })
db.users.createIndex({ email: 1 })
db.animal_patterns.createIndex({ animal_type: 1 })
db.animal_patterns.createIndex({ createdAt: -1 })
db.sync_history.createIndex({ createdAt: -1 })
db.sync_history.createIndex({ userId: 1 })
```

### 2. Configurar TTL (Time To Live)

```javascript
// Eliminar documentos automáticamente después de 30 días
db.sync_history.createIndex({ createdAt: 1 }, { expireAfterSeconds: 2592000 })
```

### 3. Configurar replicación (Atlas)

En MongoDB Atlas:
1. Ir a "Replication"
2. Aumentar número de replicas a 3
3. Distribuir en diferentes regiones

---

## 🔐 Seguridad

### 1. Crear usuario con permisos limitados

```javascript
db.createUser({
  user: "animalia_user",
  pwd: "contraseña_segura",
  roles: [
    { role: "readWrite", db: "animalia" }
  ]
})
```

### 2. Actualizar .env

```
DATABASE_URL=mongodb://animalia_user:contraseña_segura@localhost:27017/animalia
```

### 3. En Atlas: Whitelist IPs

1. Ir a "Network Access"
2. Agregar IP de tu servidor
3. Restringir a solo esas IPs

---

## 📊 Estructura de Datos

### Colección: users

```javascript
{
  _id: ObjectId(),
  openId: "usuario123",
  name: "Juan Pérez",
  email: "juan@example.com",
  role: "user",
  createdAt: ISODate("2026-02-24T07:00:00Z"),
  updatedAt: ISODate("2026-02-24T07:00:00Z")
}
```

### Colección: animal_patterns

```javascript
{
  _id: ObjectId(),
  userId: ObjectId(),
  animal_type: "Canis lupus",
  pattern_name: "Comportamiento de caza",
  description: "Patrón de comportamiento de caza en manada",
  data: { ... },
  createdAt: ISODate(),
  updatedAt: ISODate()
}
```

---

## 🚨 Problemas Comunes

### "connect ECONNREFUSED 127.0.0.1:27017"

**Causa**: MongoDB no está ejecutándose

**Solución**:
```bash
# Iniciar MongoDB
mongosh

# O en Linux
sudo systemctl start mongodb
```

### "Authentication failed"

**Causa**: Credenciales incorrectas

**Solución**:
```javascript
// Verificar usuario
db.getUser("animalia_user")

// Resetear contraseña
db.changeUserPassword("animalia_user", "nueva_contraseña")
```

### "Database not found"

**Causa**: Base de datos no existe

**Solución**:
```javascript
use animalia
db.createCollection("users")
```

---

## 📞 Archivos Clave

| Archivo | Propósito |
|---------|-----------|
| `.env` | Configuración (URL de MongoDB) |
| `deploy-multi-db.sh` | Script de despliegue automático |
| `server/db-config.ts` | Configuración de BD |

---

## ✅ Checklist

- [ ] MongoDB instalado o Atlas configurado
- [ ] Archivo `.env` con URL de MongoDB
- [ ] Colecciones creadas
- [ ] Índices configurados
- [ ] Dependencias instaladas (`pnpm install`)
- [ ] Servidor inicia correctamente (`pnpm run dev`)
- [ ] Puedes acceder a `http://localhost:3000`

---

## 🎓 Comparativa: MongoDB vs MySQL

| Aspecto | MongoDB | MySQL |
|--------|---------|-------|
| **Tipo** | NoSQL | SQL |
| **Esquema** | Flexible | Rígido |
| **Escalabilidad** | Horizontal | Vertical |
| **Transacciones** | Soportadas (v4.0+) | Soportadas |
| **Consultas** | JSON | SQL |
| **Ideal para** | Datos no estructurados | Datos estructurados |

---

**¡Listo! Animalia está funcionando con MongoDB.** ✅

