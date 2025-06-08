# Sistema de Monitoreo y Observabilidad

Este documento describe la implementación del stack de monitoreo y observabilidad para el proyecto SaaS multi-tenant.

## Componentes del Stack

### 1. Prometheus
- **Puerto**: 9090
- **Función**: Recolección y almacenamiento de métricas
- **Retención**: 30 días
- **Configuración**: `monitoring/prometheus/prometheus.yml`

### 2. Grafana
- **Puerto**: 3002
- **Usuario**: admin
- **Password**: admin123
- **Base de datos**: PostgreSQL (grafana_db en servidor compartido)
- **Configuración**: `monitoring/grafana/`

### 3. Loki
- **Puerto**: 3100
- **Función**: Almacenamiento y consulta de logs
- **Configuración**: `monitoring/loki/loki-config.yml`

### 4. Promtail
- **Puerto**: 9080
- **Función**: Recolección de logs de contenedores Docker
- **Configuración**: `monitoring/promtail/promtail-config.yml`

### 5. Exporters Adicionales
- **PostgreSQL Exporter**: Puerto 9187
- **cAdvisor**: Puerto 8080 (métricas de contenedores)

## Configuración de Servicios

### Servicios Go (IAM y PIM)
Para integrar métricas de Prometheus en servicios Go, agregar:

```go
import (
    "github.com/prometheus/client_golang/prometheus"
    "github.com/prometheus/client_golang/prometheus/promhttp"
)

// Variables de entorno necesarias:
// PROMETHEUS_ENABLED=true
// PROMETHEUS_PORT=2112 (o 2113 para PIM)

// Métricas comunes a implementar:
var (
    httpRequestsTotal = prometheus.NewCounterVec(
        prometheus.CounterOpts{
            Name: "http_requests_total",
            Help: "Total number of HTTP requests",
        },
        []string{"method", "endpoint", "status"},
    )
    
    httpDuration = prometheus.NewHistogramVec(
        prometheus.HistogramOpts{
            Name: "http_request_duration_seconds",
            Help: "Duration of HTTP requests",
        },
        []string{"method", "endpoint"},
    )
)

// Endpoint de métricas:
http.Handle("/metrics", promhttp.Handler())
```

### Servicio Python (Chat)
Para integrar métricas en FastAPI:

```python
from prometheus_client import Counter, Histogram, generate_latest
from fastapi import FastAPI
from fastapi.responses import Response

# Variables de entorno:
# PROMETHEUS_ENABLED=true
# PROMETHEUS_PORT=8001

# Métricas comunes:
REQUEST_COUNT = Counter('http_requests_total', 'Total HTTP requests', ['method', 'endpoint', 'status'])
REQUEST_DURATION = Histogram('http_request_duration_seconds', 'HTTP request duration')

@app.get("/metrics")
async def metrics():
    return Response(generate_latest(), media_type="text/plain")
```

### Kong API Gateway
Kong ya está configurado con el plugin Prometheus que exporta métricas en:
- **Endpoint**: `/metrics` en el puerto 8000
- **Métricas incluidas**: latencia, bandwidth, status codes, por consumer

## Dashboards Disponibles

### Dashboard Principal: SaaS Services Overview
- **UID**: saas-overview
- **Ubicación**: `monitoring/grafana/dashboards/saas-services-overview.json`
- **Métricas mostradas**:
  - Estado de servicios (UP/DOWN)
  - Rate de requests HTTP
  - Latencia de respuesta

### Dashboards de Datos PostgreSQL
Para más detalles sobre estos dashboards, ver [DASHBOARDS_GRAFANA.md](./DASHBOARDS_GRAFANA.md)

1. **IAM - Dashboard de Usuarios y Tenants**
   - **UID**: iam-dashboard
   - **Contenido**: Estadísticas de usuarios, registros, distribución por tenant y roles

2. **Chat - Dashboard de Comunicaciones**
   - **UID**: chat-dashboard
   - **Contenido**: Métricas de mensajes, conversaciones, estados de entrega

3. **PIM - Dashboard de Productos**
   - **UID**: pim-dashboard
   - **Contenido**: Estadísticas de productos, distribución por categoría y marca

4. **Dashboard Operativo Integrado**
   - **UID**: saas-operativo
   - **Contenido**: Visión 360° de todos los servicios en una sola vista

### Dashboards Adicionales Recomendados
1. **Infrastructure Overview**: CPU, memoria, disco de contenedores
2. **Database Monitoring**: Conexiones PostgreSQL, queries lentas

## Alertas Configuradas

### Alertas Críticas
- **ServiceDown**: Servicio no disponible por más de 1 minuto
- **DatabaseConnectionFailure**: PostgreSQL no accesible

### Alertas de Warning
- **HighErrorRate**: Tasa de error > 10% por 5 minutos
- **HighResponseTime**: P95 latencia > 1 segundo por 5 minutos
- **HighMemoryUsage**: Uso de memoria > 80% por 5 minutos
- **HighCPUUsage**: Uso de CPU > 80% por 5 minutos

## URLs de Acceso

| Servicio | URL | Credenciales |
|----------|-----|--------------|
| Prometheus | http://localhost:9090 | N/A |
| Grafana | http://localhost:3002 | admin/admin123 |
| cAdvisor | http://localhost:8080 | N/A |

## Métricas por Servicio

### Kong API Gateway (Puerto 8000/metrics)
- `kong_http_requests_total`
- `kong_request_latency_ms`
- `kong_bandwidth_bytes`

### IAM Service (Puerto 2112/metrics)
- `http_requests_total`
- `http_request_duration_seconds`
- `database_connections_active`
- `user_authentications_total`
- `active_users`
- `tenants_created_total`

### PIM Service (Puerto 2113/metrics)  
- `http_requests_total`
- `http_request_duration_seconds`
- `database_connections_active`

### Chat Service (Puerto 8002/metrics)
- `http_requests_total`
- `http_request_duration_seconds`
- `whatsapp_messages_total`

### PostgreSQL (Puerto 9187/metrics)
- `pg_up`
- `pg_database_size_bytes`
- `pg_stat_database_tup_returned`

## Logs Estructurados

### Configuración de Logging por Servicio

Todos los servicios deben generar logs en formato JSON para facilitar el parsing en Loki:

```json
{
  "timestamp": "2023-10-01T12:00:00Z",
  "level": "INFO",
  "service": "iam-service",
  "tenant_id": "tenant-123",
  "user_id": "user-456", 
  "message": "User authentication successful",
  "request_id": "req-789",
  "duration_ms": 150
}
```

### Niveles de Log Recomendados
- **ERROR**: Errores que requieren atención inmediata
- **WARN**: Situaciones anómalas pero no críticas
- **INFO**: Eventos importantes del sistema
- **DEBUG**: Información detallada para desarrollo

## Deployment y Maintenance

### Iniciar el Stack Completo
```bash
docker-compose up -d prometheus grafana loki promtail postgres-exporter cadvisor
```

### Verificar Estado de Servicios
```bash
# Verificar targets de Prometheus
curl http://localhost:9090/api/v1/targets

# Verificar salud de Grafana
curl http://localhost:3002/api/health

# Verificar Loki
curl http://localhost:3100/ready
```

### Backup de Configuraciones
- Dashboards de Grafana se versionan en: `monitoring/grafana/dashboards/`
- Configuraciones de alertas: `monitoring/prometheus/rules/`
- Configuración de datasources: `monitoring/grafana/provisioning/`

### Escalabilidad
Para entornos de producción considerar:
- Prometheus en modo HA con Thanos
- Grafana con múltiples instancias + balanceador
- Loki distribuido
- AlertManager para notificaciones

## Troubleshooting

### Problema: Métricas no aparecen en Prometheus
1. Verificar que el servicio expone `/metrics` en el puerto correcto
2. Comprobar configuración en `prometheus.yml`
3. Revisar logs de Prometheus: `docker logs saas-prometheus`

### Problema: Dashboards no cargan en Grafana
1. Verificar conexión a datasource Prometheus
2. Revisar permisos de archivos en `monitoring/grafana/`
3. Comprobar logs: `docker logs saas-grafana`

### Problema: Logs no aparecen en Loki
1. Verificar que Promtail puede acceder a Docker socket
2. Revisar configuración de labels en `promtail-config.yml`
3. Comprobar conectividad Promtail -> Loki

## Métricas de Negocio Recomendadas

### Multi-tenancy
- Requests por tenant
- Uso de features por tenant
- Límites de rate limiting por tenant

### Performance
- Tiempo de respuesta por endpoint
- Throughput de la aplicación
- Errores por servicio y endpoint

### Recursos
- Uso de base de datos por tenant
- Almacenamiento utilizado 