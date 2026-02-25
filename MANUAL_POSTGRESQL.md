# 📘 Manual de Configuración - PostgreSQL

**Para Animalia Web Hosting con PostgreSQL (Producción Robusta)**

---

## 🎯 ¿Por qué PostgreSQL?

- ✅ **Robusto**: Ideal para producción
- ✅ **Potente**: Características avanzadas
- ✅ **Escalable**: Soporta millones de registros
- ✅ **Confiable**: ACID garantizado

**Ideal para**: Producción, aplicaciones críticas, datos complejos

---

## 🚀 Despliegue Automático (Recomendado)

### Paso 1: Ejecutar script de despliegue

```bash
cd /ruta/a/animalia_web_hosting
./deploy-multi-db.sh postgresql producción
```

**¿Qué hace?**
- ✅ Instala dependencias de PostgreSQL
- ✅ Crea base de datos `animalia`
- ✅ Crea usuario `animalia_user`
- ✅ Aplica migraciones
- ✅ Configura índices
- ✅ Inicia el servidor

---

## 🔧 Despliegue Manual

### Opción A: PostgreSQL Local

#### Paso 1: Instalar PostgreSQL

**Windows**:
```bash
# Descargar desde: https://www.postgresql.org/download/windows/
# Ejecutar instalador
# Recordar contraseña de postgres
```

**Mac**:
```bash
brew install postgresql@15
brew services start postgresql@15
```

**Linux**:
```bash
sudo apt-get install postgresql postgresql-contrib
sudo systemctl start postgresql
```

#### Paso 2: Crear archivo .env

```bash
cat > .env << EOF
DATABASE_URL=postgresql://animalia_user:contraseña@localhost:5432/animalia
DB_TYPE=postgresql
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

#### Paso 3: Crear base de datos y usuario

```bash
# Conectar como postgres
sudo -u postgres psql

# Dentro de psql:
CREATE DATABASE animalia;
CREATE USER animalia_user WITH PASSWORD 'contraseña_segura';
GRANT ALL PRIVILEGES ON DATABASE animalia TO animalia_user;
\q
```

#### Paso 4: Instalar dependencias

```bash
pnpm install
pnpm add postgres drizzle-orm
```

#### Paso 5: Aplicar migraciones

```bash
pnpm exec drizzle-kit generate
pnpm exec drizzle-kit migrate
```

#### Paso 6: Ejecutar

```bash
pnpm run dev
```

---

### Opción B: PostgreSQL en Hosting Remoto

Si tu hosting proporciona PostgreSQL:

#### Paso 1: Obtener credenciales

El hosting te proporciona:
- **Host**: `postgres.hosting.com`
- **Puerto**: `5432`
- **Usuario**: `tu_usuario`
- **Contraseña**: Tu contraseña
- **Base de datos**: `tu_base`

#### Paso 2: Actualizar .env

```
DATABASE_URL=postgresql://tu_usuario:tu_contraseña@postgres.hosting.com:5432/tu_base
```

#### Paso 3: Verificar conexión

```bash
psql postgresql://tu_usuario:tu_contraseña@postgres.hosting.com:5432/tu_base -c "SELECT 1;"
```

---

## 📊 Verificar Instalación

### Conectar a PostgreSQL

```bash
# Local
psql -U animalia_user -d animalia

# Remoto
psql postgresql://animalia_user:contraseña@localhost:5432/animalia
```

### Ver tablas

```sql
\dt

-- Deberías ver:
-- animal_patterns
-- conversations
-- knowledge
-- retraining_requests
-- sync_history
-- users
```

### Ver estructura de tabla

```sql
\d users
\d animal_patterns
```

### Ver datos

```sql
SELECT * FROM users;
SELECT COUNT(*) FROM animal_patterns;
```

---

## 📈 Optimización para Producción

### 1. Crear índices

```sql
-- Índices para búsquedas rápidas
CREATE INDEX idx_user_openid ON users(openId);
CREATE INDEX idx_user_email ON users(email);
CREATE INDEX idx_pattern_animal ON animal_patterns(animal_type);
CREATE INDEX idx_pattern_created ON animal_patterns(createdAt DESC);
CREATE INDEX idx_sync_created ON sync_history(createdAt DESC);
CREATE INDEX idx_sync_user ON sync_history(userId);
```

### 2. Configurar autovacuum

```sql
-- Optimizar automáticamente
ALTER TABLE users SET (autovacuum_vacuum_scale_factor = 0.01);
ALTER TABLE animal_patterns SET (autovacuum_vacuum_scale_factor = 0.01);
```

### 3. Aumentar conexiones

En `/etc/postgresql/15/main/postgresql.conf`:
```
max_connections = 200
shared_buffers = 256MB
effective_cache_size = 1GB
```

### 4. Configurar backups

```bash
# Backup diario
0 2 * * * pg_dump -U animalia_user animalia > /backups/animalia_$(date +\%Y\%m\%d).sql
```

---

## 🔐 Seguridad

### 1. Crear usuario con permisos limitados

```sql
CREATE USER animalia_user WITH PASSWORD 'contraseña_segura';
GRANT CONNECT ON DATABASE animalia TO animalia_user;
GRANT USAGE ON SCHEMA public TO animalia_user;
GRANT CREATE ON SCHEMA public TO animalia_user;
GRANT ALL PRIVILEGES ON ALL TABLES IN SCHEMA public TO animalia_user;
GRANT ALL PRIVILEGES ON ALL SEQUENCES IN SCHEMA public TO animalia_user;
```

### 2. Configurar SSL/TLS

En `/etc/postgresql/15/main/postgresql.conf`:
```
ssl = on
ssl_cert_file = '/etc/postgresql/server.crt'
ssl_key_file = '/etc/postgresql/server.key'
```

### 3. Configurar pg_hba.conf

```
# Permitir conexiones locales
local   animalia   animalia_user                    md5

# Permitir conexiones remotas (con SSL)
hostssl animalia   animalia_user   0.0.0.0/0       md5
```

---

## 📊 Estructura de Datos

### Tabla: users

```sql
CREATE TABLE users (
  id SERIAL PRIMARY KEY,
  openId VARCHAR(64) UNIQUE NOT NULL,
  name TEXT,
  email VARCHAR(320),
  loginMethod VARCHAR(64),
  role VARCHAR(20) DEFAULT 'user',
  createdAt TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updatedAt TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  lastSignedIn TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

### Tabla: animal_patterns

```sql
CREATE TABLE animal_patterns (
  id SERIAL PRIMARY KEY,
  userId INTEGER NOT NULL REFERENCES users(id),
  animal_type VARCHAR(255) NOT NULL,
  pattern_name VARCHAR(255),
  description TEXT,
  data JSONB,
  createdAt TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updatedAt TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

---

## 🚨 Problemas Comunes

### "FATAL: Ident authentication failed"

**Causa**: Método de autenticación incorrecto

**Solución**:
```bash
# Editar pg_hba.conf
sudo nano /etc/postgresql/15/main/pg_hba.conf

# Cambiar "ident" a "md5"
# local   all             all                                     md5

sudo systemctl restart postgresql
```

### "FATAL: remaining connection slots are reserved"

**Causa**: Demasiadas conexiones

**Solución**:
```sql
-- Ver conexiones activas
SELECT datname, count(*) FROM pg_stat_activity GROUP BY datname;

-- Aumentar límite en postgresql.conf
max_connections = 300
```

### "permission denied for schema public"

**Causa**: Permisos insuficientes

**Solución**:
```sql
GRANT ALL PRIVILEGES ON SCHEMA public TO animalia_user;
GRANT ALL PRIVILEGES ON ALL TABLES IN SCHEMA public TO animalia_user;
```

---

## 💾 Backup y Restauración

### Backup completo

```bash
# Backup con formato SQL
pg_dump -U animalia_user animalia > animalia_backup.sql

# Backup con formato custom (más pequeño)
pg_dump -U animalia_user -Fc animalia > animalia_backup.dump
```

### Restaurar

```bash
# Desde SQL
psql -U animalia_user animalia < animalia_backup.sql

# Desde custom
pg_restore -U animalia_user -d animalia animalia_backup.dump
```

### Backup incremental

```bash
# Usar WAL (Write-Ahead Logging)
wal_level = replica
archive_mode = on
archive_command = 'cp %p /backups/wal_archive/%f'
```

---

## 📞 Archivos Clave

| Archivo | Propósito |
|---------|-----------|
| `.env` | Configuración (URL de PostgreSQL) |
| `deploy-multi-db.sh` | Script de despliegue automático |
| `server/db-config.ts` | Configuración de BD |
| `postgresql.conf` | Configuración de PostgreSQL |
| `pg_hba.conf` | Configuración de autenticación |

---

## ✅ Checklist

- [ ] PostgreSQL instalado y ejecutándose
- [ ] Base de datos `animalia` creada
- [ ] Usuario `animalia_user` creado
- [ ] Archivo `.env` configurado
- [ ] Dependencias instaladas (`pnpm install`)
- [ ] Migraciones aplicadas (`pnpm exec drizzle-kit migrate`)
- [ ] Índices creados
- [ ] Backups configurados
- [ ] Servidor inicia correctamente (`pnpm run dev`)
- [ ] Puedes acceder a `http://localhost:3000`

---

## 🎓 Comparativa: PostgreSQL vs MySQL

| Aspecto | PostgreSQL | MySQL |
|--------|-----------|-------|
| **Transacciones** | ACID completo | ACID (InnoDB) |
| **Características** | Avanzadas | Básicas |
| **JSON** | JSONB nativo | JSON simple |
| **Escalabilidad** | Excelente | Buena |
| **Complejidad** | Media | Baja |
| **Ideal para** | Producción crítica | Aplicaciones web |

---

**¡Listo! Animalia está funcionando con PostgreSQL.** ✅

