# 🌐 Mapeo de Puertos - Servicios SaaS-MT

> **Documentación completa de puertos utilizados y disponibles en la arquitectura de microservicios**

## 📋 Índice
1. [Visión General](#visión-general)
2. [Servicios Activos](#servicios-activos)
3. [Puertos de Métricas](#puertos-de-métricas)
4. [Puertos Disponibles](#puertos-disponibles)
5. [Recomendaciones](#recomendaciones)
6. [Configuración Docker](#configuración-docker)

## 🎯 Visión General

Este documento mantiene el registro actualizado de todos los puertos utilizados en la arquitectura SaaS-MT, facilitando la planificación de nuevos servicios y evitando conflictos de puertos.

### 📊 Estado Actual
- **Servicios activos**: 7
- **Puertos principales ocupados**: 7
- **Puertos de métricas**: 4
- **Base de datos**: 1

## 🚀 Servicios Activos

### Tabla Principal de Servicios

| Servicio | Puerto Externo | Puerto Interno | Puerto Métricas | Tecnología | Estado |
|----------|----------------|----------------|-----------------|------------|--------|
| **PostgreSQL** | `5432` | `5432` | - | PostgreSQL 15 | ✅ Activo |
| **IAM Service** | `8080` | `8080` | `2112` | Go + Gin | ✅ Activo |
| **Chat Service** | `8000` | `8000` | `8002` | Python + FastAPI | ✅ Activo |
| **PIM Service** | `8090` | `8080` | `2113` | Go + Gin | ✅ Activo |
| **Stock Service** | `8100` | `8080` | `2114` | Go + Gin | ✅ Activo |
| **API Gateway** | `8001` | `8000` | - | Kong | ✅ Activo |
| **Kong Admin** | `8444` | `8001` | - | Kong Admin | ✅ Activo |
| **Backoffice** | `3000` | `3001` | - | Next.js | ✅ Activo |

### 🔍 Detalle por Servicio

#### 🔐 IAM Service
```yaml
ports:
  - "8080:8080"    # HTTP API
  - "2112:2112"    # Prometheus metrics
```
- **Responsabilidad**: Autenticación, autorización, gestión multi-tenant
- **Base de datos**: `iam_db`
- **Salud**: `/health`

#### 📦 PIM Service  
```yaml
ports:
  - "8090:8080"    # HTTP API (puerto externo diferente)
  - "2113:2113"    # Prometheus metrics
```
- **Responsabilidad**: Gestión de productos, categorías, inventario
- **Base de datos**: `pim_db`
- **Salud**: `/health`

#### 📊 Stock Service
```yaml
ports:
  - "8100:8080"    # HTTP API
  - "2114:2114"    # Prometheus metrics
```
- **Responsabilidad**: Gestión de inventario y stock
- **Base de datos**: `stock_db`
- **Salud**: `/health`

#### 💬 Chat Service
```yaml
ports:
  - "8000:8000"    # HTTP API
  - "8002:8001"    # Prometheus metrics
```
- **Responsabilidad**: Mensajería, WebSockets, chat con IA
- **Base de datos**: `chat_db`
- **Tecnología**: Python + FastAPI

#### 🌐 API Gateway (Kong)
```yaml
ports:
  - "8001:8000"    # Gateway principal
  - "8444:8001"    # Kong Admin API
```
- **Responsabilidad**: Enrutamiento, autenticación, rate limiting
- **Configuración**: `kong.yml`

#### 🖥️ Backoffice
```yaml
ports:
  - "3000:3001"    # Frontend Next.js
```
- **Responsabilidad**: Panel de administración
- **Tecnología**: Next.js

## 📊 Puertos de Métricas

### Prometheus Endpoints

| Servicio | Puerto | Endpoint | Descripción |
|----------|--------|----------|-------------|
| **IAM Service** | `2112` | `/metrics` | Métricas de autenticación |
| **PIM Service** | `2113` | `/metrics` | Métricas de productos |
| **Stock Service** | `2114` | `/metrics` | Métricas de inventario |
| **Chat Service** | `8002` | `/metrics` | Métricas de chat |

### 📈 Configuración Grafana
```yaml
# Scrape configs para prometheus.yml
- job_name: 'iam-service'
  static_configs:
    - targets: ['iam-service:2112']

- job_name: 'pim-service'
  static_configs:
    - targets: ['pim-service:2113']

- job_name: 'stock-service'
  static_configs:
    - targets: ['stock-service:2114']

- job_name: 'chat-service'
  static_configs:
    - targets: ['chat-service:8002']
```

## ✅ Puertos Disponibles

### 🔄 Para Servicios Go

| Puerto | Métricas | Recomendado Para | Estado |
|--------|----------|------------------|--------|
| **8110** | `2115` | Notification Service | 🟢 Disponible |
| **8120** | `2116` | Audit Service | 🟢 Disponible |
| **8130** | `2117` | Analytics Service | 🟢 Disponible |
| **8140** | `2118` | Billing Service | 🟢 Disponible |
| **8150** | `2119` | Integration Service | 🟢 Disponible |

### 🌐 Para Servicios Node.js/Frontend

| Puerto | Recomendado Para | Estado |
|--------|------------------|--------|
| **3002** | CRM Frontend | 🟢 Disponible |
| **3003** | Landing Page | 🟢 Disponible |
| **3004** | Admin Dashboard | 🟢 Disponible |
| **3005** | Mobile API | 🟢 Disponible |

### 🐍 Para Servicios Python

| Puerto | Métricas | Recomendado Para | Estado |
|--------|----------|------------------|--------|
| **8200** | `8201` | ML/AI Service | 🟢 Disponible |
| **8210** | `8211` | Analytics Engine | 🟢 Disponible |
| **8220** | `8221` | Report Generator | 🟢 Disponible |

## 💡 Recomendaciones

### 🏗️ Para Nuevos Servicios Go

**Template recomendado:**
```bash
# Ejemplo con MCP
Crea un servicio Go llamado "notification-service" en el puerto 8110 con los módulos ["notification", "template"]
```

**Configuración Docker:**
```yaml
notification-service:
  build:
    context: ./services/notification-service
    dockerfile: Dockerfile
  container_name: notification-service
  environment:
    DB_HOST: postgres
    DB_PORT: 5432
    DB_USER: postgres
    DB_PASSWORD: postgres
    DB_NAME: notification_db
    PROMETHEUS_ENABLED: "true"
    PROMETHEUS_PORT: 2115
  ports:
    - "8110:8080"      # HTTP API
    - "2115:2115"      # Prometheus metrics
  depends_on:
    postgres-setup:
      condition: service_completed_successfully
  networks:
    - saas-network
```

### 🔄 Patrones de Puertos

#### Servicios Go:
- **API Principal**: `8XXX` (8110, 8120, 8130...)
- **Puerto Interno**: Siempre `8080`
- **Métricas**: `21XX` (2115, 2116, 2117...)

#### Servicios Python:
- **API Principal**: `8X0X` (8200, 8210, 8220...)  
- **Métricas**: `8X1X` (8201, 8211, 8221...)

#### Servicios Node.js:
- **Frontend**: `30XX` (3002, 3003, 3004...)
- **API**: `9XXX` (9000, 9001, 9002...)

## 🔧 Configuración Docker

### Variables de Entorno Estándar

```bash
# Para servicios Go
DB_HOST=postgres
DB_PORT=5432
DB_USER=postgres
DB_PASSWORD=postgres
DB_NAME={service}_db
PROMETHEUS_ENABLED=true
PROMETHEUS_PORT={metrics_port}
GIN_MODE=debug

# Para servicios Python
DATABASE_URL=postgresql://postgres:postgres@postgres:5432/{service}_db
PROMETHEUS_ENABLED=true
PROMETHEUS_PORT={metrics_port}
LOG_LEVEL=INFO

# Para servicios Node.js
NODE_ENV=development
API_URL=http://api-gateway:8000
```

### 🌐 Actualización API Gateway

Para cada nuevo servicio, agregar a `kong.yml`:

```yaml
# Ejemplo para notification-service
services:
  - name: notification-service
    url: http://notification-service:8080
    routes:
      - name: notification-route
        paths:
          - /notification/api/v1
```

## 📋 Checklist para Nuevo Servicio

- [ ] 🔍 Verificar puerto disponible en esta documentación
- [ ] 🛠️ Crear servicio con MCP (si es Go)
- [ ] 🐳 Agregar configuración a `docker-compose.yml`
- [ ] 🌐 Actualizar `kong.yml` con nueva ruta
- [ ] 📊 Configurar métricas Prometheus
- [ ] 🗄️ Crear base de datos si es necesaria
- [ ] ✅ Probar conectividad y health checks
- [ ] 📝 Actualizar esta documentación

## 🔄 Mantenimiento

### Última Actualización
- **Fecha**: 2024-12-08
- **Versión**: 1.0.0
- **Servicios Documentados**: 8
- **Puertos Mapeados**: 11

### 📞 Contacto y Updates
- **Actualizar**: Cada vez que se agregue/modifique un servicio
- **Revisar**: Mensualmente para detectar puertos no utilizados
- **Validar**: Antes de deployments de producción

---

**💡 Tip**: Usa este documento como referencia antes de crear nuevos servicios para evitar conflictos de puertos y mantener consistencia en la arquitectura. 