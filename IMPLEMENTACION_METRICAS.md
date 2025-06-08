# Implementación de Métricas por Servicio

Esta guía explica cómo implementar las métricas de Prometheus en cada servicio del proyecto SaaS.

## Servicios Go (IAM y PIM)

### 1. Dependencias Requeridas

Agregar al `go.mod`:
```bash
go get github.com/prometheus/client_golang/prometheus
go get github.com/prometheus/client_golang/prometheus/promhttp
```

### 2. Implementación en el Servicio IAM

El archivo `iam/src/api/monitoring/prometheus.go` ya está configurado. Para usarlo:

```go
package main

import (
    "iam/src/api/monitoring"
    "github.com/gin-gonic/gin"
)

func main() {
    // Iniciar servidor de métricas
    monitoring.StartPrometheusServer()
    
    // Configurar router con middleware
    r := gin.Default()
    r.Use(monitoring.PrometheusMiddleware())
    
    // Tus rutas aquí...
    
    r.Run(":8080")
}
```

### 3. Implementación en el Servicio PIM

Similar al IAM, usar el archivo `pim/src/api/monitoring/prometheus.go`:

```go
package main

import (
    "pim/src/api/monitoring"
    "github.com/gin-gonic/gin"
)

func main() {
    // Iniciar servidor de métricas
    monitoring.StartPrometheusServer()
    
    // Configurar router con middleware
    r := gin.Default()
    r.Use(monitoring.PrometheusMiddleware())
    
    // Tus rutas aquí...
    
    r.Run(":8080")
}
```

### 4. Registrar Métricas de Negocio

#### En el servicio IAM:
```go
import "iam/src/api/monitoring"

// Registrar autenticación exitosa
monitoring.RecordUserAuthentication("tenant-123", "success")

// Registrar consulta a base de datos
monitoring.RecordDatabaseQuery("SELECT", "users")

// Actualizar usuarios activos
monitoring.SetActiveUsers("tenant-123", 150)

// Registrar creación de tenant
monitoring.RecordTenantCreated("plan-456", "success")
```

#### En el servicio PIM:
```go
import "pim/src/api/monitoring"

// Registrar operación de producto
monitoring.RecordProductOperation("tenant-123", "create")

// Actualizar conteo de productos
monitoring.SetProductsCount("tenant-123", "electronics", 500)

// Registrar consulta a base de datos
monitoring.RecordDatabaseQuery("INSERT", "products")
```

## Servicio Python (Chat)

### 1. Dependencias Requeridas

Agregar al `pyproject.toml`:
```toml
[tool.poetry.dependencies]
prometheus-client = "^0.20.0"
```

Instalar:
```bash
poetry install
```

### 2. Configuración en FastAPI

En tu archivo principal (ej: `main.py`):

```python
from fastapi import FastAPI
from src.infrastructure.monitoring.prometheus_metrics import setup_metrics_endpoint

app = FastAPI()

# Configurar endpoint de métricas y middleware
setup_metrics_endpoint(app)

# Tus rutas aquí...
```

### 3. Registrar Métricas de Negocio

```python
from src.infrastructure.monitoring.prometheus_metrics import (
    record_whatsapp_message,
    set_active_conversations,
    record_message_processing_time,
    record_database_query
)

# Registrar mensaje de WhatsApp
record_whatsapp_message("tenant-123", "incoming", "processed")

# Actualizar conversaciones activas
set_active_conversations("tenant-123", 25)

# Registrar tiempo de procesamiento
record_message_processing_time("tenant-123", "text", 0.150)

# Registrar consulta a base de datos
record_database_query("INSERT", "messages")
```

## Variables de Entorno

Cada servicio debe configurar estas variables de entorno:

### Servicios Go:
```bash
PROMETHEUS_ENABLED=true
PROMETHEUS_PORT=2112  # 2113 para PIM
```

### Servicio Python:
```bash
PROMETHEUS_ENABLED=true
PROMETHEUS_PORT=8001
```

## Kong API Gateway

Kong ya está configurado con el plugin Prometheus en `api-gateway/kong.yml`. Las métricas se exponen automáticamente en `/metrics`.

## Métricas Disponibles por Servicio

### IAM Service (Puerto 2112/metrics)
- `http_requests_total{method, endpoint, status, tenant_id}`
- `http_request_duration_seconds{method, endpoint, tenant_id}`
- `database_connections_active`
- `database_queries_total{operation, table}`
- `user_authentications_total{tenant_id, status}`
- `active_users{tenant_id}`
- `tenants_created_total{plan_id, status}`

### PIM Service (Puerto 2113/metrics)
- `http_requests_total{method, endpoint, status, tenant_id}`
- `http_request_duration_seconds{method, endpoint, tenant_id}`
- `database_connections_active`
- `database_queries_total{operation, table}`
- `products_total{tenant_id, category}`
- `product_operations_total{tenant_id, operation}`

### Chat Service (Puerto 8002/metrics)
- `http_requests_total{method, endpoint, status, tenant_id}`
- `http_request_duration_seconds{method, endpoint, tenant_id}`
- `database_connections_active`
- `database_queries_total{operation, table}`
- `whatsapp_messages_total{tenant_id, direction, status}`
- `active_conversations{tenant_id}`
- `message_processing_duration_seconds{tenant_id, message_type}`

### Kong Gateway (Puerto 8000/metrics)
- `kong_http_requests_total`
- `kong_request_latency_ms`
- `kong_bandwidth_bytes`

## Logging Estructurado

### Formato JSON Recomendado

Todos los servicios deben usar este formato para logs:

```json
{
  "timestamp": "2023-10-01T12:00:00Z",
  "level": "INFO",
  "service": "iam-service",
  "tenant_id": "tenant-123",
  "user_id": "user-456",
  "request_id": "req-789",
  "message": "User authentication successful",
  "duration_ms": 150,
  "endpoint": "/api/v1/auth/login",
  "method": "POST",
  "status_code": 200
}
```

### Implementación en Go

```go
import (
    "encoding/json"
    "log"
    "time"
)

type LogEntry struct {
    Timestamp string `json:"timestamp"`
    Level     string `json:"level"`
    Service   string `json:"service"`
    TenantID  string `json:"tenant_id,omitempty"`
    UserID    string `json:"user_id,omitempty"`
    RequestID string `json:"request_id,omitempty"`
    Message   string `json:"message"`
    Duration  int64  `json:"duration_ms,omitempty"`
    Endpoint  string `json:"endpoint,omitempty"`
    Method    string `json:"method,omitempty"`
    Status    int    `json:"status_code,omitempty"`
}

func LogInfo(message string, fields map[string]interface{}) {
    entry := LogEntry{
        Timestamp: time.Now().UTC().Format(time.RFC3339),
        Level:     "INFO",
        Service:   "iam-service", // o "pim-service"
        Message:   message,
    }
    
    // Agregar campos adicionales
    if tenantID, ok := fields["tenant_id"].(string); ok {
        entry.TenantID = tenantID
    }
    // ... otros campos
    
    jsonData, _ := json.Marshal(entry)
    log.Println(string(jsonData))
}
```

### Implementación en Python

```python
import json
import logging
from datetime import datetime
from typing import Optional, Dict, Any

class StructuredLogger:
    def __init__(self, service_name: str):
        self.service_name = service_name
        self.logger = logging.getLogger(service_name)
        
    def log(self, level: str, message: str, **kwargs):
        entry = {
            "timestamp": datetime.utcnow().isoformat() + "Z",
            "level": level,
            "service": self.service_name,
            "message": message,
            **kwargs
        }
        
        self.logger.info(json.dumps(entry))

# Uso
logger = StructuredLogger("chat-service")
logger.log("INFO", "Message processed", 
          tenant_id="tenant-123", 
          duration_ms=150,
          message_type="text")
```

## Testing de Métricas

### Verificar que las métricas se exponen correctamente:

```bash
# IAM Service
curl http://localhost:2112/metrics

# PIM Service  
curl http://localhost:2113/metrics

# Chat Service
curl http://localhost:8002/metrics

# Kong Gateway
curl http://localhost:8000/metrics
```

### Verificar en Prometheus:

1. Acceder a http://localhost:9090
2. Ir a Status > Targets
3. Verificar que todos los servicios aparezcan como "UP"
4. Probar queries como: `up{job="iam-service"}`

## Troubleshooting

### Problema: Métricas no aparecen
1. Verificar variables de entorno `PROMETHEUS_ENABLED=true`
2. Verificar que el puerto de métricas esté expuesto
3. Revisar logs del servicio para errores

### Problema: Prometheus no puede scrape
1. Verificar conectividad de red entre contenedores
2. Revisar configuración en `prometheus.yml`
3. Verificar que el servicio esté en la red `saas-network`

### Problema: Logs no aparecen en Loki
1. Verificar que Promtail tenga acceso al socket de Docker
2. Revisar configuración de labels en `promtail-config.yml`
3. Verificar conectividad Promtail -> Loki 