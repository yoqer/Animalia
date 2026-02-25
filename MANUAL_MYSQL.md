# 📘 Manual de Configuración - MySQL

**Para Animalia Web Hosting con MySQL**

---

## 🎯 Requisitos

- MySQL 5.7 o superior
- Acceso a usuario root o administrador
- Credenciales de conexión

---

## 🚀 Despliegue Automático (Recomendado)

### Paso 1: Ejecutar script de despliegue

```bash
cd /ruta/a/animalia_web_hosting
./deploy-multi-db.sh mysql producción
```

**¿Qué hace?**
- ✅ Instala todas las dependencias
- ✅ Crea base de datos `animalia`
- ✅ Crea usuario `animalia_user`
- ✅ Aplica migraciones
- ✅ Compila el proyecto
- ✅ Inicia el servidor

**Tiempo**: 10-15 minutos

---

## 🔧 Despliegue Manual

### Paso 1: Crear archivo .env

```bash
cat > .env << EOF
DATABASE_URL=mysql://usuario:contraseña@localhost:3306/animalia
DB_TYPE=mysql
JWT_SECRET=tu_secreto_aleatorio_aqui
VITE_APP_ID=tu_app_id
OAUTH_SERVER_URL=https://api.manus.im
VITE_OAUTH_PORTAL_URL=https://login.manus.im
VITE_FRONTEND_FORGE_API_URL=http://localhost:3000
BUILT_IN_FORGE_API_URL=http://localhost:3000
VITE_FRONTEND_FORGE_API_KEY=tu_api_key_aqui
BUILT_IN_FORGE_API_KEY=tu_api_key_aqui
OWNER_NAME=Tu Nombre
OWNER_OPEN_ID=tu_open_id
VITE_ANALYTICS_ENDPOINT=http://localhost:3000/analytics
VITE_ANALYTICS_WEBSITE_ID=animalia
VITE_APP_TITLE=Animalia - Sistema de Comunicación Multi-Especies
VITE_APP_LOGO=/logo.png
EOF
```

### Paso 2: Crear base de datos y usuario

```bash
# Conectar a MySQL
mysql -u root -p

# Dentro de MySQL:
CREATE DATABASE animalia CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
CREATE USER 'animalia_user'@'localhost' IDENTIFIED BY 'contraseña_segura';
GRANT ALL PRIVILEGES ON animalia.* TO 'animalia_user'@'localhost';
FLUSH PRIVILEGES;
EXIT;
```

### Paso 3: Actualizar .env con credenciales reales

```
DATABASE_URL=mysql://animalia_user:contraseña_segura@localhost:3306/animalia
```

### Paso 4: Instalar dependencias

```bash
pnpm install
pnpm add mysql2 drizzle-orm
```

### Paso 5: Aplicar migraciones

```bash
pnpm exec drizzle-kit generate
pnpm exec drizzle-kit migrate
```

### Paso 6: Compilar y ejecutar

```bash
pnpm run build
pnpm run dev
```

---

## 📊 Verificar Instalación

### Conectar a la BD

```bash
mysql -u animalia_user -p animalia
```

### Ver tablas creadas

```sql
SHOW TABLES;

# Deberías ver:
# animal_patterns
# conversations
# knowledge
# retraining_requests
# sync_history
# users
```

### Ver estructura de tabla

```sql
DESCRIBE users;
DESCRIBE animal_patterns;
```

---

## 🔐 Seguridad en Hosting Remoto

Si usas hosting remoto (no localhost):

### 1. Obtener credenciales del hosting

El hosting te proporciona:
- **Host**: `maxxine.net` o similar
- **Usuario**: `qamo038` o similar
- **Contraseña**: Tu contraseña
- **Puerto**: Generalmente `3306`

### 2. Actualizar .env

```
DATABASE_URL=mysql://qamo038:tu_contraseña@maxxine.net:3306/qamo038
```

### 3. Verificar conexión remota

```bash
mysql -h maxxine.net -u qamo038 -p -e "SELECT 1;"
```

---

## 📈 Optimización para Producción

### 1. Configurar pool de conexiones

En `.env`:
```
DATABASE_POOL_SIZE=20
DATABASE_IDLE_TIMEOUT=30000
```

### 2. Crear índices

```sql
-- Índices para búsquedas rápidas
CREATE INDEX idx_user_openid ON users(openId);
CREATE INDEX idx_pattern_animal ON animal_patterns(animal_type);
CREATE INDEX idx_sync_timestamp ON sync_history(createdAt);
```

### 3. Configurar backups automáticos

```bash
# Backup diario
0 2 * * * mysqldump -u animalia_user -p animalia > /backups/animalia_$(date +\%Y\%m\%d).sql
```

---

## 🚨 Problemas Comunes

### "Access denied for user"

**Causa**: Credenciales incorrectas

**Solución**:
```bash
# Verificar credenciales
mysql -u animalia_user -p animalia -e "SELECT 1;"

# Resetear contraseña
mysql -u root -p
ALTER USER 'animalia_user'@'localhost' IDENTIFIED BY 'nueva_contraseña';
FLUSH PRIVILEGES;
```

### "Can't connect to MySQL server"

**Causa**: MySQL no está ejecutándose

**Solución**:
```bash
# Iniciar MySQL
sudo service mysql start

# Verificar estado
sudo service mysql status
```

### "Unknown database 'animalia'"

**Causa**: Base de datos no existe

**Solución**:
```bash
mysql -u root -p
CREATE DATABASE animalia CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
```

---

## 📞 Archivos Clave

| Archivo | Propósito |
|---------|-----------|
| `.env` | Configuración (credenciales) |
| `deploy-multi-db.sh` | Script de despliegue automático |
| `server/db-config.ts` | Configuración de BD |
| `drizzle/schema.ts` | Definición de tablas |

---

## ✅ Checklist

- [ ] MySQL instalado y ejecutándose
- [ ] Base de datos `animalia` creada
- [ ] Usuario `animalia_user` creado
- [ ] Archivo `.env` configurado
- [ ] Dependencias instaladas (`pnpm install`)
- [ ] Migraciones aplicadas (`pnpm exec drizzle-kit migrate`)
- [ ] Servidor inicia correctamente (`pnpm run dev`)
- [ ] Puedes acceder a `http://localhost:3000`

---

**¡Listo! Animalia está funcionando con MySQL.** ✅

