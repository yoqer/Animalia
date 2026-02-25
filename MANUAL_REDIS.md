# 📘 Manual de Redis - Caché Distribuido Opcional

**Para Animalia Web Hosting - Optimización de Rendimiento**

---

## 🎯 ¿Qué es Redis?

**Redis** es un almacén de datos en memoria ultra-rápido que actúa como **caché distribuido**. Mejora el rendimiento almacenando datos frecuentes en memoria en lugar de consultarlos constantemente de la base de datos.

**Ventajas**:
- ✅ **10-100x más rápido** que consultas a BD
- ✅ **Reduce carga** en base de datos
- ✅ **Sincronización** entre múltiples servidores
- ✅ **Sesiones persistentes** entre instancias
- ✅ **Cola de tareas** para procesos asincronos

**Desventajas**:
- ❌ Datos en memoria (se pierden si se reinicia)
- ❌ Requiere infraestructura adicional
- ❌ Costo extra en hosting

---

## 🚀 Instalación Rápida

### Opción 1: Docker (Recomendado)

```bash
# Descargar imagen de Redis
docker pull redis:latest

# Ejecutar Redis en puerto 6379
docker run -d -p 6379:6379 --name animalia-redis redis:latest

# Verificar que está corriendo
docker ps | grep redis
```

### Opción 2: Linux/Mac (Homebrew)

```bash
# Instalar Redis
brew install redis

# Ejecutar
redis-server

# En otra terminal, verificar
redis-cli ping
# Respuesta: PONG
```

### Opción 3: Windows

```bash
# Descargar desde: https://github.com/microsoftarchive/redis/releases
# O usar WSL (Windows Subsystem for Linux)

# En WSL:
sudo apt-get install redis-server
redis-server
```

### Opción 4: Servicio en la Nube

```bash
# Redis Cloud (Gratuito hasta 30MB)
# https://redis.com/try-free/

# Upstash (Serverless Redis)
# https://upstash.com/

# AWS ElastiCache
# https://aws.amazon.com/elasticache/
```

---

## ⚙️ Configuración en Animalia

### Paso 1: Instalar Dependencia

```bash
cd /ruta/a/animalia_web_hosting

# Instalar cliente de Redis
pnpm add redis
```

### Paso 2: Configurar Variables de Entorno

```bash
# En archivo .env

# Redis local
REDIS_URL=redis://localhost:6379

# O Redis en la nube
REDIS_URL=redis://usuario:contraseña@host:puerto

# TTL por defecto (segundos)
REDIS_TTL=3600

# Habilitar Redis
REDIS_ENABLED=true
```

### Paso 3: Crear Archivo de Configuración

```typescript
// server/redis-config.ts

import { createClient } from 'redis';

export const redisClient = createClient({
  url: process.env.REDIS_URL || 'redis://localhost:6379',
});

redisClient.on('error', (err) => console.log('Redis error:', err));

export async function connectRedis() {
  if (!process.env.REDIS_ENABLED) return;
  
  try {
    await redisClient.connect();
    console.log('✅ Redis conectado');
  } catch (error) {
    console.error('❌ Error conectando a Redis:', error);
  }
}

export async function cacheGet(key: string) {
  if (!process.env.REDIS_ENABLED) return null;
  
  try {
    const value = await redisClient.get(key);
    return value ? JSON.parse(value) : null;
  } catch (error) {
    console.error('Error obteniendo del caché:', error);
    return null;
  }
}

export async function cacheSet(key: string, value: any, ttl?: number) {
  if (!process.env.REDIS_ENABLED) return;
  
  try {
    const ttlSeconds = ttl || parseInt(process.env.REDIS_TTL || '3600');
    await redisClient.setEx(key, ttlSeconds, JSON.stringify(value));
  } catch (error) {
    console.error('Error guardando en caché:', error);
  }
}

export async function cacheDel(key: string) {
  if (!process.env.REDIS_ENABLED) return;
  
  try {
    await redisClient.del(key);
  } catch (error) {
    console.error('Error eliminando del caché:', error);
  }
}
```

### Paso 4: Usar en Procedimientos tRPC

```typescript
// server/routers/sync.ts

import { cacheGet, cacheSet } from '../redis-config';

export const syncRouter = router({
  getPatterns: protectedProcedure
    .input(z.object({ since: z.date().optional() }))
    .query(async ({ ctx, input }) => {
      // Clave de caché
      const cacheKey = `patterns:${ctx.user.id}:${input.since?.getTime() || 0}`;

      // Intentar obtener del caché
      const cached = await cacheGet(cacheKey);
      if (cached) {
        console.log('✅ Datos desde caché');
        return cached;
      }

      // Si no está en caché, obtener de BD
      const patterns = await db.query.animalPatterns.findMany({
        where: input.since ? gt(animalPatterns.createdAt, input.since) : undefined,
      });

      // Guardar en caché por 1 hora
      await cacheSet(cacheKey, patterns, 3600);

      return patterns;
    }),
});
```

---

## 📊 Comandos Útiles de Redis

### Verificar Conexión

```bash
redis-cli ping
# Respuesta: PONG
```

### Ver Todas las Claves

```bash
redis-cli KEYS '*'
```

### Ver Tamaño del Caché

```bash
redis-cli INFO memory
```

### Limpiar Caché

```bash
# Eliminar una clave
redis-cli DEL clave

# Limpiar todo
redis-cli FLUSHALL
```

### Monitorear en Tiempo Real

```bash
redis-cli MONITOR
```

---

## 🎯 Casos de Uso en Animalia

### 1. Caché de Patrones de Comportamiento

```typescript
// Almacenar patrones frecuentes
const cacheKey = `patterns:${animalSpecies}:${timeRange}`;
const patterns = await cacheGet(cacheKey);

if (!patterns) {
  const newPatterns = await fetchPatternsFromDB();
  await cacheSet(cacheKey, newPatterns, 7200); // 2 horas
}
```

### 2. Sesiones de Usuario

```typescript
// Mantener sesiones activas entre servidores
const sessionKey = `session:${userId}`;
await cacheSet(sessionKey, userData, 86400); // 24 horas
```

### 3. Cola de Reentrenamiento

```typescript
// Encolar solicitudes de reentrenamiento
await redisClient.rPush('retraining-queue', JSON.stringify(request));

// Procesar desde cola
const job = await redisClient.lPop('retraining-queue');
```

### 4. Sincronización en Tiempo Real

```typescript
// Pub/Sub para sincronización
await redisClient.publish('sync-updates', JSON.stringify(data));

// Suscribirse
const subscriber = redisClient.duplicate();
await subscriber.subscribe('sync-updates', (message) => {
  console.log('Actualización recibida:', message);
});
```

---

## 📈 Monitoreo y Optimización

### Verificar Uso de Memoria

```bash
redis-cli INFO memory
```

**Salida esperada**:
```
used_memory: 1048576 (1MB)
used_memory_peak: 2097152 (2MB)
used_memory_rss: 3145728 (3MB)
```

### Configurar Límite de Memoria

```bash
# En redis.conf
maxmemory 512mb
maxmemory-policy allkeys-lru
```

### Monitorear Rendimiento

```bash
redis-cli --stat
```

---

## 🚨 Problemas Comunes

### "Connection refused"

**Causa**: Redis no está ejecutándose

**Solución**:
```bash
# Verificar si está corriendo
redis-cli ping

# Si no responde, iniciar
redis-server

# O con Docker
docker start animalia-redis
```

### "Out of memory"

**Causa**: Caché lleno

**Solución**:
```bash
# Aumentar límite en .env
REDIS_MAXMEMORY=1gb

# O limpiar caché
redis-cli FLUSHALL
```

### "Slow performance"

**Causa**: Demasiadas claves

**Solución**:
```bash
# Monitorear claves
redis-cli KEYS '*' | wc -l

# Establecer TTL más corto
REDIS_TTL=1800  # 30 minutos
```

---

## 💰 Costos

| Opción | Costo | Ventajas |
|--------|-------|----------|
| **Local** | $0 | Gratuito, control total |
| **Docker** | $0 | Fácil de usar |
| **Redis Cloud** | $0-99/mes | Administrado, escalable |
| **Upstash** | $0-199/mes | Serverless, sin servidor |
| **AWS ElastiCache** | $15-500+/mes | Integración AWS |

---

## ✅ Checklist

- [ ] Redis instalado y corriendo
- [ ] Variables de entorno configuradas
- [ ] Dependencia `redis` instalada
- [ ] Archivo de configuración creado
- [ ] Procedimientos tRPC usando caché
- [ ] TTL configurado apropiadamente
- [ ] Monitoreo en lugar
- [ ] Respaldo configurado

---

## 📞 Soporte

Si Redis no funciona:

1. Verificar conexión: `redis-cli ping`
2. Ver logs: `redis-cli MONITOR`
3. Revisar configuración en `.env`
4. Consultar documentación: https://redis.io/docs/

---

**¡Redis está configurado y listo para optimizar!** ✅

