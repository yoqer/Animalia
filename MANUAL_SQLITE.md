# 📘 Manual de Configuración - SQLite

**Para Animalia Web Hosting con SQLite (Desarrollo Local)**

---

## 🎯 ¿Por qué SQLite?

- ✅ **Sin servidor**: Funciona sin instalar nada
- ✅ **Rápido**: Perfecto para desarrollo
- ✅ **Portátil**: Un solo archivo `.db`
- ✅ **Gratuito**: Completamente open-source

**Ideal para**: Desarrollo local, pruebas, aplicaciones pequeñas

---

## 🚀 Despliegue Automático (Recomendado)

### Paso 1: Ejecutar script de despliegue

```bash
cd /ruta/a/animalia_web_hosting
./deploy-multi-db.sh sqlite desarrollo
```

**¿Qué hace?**
- ✅ Crea archivo `animalia.db`
- ✅ Crea todas las tablas
- ✅ Instala dependencias
- ✅ Inicia el servidor

**Tiempo**: 5 minutos

---

## 🔧 Despliegue Manual

### Paso 1: Crear archivo .env

```bash
cat > .env << EOF
DATABASE_URL=sqlite:///home/usuario/animalia.db
DB_TYPE=sqlite
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

### Paso 2: Instalar dependencias

```bash
pnpm install
pnpm add better-sqlite3 drizzle-orm
```

### Paso 3: Crear base de datos

```bash
# SQLite se crea automáticamente
# Pero puedes crear manualmente:
sqlite3 /home/usuario/animalia.db

# Dentro de sqlite3:
CREATE TABLE users (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  openId TEXT UNIQUE NOT NULL,
  name TEXT,
  email TEXT,
  role TEXT DEFAULT 'user',
  createdAt DATETIME DEFAULT CURRENT_TIMESTAMP,
  updatedAt DATETIME DEFAULT CURRENT_TIMESTAMP
);

.quit
```

### Paso 4: Aplicar migraciones

```bash
pnpm exec drizzle-kit generate
pnpm exec drizzle-kit migrate
```

### Paso 5: Ejecutar

```bash
pnpm run dev
```

---

## 📊 Verificar Instalación

### Ver archivo de BD

```bash
ls -lh /home/usuario/animalia.db

# Deberías ver algo como:
# -rw-r--r-- 1 usuario grupo 256K Feb 24 07:00 animalia.db
```

### Conectar a SQLite

```bash
sqlite3 /home/usuario/animalia.db
```

### Ver tablas

```sql
.tables

-- Deberías ver:
-- animal_patterns  conversations  knowledge  retraining_requests  sync_history  users
```

### Ver estructura de tabla

```sql
.schema users
.schema animal_patterns
```

### Ver datos

```sql
SELECT * FROM users;
SELECT COUNT(*) FROM animal_patterns;
```

---

## 📈 Optimización para Desarrollo

### 1. Configurar WAL (Write-Ahead Logging)

```bash
sqlite3 /home/usuario/animalia.db
PRAGMA journal_mode = WAL;
PRAGMA synchronous = NORMAL;
PRAGMA cache_size = 10000;
.quit
```

**Ventajas**:
- Mejor rendimiento
- Mejor concurrencia
- Recuperación más rápida

### 2. Crear índices

```sql
CREATE INDEX idx_user_openid ON users(openId);
CREATE INDEX idx_pattern_animal ON animal_patterns(animal_type);
CREATE INDEX idx_sync_timestamp ON sync_history(createdAt);
```

### 3. Configurar VACUUM

```bash
# Optimizar BD (hacer más pequeña)
sqlite3 /home/usuario/animalia.db "VACUUM;"
```

---

## 💾 Backup y Restauración

### Backup manual

```bash
# Copiar archivo
cp /home/usuario/animalia.db /home/usuario/animalia.db.backup

# O exportar a SQL
sqlite3 /home/usuario/animalia.db ".dump" > animalia_backup.sql
```

### Restaurar desde backup

```bash
# Desde copia
cp /home/usuario/animalia.db.backup /home/usuario/animalia.db

# Desde SQL
sqlite3 /home/usuario/animalia.db < animalia_backup.sql
```

---

## 🔐 Seguridad

### 1. Proteger archivo con contraseña

```bash
# SQLite soporta encriptación con SQLCipher
pnpm add sql.js
```

### 2. Restringir permisos

```bash
chmod 600 /home/usuario/animalia.db
chmod 700 /home/usuario/  # Directorio
```

### 3. No compartir archivo

⚠️ **Importante**: El archivo `.db` contiene toda la BD
- No lo subas a Git
- No lo compartas públicamente
- Haz backups regulares

---

## 🚨 Problemas Comunes

### "database is locked"

**Causa**: Múltiples procesos accediendo simultáneamente

**Solución**:
```bash
# Habilitar WAL
sqlite3 /home/usuario/animalia.db "PRAGMA journal_mode = WAL;"
```

### "no such table: users"

**Causa**: Migraciones no aplicadas

**Solución**:
```bash
pnpm exec drizzle-kit generate
pnpm exec drizzle-kit migrate
```

### "disk I/O error"

**Causa**: Permisos insuficientes

**Solución**:
```bash
chmod 666 /home/usuario/animalia.db
chmod 777 /home/usuario/
```

### "database disk image is malformed"

**Causa**: Archivo corrupto

**Solución**:
```bash
# Restaurar desde backup
cp /home/usuario/animalia.db.backup /home/usuario/animalia.db

# O recuperar datos
sqlite3 /home/usuario/animalia.db ".recover" > recovery.sql
```

---

## 📊 Estructura de Datos

### Tabla: users

```sql
CREATE TABLE users (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  openId TEXT UNIQUE NOT NULL,
  name TEXT,
  email TEXT,
  loginMethod TEXT,
  role TEXT DEFAULT 'user',
  createdAt DATETIME DEFAULT CURRENT_TIMESTAMP,
  updatedAt DATETIME DEFAULT CURRENT_TIMESTAMP,
  lastSignedIn DATETIME DEFAULT CURRENT_TIMESTAMP
);
```

### Tabla: animal_patterns

```sql
CREATE TABLE animal_patterns (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  userId INTEGER NOT NULL,
  animal_type TEXT NOT NULL,
  pattern_name TEXT,
  description TEXT,
  data JSON,
  createdAt DATETIME DEFAULT CURRENT_TIMESTAMP,
  updatedAt DATETIME DEFAULT CURRENT_TIMESTAMP,
  FOREIGN KEY (userId) REFERENCES users(id)
);
```

---

## 📞 Archivos Clave

| Archivo | Propósito |
|---------|-----------|
| `animalia.db` | Base de datos SQLite |
| `.env` | Configuración (ruta de BD) |
| `deploy-multi-db.sh` | Script de despliegue automático |
| `server/db-config.ts` | Configuración de BD |

---

## ✅ Checklist

- [ ] Archivo `.env` configurado con ruta SQLite
- [ ] Dependencias instaladas (`pnpm install`)
- [ ] Archivo `animalia.db` creado
- [ ] Migraciones aplicadas (`pnpm exec drizzle-kit migrate`)
- [ ] Servidor inicia correctamente (`pnpm run dev`)
- [ ] Puedes acceder a `http://localhost:3000`
- [ ] Backup del archivo `.db` creado

---

## 🎓 Cuándo Usar SQLite

| Caso | Recomendación |
|------|---------------|
| **Desarrollo local** | ✅ Ideal |
| **Pruebas** | ✅ Perfecto |
| **Aplicación pequeña** | ✅ Bueno |
| **Múltiples usuarios** | ❌ No recomendado |
| **Producción** | ❌ Usa MySQL/PostgreSQL |
| **Alta concurrencia** | ❌ Usa MySQL/PostgreSQL |

---

## 🔄 Migrar de SQLite a MySQL

Cuando necesites escalar:

```bash
# 1. Exportar desde SQLite
sqlite3 animalia.db ".dump" > animalia_export.sql

# 2. Crear BD MySQL
mysql -u root -p < setup_mysql.sql

# 3. Importar datos
mysql -u animalia_user -p animalia < animalia_export.sql

# 4. Actualizar .env
DATABASE_URL=mysql://animalia_user:password@localhost:3306/animalia
```

---

**¡Listo! Animalia está funcionando con SQLite.** ✅

