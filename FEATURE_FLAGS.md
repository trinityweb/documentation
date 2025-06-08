# Feature Flags - Implementación Multi-Tenant

## Descripción General

Esta documentación describe la implementación de feature flags para el sistema SaaS multi-tenant, diseñada para proporcionar verificación de características con **latencia cero** utilizando JWT y Kong API Gateway.

## Arquitectura

### Flujo de Datos

```
1. [Usuario hace login] → IAM Service
2. [IAM obtiene tenant features] → Database JSONB
3. [IAM genera JWT con features] → JWT Claims
4. [Cliente envía request] → Kong Gateway
5. [Kong extrae features de JWT] → Headers HTTP
6. [Servicios backend leen headers] → Lógica de negocio
```

### Componentes Involucrados

- **IAM Service**: Gestión de features y generación de JWT
- **Kong Gateway**: Extracción de features y headers HTTP
- **Servicios Backend**: Consumo de features via headers
- **PostgreSQL**: Almacenamiento persistente en JSONB

## Features Disponibles

### 1. Friends & Family (`friends_family`)
- **Descripción**: Funcionalidad para invitar familiares y amigos
- **Default**: `false`
- **Header Kong**: `X-Features-Friends-Family`

### 2. Premium Analytics (`premium_analytics`)
- **Descripción**: Analíticas avanzadas y reportes premium
- **Default**: `false`
- **Header Kong**: `X-Features-Premium-Analytics`

## Implementación por Servicio

### IAM Service

#### Base de Datos
```sql
-- Columna JSONB con índices GIN para búsquedas eficientes
ALTER TABLE tenants ADD COLUMN features JSONB DEFAULT '{"friends_family": false, "premium_analytics": false}';

-- Índices para consultas específicas
CREATE INDEX idx_tenants_features_friends_family ON tenants USING GIN ((features->'friends_family'));
CREATE INDEX idx_tenants_features_premium_analytics ON tenants USING GIN ((features->'premium_analytics'));
```

#### Value Object
```go
type TenantFeatures struct {
    FriendsFamily    bool `json:"friends_family"`
    PremiumAnalytics bool `json:"premium_analytics"`
}
```

#### JWT Claims
```go
type TokenClaims struct {
    UserID    uuid.UUID     `json:"user_id"`
    Email     string        `json:"email"`
    TenantID  uuid.UUID     `json:"tenant_id"`
    RoleID    uuid.UUID     `json:"role_id"`
    Features  *TenantFeatures `json:"features"`
    ExpiresAt int64         `json:"exp"`
}
```

#### API Endpoints

**Actualizar Features de Tenant**
```http
PATCH /api/v1/tenants/{id}/features
Content-Type: application/json

{
  "friends_family": true,
  "premium_analytics": false
}
```

**Response:**
```json
{
  "id": "tenant-uuid",
  "name": "Mi Empresa",
  "features": {
    "friends_family": true,
    "premium_analytics": false
  }
}
```

### Kong Gateway

#### Plugin Personalizado: `feature-flags`

**Archivo:** `api-gateway/plugins/feature-flags/handler.lua`

**Funcionalidad:**
- Extrae features del JWT automáticamente
- Convierte features a headers HTTP
- Maneja casos de JWT sin features
- Logging para debugging

**Headers Generados:**
- `X-Features-Friends-Family`: "true" | "false"
- `X-Features-Premium-Analytics`: "true" | "false"
- `X-Tenant-ID`: UUID del tenant
- `X-User-ID`: UUID del usuario

**Documentación detallada:** Ver `api-gateway/documentation/KONG_SETUP.md`

#### Configuración Kong

```yaml
plugins:
  - name: feature-flags
    service: pim-service
    config:
      enabled: true
      default_friends_family: false
      default_premium_analytics: false
```

### Servicios Backend (PIM, etc.)

#### Lectura de Features

**Middleware Go:**
```go
func FeatureFlagsMiddleware() gin.HandlerFunc {
    return func(c *gin.Context) {
        friendsFamily := c.GetHeader("X-Features-Friends-Family") == "true"
        premiumAnalytics := c.GetHeader("X-Features-Premium-Analytics") == "true"
        
        // Agregar al contexto
        c.Set("features.friends_family", friendsFamily)
        c.Set("features.premium_analytics", premiumAnalytics)
        
        c.Next()
    }
}
```

**Uso en Handlers:**
```go
func GetAnalytics(c *gin.Context) {
    premiumAnalytics, _ := c.Get("features.premium_analytics")
    
    if premiumAnalytics.(bool) {
        // Mostrar analíticas premium
        return getPremiumAnalytics()
    }
    
    // Mostrar analíticas básicas
    return getBasicAnalytics()
}
```

## Casos de Uso

### 1. Habilitar Feature para Tenant Específico

```bash
# Habilitar friends_family para tenant
curl -X PATCH http://localhost:8080/api/v1/tenants/{tenant-id}/features \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer {admin-jwt}" \
  -d '{
    "friends_family": true,
    "premium_analytics": false
  }'
```

### 2. Verificar Features en Runtime

```bash
# El JWT incluirá automáticamente las features
# Kong las convertirá a headers HTTP
curl -X GET http://localhost:8080/pim/api/v1/products \
  -H "Authorization: Bearer {user-jwt}"

# Los servicios backend recibirán:
# X-Features-Friends-Family: true
# X-Features-Premium-Analytics: false
```

### 3. Rollout Gradual de Features

```go
// Habilitar premium_analytics para tenants Enterprise
func EnablePremiumAnalyticsForEnterprise() {
    tenants := getTenantsByType("ENTERPRISE")
    for _, tenant := range tenants {
        tenant.EnablePremiumAnalytics()
        repository.Update(tenant)
    }
}
```

## Ventajas de esta Implementación

### 🚀 **Latencia Cero**
- Features embebidas en JWT
- No consultas a base de datos por request
- Kong procesa features directamente

### 🔧 **Stateless**
- JWT contiene toda la información necesaria
- Servicios backend sin dependencias externas
- Escalabilidad horizontal sin problemas

### 🎯 **Type-Safe**
- Value objects tipados en Go
- Validación en compile-time
- Menos errores en runtime

### 📊 **Observabilidad**
- Logging automático en Kong
- Headers HTTP rastreables
- Debugging simplificado

### 🔄 **Retrocompatibilidad**
- Valores por defecto para features nuevos
- JWTs antiguos siguen funcionando
- Migración gradual

## Monitoring y Debugging

### Logs de Kong
```bash
# Ver extracción de features
docker logs kong-gateway | grep "Features extracted"
```

### Headers de Debug
```bash
# Verificar headers recibidos por servicio backend
curl -v http://localhost:8080/pim/api/v1/products \
  -H "Authorization: Bearer {jwt}"
```

### Verificar JWT
```bash
# Decodificar JWT para ver features
echo "{jwt-token}" | base64 -d | jq .features
```

## Consideraciones de Seguridad

### 1. **Validación de JWT**
- Kong valida signature automáticamente
- Features no pueden ser manipuladas por cliente
- Expiración de tokens controlada

### 2. **Authorización por Features**
- Features != Permisos
- Combinar con sistema de roles
- Validación adicional en backend si es crítica

### 3. **Auditoría**
- Log de cambios de features
- Tracking de uso por tenant
- Métricas de adopción

## Próximos Pasos

### 1. **Nuevos Features**
- Agregar al value object `TenantFeatures`
- Actualizar migración de base de datos
- Configurar header en Kong plugin

### 2. **Dashboard de Features**
- Interface para administradores
- Habilitación masiva por tipo de tenant
- Métricas de uso

### 3. **A/B Testing**
- Features con porcentajes de rollout
- Cohorts de usuarios
- Métricas de conversión

## Troubleshooting

### JWT no contiene features
```
- Verificar que TenantService esté funcionando
- Comprobar que tenant tenga features en DB
- Revisar logs de IAM service
```

### Kong no envía headers
```
- Verificar plugin feature-flags está activo
- Comprobar configuración en kong.yml
- Revisar logs de Kong
```

### Backend no recibe features
```
- Verificar middleware de features está activo
- Comprobar que headers están llegando
- Revisar configuración de servicio
``` 