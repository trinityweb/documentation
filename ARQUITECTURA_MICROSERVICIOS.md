# Arquitectura de Microservicios SaaS

## Índice
1. [Visión General](#visión-general)
2. [Arquitectura Final](#arquitectura-final)
3. [Servicios](#servicios)
4. [API Gateway](#api-gateway)
5. [Patrón Criteria](#patrón-criteria)
6. [Base de Datos](#base-de-datos)
7. [Comunicación entre Servicios](#comunicación-entre-servicios)
8. [Monitoreo y Observabilidad](#monitoreo-y-observabilidad)

## Visión General

Esta arquitectura implementa un sistema SaaS multi-tenant utilizando microservicios, con un API Gateway centralizado y patrones avanzados de búsqueda y paginación.

### Principios Arquitectónicos
- **Separación de responsabilidades**: Cada servicio tiene una responsabilidad específica
- **Independencia**: Los servicios pueden desarrollarse, desplegarse y escalarse independientemente
- **Consistencia**: Patrones comunes para autenticación, paginación y filtros
- **Observabilidad**: Monitoreo completo con métricas, logs y trazas

## Arquitectura Final

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│                                 FRONTEND LAYER                                  │
├─────────────────┬─────────────────┬─────────────────┬─────────────────────────┤
│   Backoffice    │      CRM        │   Landing Page  │    Mobile Apps          │
│   (Next.js)     │   (Next.js)     │   (Next.js)     │   (React Native)        │
│                 │                 │                 │                         │
│ - IAM Client    │ - IAM Client    │ - Marketing     │ - IAM Client            │
│ - PIM Client    │ - CRM Features  │ - Public APIs   │ - Core Features         │
│ - Admin Panel   │ - User Portal   │ - SEO           │ - Offline Support       │
└─────────────────┴─────────────────┴─────────────────┴─────────────────────────┘
                                        │
                                        ▼
┌─────────────────────────────────────────────────────────────────────────────────┐
│                              API GATEWAY LAYER                                  │
├─────────────────────────────────────────────────────────────────────────────────┤
│                            Kong API Gateway                                     │
│                                                                                 │
│ • Autenticación JWT           • Rate Limiting         • CORS                   │
│ • Enrutamiento                • Transformaciones      • Métricas               │
│ • Load Balancing              • Circuit Breaker       • Logs                   │
│ • SSL Termination             • Request/Response      • Health Checks          │
│                                 Validation                                      │
└─────────────────────────────────────────────────────────────────────────────────┘
                                        │
                    ┌───────────────────┼───────────────────┐
                    │                   │                   │
                    ▼                   ▼                   ▼
┌─────────────────────────────────────────────────────────────────────────────────┐
│                              MICROSERVICES LAYER                               │
├─────────────────┬─────────────────┬─────────────────┬─────────────────────────┤
│   IAM Service   │   PIM Service   │  Chat Service   │   Future Services       │
│   (Go + Gin)    │   (Go + Gin)    │  (Python)       │                         │
│                 │                 │                 │                         │
│ • Users         │ • Categories    │ • Messaging     │ • Notifications         │
│ • Tenants       │ • Products      │ • Real-time     │ • Analytics             │
│ • Roles         │ • Inventory     │ • WebSockets    │ • Reporting             │
│ • Plans         │ • Variants      │ • AI Chat       │ • Billing               │
│ • Auth/JWT      │ • Assets        │                 │ • Integrations          │
│                 │                 │                 │                         │
│ ✅ Criteria     │ ✅ Criteria     │ 🔄 In Progress  │ 📋 Planned              │
│ ✅ Pagination   │ ✅ Pagination   │                 │                         │
│ ✅ Filters      │ ✅ Filters      │                 │                         │
└─────────────────┴─────────────────┴─────────────────┴─────────────────────────┘
                                        │
                                        ▼
┌─────────────────────────────────────────────────────────────────────────────────┐
│                               PERSISTENCE LAYER                                │
├─────────────────┬─────────────────┬─────────────────┬─────────────────────────┤
│  PostgreSQL     │    Redis        │   File Storage  │   External APIs         │
│                 │                 │                 │                         │
│ • IAM Database  │ • Session Store │ • Images        │ • Payment Gateways      │
│ • PIM Database  │ • Cache Layer   │ • Documents     │ • Email Services        │
│ • Chat Database │ • Rate Limiting │ • Videos        │ • SMS Services          │
│                 │ • Pub/Sub       │ • Backups       │ • Cloud Storage         │
│                 │                 │                 │ • AI/ML Services        │
└─────────────────┴─────────────────┴─────────────────┴─────────────────────────┘
```

## Servicios

### IAM Service (Identity and Access Management)
**Puerto**: 8080  
**Base de datos**: PostgreSQL  
**Tecnología**: Go + Gin Framework  

**Responsabilidades**:
- Gestión de usuarios, roles y permisos
- Autenticación JWT y refresh tokens
- Gestión multi-tenant
- Gestión de planes y suscripciones

**Endpoints principales**:
```
GET    /api/v1/users               # Lista usuarios con criteria
GET    /api/v1/users/{id}          # Obtiene usuario por ID
POST   /api/v1/users               # Crea nuevo usuario
PUT    /api/v1/users/{id}          # Actualiza usuario
DELETE /api/v1/users/{id}          # Elimina usuario

GET    /api/v1/tenants             # Lista tenants
GET    /api/v1/roles               # Lista roles
GET    /api/v1/plans               # Lista planes

POST   /api/v1/auth/login          # Autenticación
POST   /api/v1/auth/refresh        # Renovar token
POST   /api/v1/auth/logout         # Cerrar sesión
```

### PIM Service (Product Information Management)
**Puerto**: 8080  
**Base de datos**: PostgreSQL  
**Tecnología**: Go + Gin Framework  

**Responsabilidades**:
- Gestión de categorías de productos
- Gestión de productos y variantes
- Gestión de inventario
- Assets y multimedia

**Endpoints principales**:
```
GET    /api/v1/categories          # Lista categorías con criteria
GET    /api/v1/categories/tree     # Árbol de categorías
GET    /api/v1/categories/{id}     # Obtiene categoría por ID
POST   /api/v1/categories          # Crea nueva categoría
PUT    /api/v1/categories/{id}     # Actualiza categoría
PATCH  /api/v1/categories/{id}/activate    # Activa categoría
PATCH  /api/v1/categories/{id}/deactivate  # Desactiva categoría
```

### Chat Service
**Puerto**: 8000  
**Base de datos**: PostgreSQL  
**Tecnología**: Python + FastAPI  

**Responsabilidades**:
- Mensajería en tiempo real
- WebSockets para comunicación
- Integración con IA para chat automatizado
- Notificaciones push

## API Gateway

### Kong Configuration
**Puerto externo**: 8001  
**Admin API**: 8444  

**Características**:
- **Enrutamiento**: Basado en prefijos de path
- **Autenticación**: JWT validation para servicios protegidos
- **Rate Limiting**: Límites por servicio y consumer
- **CORS**: Configuración global para todos los orígenes
- **Métricas**: Prometheus para monitoreo

**Configuración de rutas**:
```yaml
# IAM Service
/iam/api/v1/* → http://iam-service:8080/api/v1/*

# PIM Service  
/pim/api/v1/* → http://pim-service:8080/api/v1/*

# Frontend Apps
/backoffice/* → http://backoffice:3001/*
/crm/*        → http://frontend-activo-crm:3000/*
```

## Patrón Criteria

### Implementación
El patrón Criteria está implementado en todos los servicios para proporcionar:
- **Búsqueda avanzada**: Múltiples filtros combinables
- **Paginación**: Consistente en todos los endpoints
- **Ordenamiento**: Campos y direcciones configurables

### Estructura de Respuesta
```json
{
  "items": [...],
  "total_count": 100,
  "page": 1,
  "page_size": 10,
  "total_pages": 10
}
```

### Parámetros Comunes
```
page=1              # Número de página
page_size=10        # Tamaño de página (max 100)
sort_by=created_at  # Campo de ordenamiento
sort_dir=desc       # Dirección (asc/desc)
```

### Filtros por Servicio

**IAM - Usuarios**:
```
tenant_id=uuid      # Filtrar por tenant
status=ACTIVE       # Filtrar por estado
role_id=uuid        # Filtrar por rol
email=texto         # Búsqueda en email
provider=LOCAL      # Filtrar por proveedor
```

**PIM - Categorías**:
```
name=texto          # Búsqueda en nombre
description=texto   # Búsqueda en descripción
status=ACTIVE       # Filtrar por estado
parent_id=uuid      # Filtrar por padre
active=true         # Solo activas
root_only=true      # Solo categorías raíz
```

## Base de Datos

### Esquema Multi-Tenant
- **Tenant Isolation**: Cada tenant tiene sus propios datos
- **Shared Schema**: Esquema compartido con `tenant_id` en cada tabla
- **Cross-Tenant Data**: Algunos datos son globales (roles, planes)

### Migraciones
- **Evolutivas**: Migraciones que preservan datos existentes
- **Rollback**: Capacidad de revertir cambios
- **Validación**: Verificación de integridad antes y después

### Conexiones
- **Connection Pooling**: Gestión eficiente de conexiones
- **Read Replicas**: Para consultas de solo lectura (futuro)
- **Backup Strategy**: Backups automáticos y punto en el tiempo

## Comunicación entre Servicios

### Patrones de Comunicación
1. **Síncronos**: HTTP/REST para operaciones inmediatas
2. **Asíncronos**: Events para operaciones no críticas (futuro)
3. **Real-time**: WebSockets para chat y notificaciones

### Headers Obligatorios
```
Authorization: Bearer <jwt_token>    # Para autenticación
X-Tenant-ID: <tenant_uuid>          # Para multi-tenancy
Content-Type: application/json      # Para APIs REST
```

### Circuit Breaker
- **Timeout**: 30 segundos por defecto
- **Retry**: 3 intentos con backoff exponencial
- **Fallback**: Respuestas por defecto cuando sea posible

## Monitoreo y Observabilidad

### Stack de Monitoreo
- **Prometheus**: Recolección de métricas
- **Grafana**: Visualización y dashboards
- **Loki**: Agregación de logs
- **Promtail**: Envío de logs
- **cAdvisor**: Métricas de contenedores

### Métricas Clave
- **Latencia**: P50, P95, P99 por endpoint
- **Throughput**: Requests por segundo
- **Error Rate**: Porcentaje de errores 4xx/5xx
- **Availability**: Uptime de servicios

### Health Checks
Cada servicio expone:
```
GET /health         # Estado básico del servicio
GET /api/v1/health  # Estado con información detallada
```

### Logs Estructurados
Formato JSON con campos estándar:
```json
{
  "timestamp": "2025-01-01T00:00:00Z",
  "level": "INFO",
  "service": "iam-service",
  "message": "User created successfully",
  "user_id": "uuid",
  "tenant_id": "uuid",
  "trace_id": "uuid"
}
```

## Próximos Pasos

### Desarrollo Inmediato
1. **Chat Service**: Completar implementación de criteria
2. **Notification Service**: Nuevo microservicio
3. **WebSockets**: Real-time para chat y notificaciones

### Desarrollo Medio Plazo
1. **Analytics Service**: Métricas de negocio
2. **Billing Service**: Facturación y pagos
3. **Integration Service**: APIs externas

### Desarrollo Largo Plazo
1. **Event-Driven Architecture**: Eventos asíncronos
2. **Service Mesh**: Istio para comunicación avanzada
3. **Multi-Region**: Despliegue en múltiples regiones

---

**Última actualización**: Enero 2025  
**Versión**: 1.0  
**Mantenido por**: Equipo de Arquitectura 