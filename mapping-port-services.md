# 🌐 Mapeo de Puertos - Servicios SaaS-MT

> **Documentación completa de puertos utilizados y propuesta de normalización**

## 📋 Índice
1. [Estado Actual vs Documentado](#estado-actual-vs-documentado)
2. [Propuesta de Normalización](#propuesta-de-normalización)
3. [Servicios de Monitoreo](#servicios-de-monitoreo)
4. [Plan de Migración](#plan-de-migración)
5. [Configuración Docker](#configuración-docker)

## 🚨 Estado Actual vs Documentado

### **Inconsistencias Detectadas**

| Servicio | Documentado | Real | Estado | Acción Requerida |
|---------|-------------|------|--------|------------------|
| **Backoffice** | `3000:3001` | `3005` (nativo) | ❌ Inconsistente | Normalizar |
| **Marketplace Admin** | No documentado | `3004` (nativo) | ❌ Faltante | Documentar |
| **Marketplace Frontend** | No documentado | `3003` (nativo) | ❌ Faltante | Documentar |
| **Prometheus** | `9090:9090` | ❌ No corriendo | ❌ Inactivo | Activar |
| **Grafana** | `3002:3000` | ❌ No corriendo | ❌ Inactivo | Activar |
| **Servicios Métricas** | Configurados | ❌ No expuestos | ❌ Inactivos | Corregir |

## 🎯 Propuesta de Normalización

### **🏗️ Arquitectura de Puertos Estandarizada**

```
┌─────────────────────────────────────────────────────────┐
│                     FRONTEND TIER                       │
│  3000-3099: Aplicaciones de Usuario                    │
└─────────────────────────────────────────────────────────┘
┌─────────────────────────────────────────────────────────┐
│                     BACKEND TIER                        │
│  8000-8099: APIs y Microservicios                      │
│  8100-8199: Servicios de Negocio                       │
└─────────────────────────────────────────────────────────┘
┌─────────────────────────────────────────────────────────┐
│                   INFRASTRUCTURE TIER                   │
│  5000-5999: Bases de Datos                             │
│  9000-9999: Monitoreo y Observabilidad                 │
│  2000-2999: Métricas y Health Checks                   │
└─────────────────────────────────────────────────────────┘
```

## 📱 **FRONTEND TIER (3000-3099)**

### **Aplicaciones de Usuario**

| Servicio | Puerto Propuesto | Puerto Actual | Responsabilidad | Estado |
|----------|----------------|---------------|-----------------|--------|
| **Marketplace Frontend** | `3000` | `3003` | Tienda pública, catálogo | 🔄 Migrar |
| **Backoffice Admin** | `3001` | `3005` | Panel administración | 🔄 Migrar |
| **Marketplace Admin** | `3002` | `3004` | Gestión marketplace | 🔄 Migrar |
| **Customer Portal** | `3003` | - | Portal clientes | 🆕 Reservado |
| **Vendor Dashboard** | `3004` | - | Panel vendedores | 🆕 Reservado |

## 🔧 **BACKEND TIER (8000-8199)**

### **APIs y Microservicios (8000-8099)**

| Servicio | Puerto Externo | Puerto Interno | Métricas | Tecnología | Estado |
|----------|----------------|----------------|----------|------------|--------|
| **API Gateway** | `8001` | `8000` | - | Kong | ✅ Activo |
| **Chat Service** | `8000` | `8000` | `8002` | Python FastAPI | ✅ Activo |
| **Notification Service** | `8010` | `8080` | `2110` | Go + Gin | 🆕 Futuro |
| **Auth Service** | `8020` | `8080` | `2120` | Go + Gin | 🆕 Futuro |

### **Servicios de Negocio (8100-8199)**

| Servicio | Puerto Externo | Puerto Interno | Métricas | Tecnología | Estado |
|----------|----------------|----------------|----------|------------|--------|
| **IAM Service** | `8080` | `8080` | `2112` | Go + Gin | ✅ Activo |
| **PIM Service** | `8090` | `8080` | `2113` | Go + Gin | ✅ Activo |
| **Stock Service** | `8100` | `8080` | `2114` | Go + Gin | ✅ Activo |
| **Billing Service** | `8110` | `8080` | `2115` | Go + Gin | 🆕 Futuro |
| **Analytics Service** | `8120` | `8080` | `2116` | Go + Gin | 🆕 Futuro |

## 🗄️ **INFRASTRUCTURE TIER**

### **Bases de Datos (5000-5999)**

| Servicio | Puerto | Responsabilidad | Estado |
|----------|--------|-----------------|--------|
| **PostgreSQL** | `5432` | Base de datos principal | ✅ Activo |
| **MongoDB** | `27017` | Datos NoSQL, catálogos | ✅ Activo |
| **Redis** | `6379` | Cache, sesiones | 🆕 Futuro |
| **MinIO** | `9000` | Almacenamiento S3 | 🆕 Futuro |

### **Monitoreo y Observabilidad (9000-9999)**

| Servicio | Puerto | Responsabilidad | Estado |
|----------|--------|-----------------|--------|
| **Prometheus** | `9090` | Métricas y alertas | ❌ Configurar |
| **Grafana** | `9091` | Dashboards | ❌ Configurar |
| **Loki** | `3100` | Agregación de logs | ❌ Configurar |
| **Postgres Exporter** | `9187` | Métricas PostgreSQL | ❌ Configurar |
| **cAdvisor** | `8082` | Métricas contenedores | ❌ Configurar |
| **Kong Admin** | `8444` | Administración Kong | ✅ Activo |

### **Métricas y Health (2000-2999)**

| Servicio | Puerto | Endpoint | Estado |
|----------|--------|----------|--------|
| **IAM Metrics** | `2112` | `/metrics` | ⚠️ Config |
| **PIM Metrics** | `2113` | `/metrics` | ⚠️ Config |
| **Stock Metrics** | `2114` | `/metrics` | ⚠️ Config |
| **Chat Metrics** | `8002` | `/metrics` | ⚠️ Config |

## 🔄 Plan de Migración

### **Fase 1: Servicios de Monitoreo (Prioridad Alta)**

```bash
# Activar servicios de monitoreo
docker-compose -f docker-compose.infra.yml up -d prometheus grafana
```

### **Fase 2: Normalización de Frontends**

| Servicio | Acción | Comando Propuesto |
|----------|--------|-------------------|
| **Marketplace Frontend** | `3003 → 3000` | Actualizar package.json |
| **Backoffice** | `3005 → 3001` | Actualizar package.json |
| **Marketplace Admin** | `3004 → 3002` | Actualizar package.json |

### **Fase 3: Containerización de Frontends**

```yaml
# Agregar a docker-compose.yml
marketplace-frontend:
  build: ./services/saas-mt-marketplace-frontend
  ports:
    - "3000:3000"
  environment:
    - NEXT_PUBLIC_API_URL=http://localhost:8001
```

## ⚡ Acciones Inmediatas Recomendadas

### **1. ✅ Servicios de Monitoreo Activados**

```bash
# Ya ejecutado exitosamente
docker-compose -f docker-compose.infra.yml up -d prometheus grafana loki
```

**Servicios disponibles:**
- 🟢 **Prometheus**: http://localhost:9090
- 🟢 **Grafana**: http://localhost:3002 (admin/admin123)
- 🟢 **Loki**: http://localhost:3100

### **2. Corrección de Puertos Métricas**

Los puertos de métricas están configurados pero no se están exponiendo correctamente. Necesitamos actualizar el docker-compose.yml:

```yaml
# Corrección necesaria en docker-compose.yml
iam-service:
  ports:
    - "8080:8080"
    - "2112:2112"  # ⚠️ Exponer métricas

pim-service:
  ports:
    - "8090:8080"
    - "2113:2113"  # ⚠️ Exponer métricas

stock-service:
  ports:
    - "8100:8080"
    - "2114:2114"  # ⚠️ Exponer métricas
```

### **3. Normalización Frontends**

#### **Plan de Migración por Fases**

**Fase A: Actualizar Configuración Frontends**

```bash
# 1. Marketplace Frontend (3003 → 3000)
cd services/saas-mt-marketplace-frontend
# Actualizar package.json: "dev": "next dev -p 3000"

# 2. Backoffice (3005 → 3001)  
cd ../saas-mt-backoffice
# Actualizar package.json: "dev": "next dev -p 3001"

# 3. Marketplace Admin (3004 → 3002)
cd ../saas-mt-marketplace-admin
# Actualizar package.json: "dev": "next dev -p 3002"
```

**Fase B: Containerización Completa**

## 🔧 Configuración Docker Actualizada

### **docker-compose.yml - Sección Frontends Normalizados**

```yaml
# Frontends containerizados con puertos normalizados
marketplace-frontend:
  build:
    context: ./services/saas-mt-marketplace-frontend
    dockerfile: Dockerfile
  container_name: marketplace-frontend
  ports:
    - "3000:3000"  # Puerto normalizado
  environment:
    - NODE_ENV=development
    - NEXT_PUBLIC_API_URL=http://localhost:8001
    - API_GATEWAY_URL=http://api-gateway:8000
  depends_on:
    - api-gateway
  networks:
    - saas-network

backoffice:
  build:
    context: ./services/saas-mt-backoffice
    dockerfile: Dockerfile
  container_name: backoffice
  ports:
    - "3001:3001"  # Puerto normalizado (era 3005)
  environment:
    - NODE_ENV=development
    - NEXT_PUBLIC_API_URL=http://localhost:8001
    - API_GATEWAY_URL=http://api-gateway:8000
  depends_on:
    - api-gateway
  networks:
    - saas-network

marketplace-admin:
  build:
    context: ./services/saas-mt-marketplace-admin
    dockerfile: Dockerfile
  container_name: marketplace-admin
  ports:
    - "3002:3002"  # Puerto normalizado (era 3004)
  environment:
    - NODE_ENV=development
    - NEXT_PUBLIC_API_URL=http://api-gateway:8000
  depends_on:
    - api-gateway
  networks:
    - saas-network
```

### **docker-compose.infra.yml - Corrección Grafana**

```yaml
grafana:
  image: grafana/grafana:10.1.1
  container_name: saas-grafana
  ports:
    - "9091:3000"  # Cambio: 9091 en lugar de 3002
  environment:
    GF_SECURITY_ADMIN_USER: admin
    GF_SECURITY_ADMIN_PASSWORD: admin123
    # ... resto de configuración
```

## 📊 Estado Final Propuesto

### **Tabla Completa de Servicios Normalizados**

| Categoría | Servicio | Puerto | Estado | Tecnología |
|-----------|----------|--------|--------|------------|
| **FRONTEND** | Marketplace Frontend | `3000` | 🔄 Migrar | Next.js |
| **FRONTEND** | Backoffice Admin | `3001` | 🔄 Migrar | Next.js |
| **FRONTEND** | Marketplace Admin | `3002` | 🔄 Migrar | Next.js |
| **BACKEND** | Chat Service | `8000` | ✅ Activo | Python FastAPI |
| **BACKEND** | API Gateway | `8001` | ✅ Activo | Kong |
| **BACKEND** | IAM Service | `8080` | ✅ Activo | Go + Gin |
| **BACKEND** | PIM Service | `8090` | ✅ Activo | Go + Gin |
| **BACKEND** | Stock Service | `8100` | ✅ Activo | Go + Gin |
| **DATABASE** | PostgreSQL | `5432` | ✅ Activo | PostgreSQL 15 |
| **DATABASE** | MongoDB | `27017` | ✅ Activo | MongoDB 7.0 |
| **MONITORING** | Prometheus | `9090` | ✅ Activo | Prometheus |
| **MONITORING** | Grafana | `9091` | ✅ Activo | Grafana |
| **MONITORING** | Loki | `3100` | ✅ Activo | Loki |
| **ADMIN** | Kong Admin | `8444` | ✅ Activo | Kong Admin |

## 🎯 Próximos Pasos Recomendados

### **Paso 1: Verificar Servicios de Monitoreo**
```bash
# Verificar que estén funcionando
curl http://localhost:9090  # Prometheus
curl http://localhost:3002  # Grafana (temporal)
```

### **Paso 2: Corregir Exposición de Métricas**
- Actualizar docker-compose.yml para exponer puertos de métricas
- Reiniciar servicios backend

### **Paso 3: Migrar Frontends**
- Actualizar package.json de cada frontend
- Parar procesos nativos actuales
- Levantar versiones containerizadas

### **Paso 4: Documentar y Validar**
- Actualizar este documento con el estado final
- Crear scripts de validación
- Documentar URLs de acceso

## 📝 Scripts de Validación

### **health-check.sh**
```bash
#!/bin/bash
echo "🔍 Verificando servicios SaaS-MT..."

# Frontend
curl -f http://localhost:3000 > /dev/null && echo "✅ Marketplace Frontend OK" || echo "❌ Marketplace Frontend FAIL"
curl -f http://localhost:3001 > /dev/null && echo "✅ Backoffice OK" || echo "❌ Backoffice FAIL"  
curl -f http://localhost:3002 > /dev/null && echo "✅ Marketplace Admin OK" || echo "❌ Marketplace Admin FAIL"

# Backend
curl -f http://localhost:8001/health > /dev/null && echo "✅ API Gateway OK" || echo "❌ API Gateway FAIL"
curl -f http://localhost:8080/health > /dev/null && echo "✅ IAM Service OK" || echo "❌ IAM Service FAIL"

# Monitoring
curl -f http://localhost:9090 > /dev/null && echo "✅ Prometheus OK" || echo "❌ Prometheus FAIL"
curl -f http://localhost:9091 > /dev/null && echo "✅ Grafana OK" || echo "❌ Grafana FAIL"
```

## 📚 Referencias Rápidas

### **URLs de Acceso**
- **Marketplace**: http://localhost:3000
- **Backoffice**: http://localhost:3001  
- **Admin Panel**: http://localhost:3002
- **API Gateway**: http://localhost:8001
- **Prometheus**: http://localhost:9090
- **Grafana**: http://localhost:9091 (admin/admin123)
- **Kong Admin**: http://localhost:8444

### **Comandos Útiles**
```bash
# Ver puertos ocupados
lsof -i :3000-3099 | grep LISTEN

# Servicios Docker activos
docker ps --format "table {{.Names}}\t{{.Ports}}\t{{.Status}}"

# Logs de monitoreo
docker logs saas-prometheus
docker logs saas-grafana
```

---

**📅 Última Actualización**: 2024-12-08  
**🔄 Estado**: Servicios de monitoreo activados, frontends pendientes de normalización  
**👨‍💻 Siguiente**: Corregir exposición de métricas y migrar frontends
