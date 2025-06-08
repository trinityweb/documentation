# Dashboards de Grafana para Monitoreo SaaS

Este documento describe los dashboards de Grafana implementados para monitorear nuestra plataforma SaaS multi-tenant, incluyendo los servicios de IAM, Chat, PIM y Stock, todos operando sobre PostgreSQL.

## Acceso a Grafana

- **URL**: http://localhost:3002
- **Usuario**: admin
- **Contraseña**: admin123
- **Carpeta de Dashboards**: SaaS Monitoring

## Fuentes de Datos Configuradas

Se han configurado las siguientes fuentes de datos PostgreSQL para visualizar información directamente desde nuestras bases de datos:

| Nombre | Base de datos | Descripción |
|--------|--------------|-------------|
| PostgreSQL | postgres | Base de datos principal/sistema |
| IAM Database | iam_db | Gestión de identidad y acceso |
| Chat Database | chat_db | Sistema de mensajería |
| PIM Database | pim_db | Gestión de información de productos |
| Stock Database | stock_db | Gestión de inventario |

Además, se configuraron:
- **Prometheus**: Para métricas de rendimiento
- **Loki**: Para visualización de logs

## Dashboards Disponibles

### 1. IAM - Dashboard de Usuarios y Tenants
**UID**: `iam-dashboard`

Este dashboard proporciona una visión completa de la gestión de usuarios:

- **Estadísticas de Usuarios**: Total, activos, verificados y activos en los últimos 7 días
- **Nuevos Usuarios por Día**: Gráfico temporal de registros de usuarios
- **Distribución por Tenant**: Gráfico circular que muestra usuarios por tenant
- **Distribución de Roles**: Tabla detallada de roles por tenant

**Consultas clave:**
```sql
-- Estadísticas de usuarios
SELECT 
  COUNT(*) as total_users,
  COUNT(*) FILTER (WHERE is_active = true) as active_users,
  COUNT(*) FILTER (WHERE is_verified = true) as verified_users,
  COUNT(*) FILTER (WHERE last_login > now() - interval '7 days') as users_active_last_7_days
FROM users;

-- Nuevos usuarios por día
SELECT 
  date_trunc('day', created_at) as time,
  COUNT(*) as new_users
FROM users
GROUP BY date_trunc('day', created_at)
ORDER BY time;
```

### 2. Chat - Dashboard de Comunicaciones
**UID**: `chat-dashboard`

Este dashboard monitorea el sistema de mensajería:

- **Estadísticas de Mensajes**: Total mensajes, conversaciones y mensajes en las últimas 24h
- **Volumen de Mensajes por Día**: Tendencias temporales de mensajes
- **Distribución por Dirección**: Mensajes entrantes vs salientes
- **Estado de Entrega**: Métricas de estado de entrega de mensajes
- **Últimas Conversaciones**: Tabla con detalles de las conversaciones más recientes

**Consultas clave:**
```sql
-- Estadísticas de mensajes
SELECT 
  COUNT(*) as total_messages,
  COUNT(DISTINCT conversation_id) as total_conversations,
  COUNT(*) FILTER (WHERE sent_at > now() - interval '24 hours') as messages_last_24h
FROM messages;

-- Volumen de mensajes diario
SELECT 
  date_trunc('day', sent_at) as time,
  COUNT(*) as message_count
FROM messages
WHERE sent_at IS NOT NULL
GROUP BY date_trunc('day', sent_at)
ORDER BY time;
```

### 3. PIM - Dashboard de Productos
**UID**: `pim-dashboard`

Este dashboard visualiza la gestión de productos:

- **Estadísticas de Productos**: Total, activos, inactivos y nuevos en los últimos 30 días
- **Nuevos Productos por Mes**: Tendencia temporal de creación de productos
- **Productos por Categoría**: Top 10 categorías con más productos
- **Productos por Marca**: Top 10 marcas con más productos
- **Últimos Productos Añadidos**: Tabla con detalles de productos recientemente creados

**Consultas clave:**
```sql
-- Estadísticas de productos
SELECT 
  COUNT(*) as total_products,
  COUNT(*) FILTER (WHERE status = 'active') as active_products,
  COUNT(*) FILTER (WHERE status = 'inactive') as inactive_products,
  COUNT(*) FILTER (WHERE created_at > now() - interval '30 days') as new_products_30d
FROM products;

-- Distribución por categoría
SELECT 
  category_name,
  COUNT(*) as product_count
FROM products
WHERE status = 'active' AND category_name IS NOT NULL
GROUP BY category_name
ORDER BY product_count DESC
LIMIT 10;
```

### 4. Dashboard Operativo Integrado
**UID**: `saas-operativo`

Este dashboard proporciona una visión unificada de toda la plataforma:

- **Indicadores Generales**: Usuarios activos, conversaciones, productos activos y almacenes
- **Registro de Usuarios**: Gráfico temporal de registros de usuarios
- **Volumen de Mensajes**: Gráfico de actividad de mensajería
- **Creación de Productos**: Tendencia de creación de productos a lo largo del tiempo
- **Resumen por Tenant**: Tabla con métricas por tenant

**Consultas clave:**
```sql
-- Usuarios activos
SELECT COUNT(*) as total_users FROM users WHERE is_active = true;

-- Registro de usuarios
SELECT 
  date_trunc('day', created_at) as time,
  COUNT(*) as nuevos_usuarios
FROM users
GROUP BY date_trunc('day', created_at)
ORDER BY time;

-- Resumen por tenant
SELECT 
  t.name as tenant,
  COUNT(DISTINCT u.id) as usuarios,
  MAX(u.created_at) as ultimo_usuario,
  MAX(u.last_login) as ultimo_login
FROM tenants t
LEFT JOIN users u ON t.id = u.tenant_id
GROUP BY t.name
ORDER BY usuarios DESC;
```

## Configuración Técnica

### Archivos de Configuración

Los dashboards están definidos como archivos JSON en:
- `monitoring/grafana/dashboards/iam-dashboard.json`
- `monitoring/grafana/dashboards/chat-dashboard.json`
- `monitoring/grafana/dashboards/pim-dashboard.json`
- `monitoring/grafana/dashboards/operativo-integrado.json`

Las fuentes de datos están configuradas en:
- `monitoring/grafana/provisioning/datasources/datasources.yml`

### Provisión Automática

Grafana está configurado para cargar automáticamente los dashboards:
```yaml
# monitoring/grafana/provisioning/dashboards/dashboards.yml
apiVersion: 1

providers:
  - name: 'SaaS Dashboards'
    orgId: 1
    folder: 'SaaS Monitoring'
    type: file
    disableDeletion: false
    updateIntervalSeconds: 10
    allowUiUpdates: true
    options:
      path: /var/lib/grafana/dashboards
```

## Casos de Uso y Beneficios

### Monitoreo Operacional
- Detectar picos inusuales de usuarios/mensajes/productos
- Identificar tendencias de uso por tenant
- Visualizar distribución de carga en el sistema

### Business Intelligence
- Analizar patrones de registro de usuarios
- Evaluar tendencias de comunicaciones por dirección
- Identificar categorías y marcas más populares

### Diagnóstico
- Detectar problemas de entrega de mensajes
- Identificar tenants inactivos
- Supervisar la creación de recursos

## Extensión y Personalización

### Variables de Dashboard
Se pueden añadir variables para filtrar por:
- Rango de tiempo personalizado
- Tenant específico
- Estado de usuario/producto/mensaje

### Alertas
Se pueden configurar alertas para:
- Caídas en registro de usuarios
- Picos anormales de mensajes
- Inactividad de tenants

### Mejoras Futuras Recomendadas
- Dashboard de métricas de sesión (tiempo de uso, retención)
- Dashboard de análisis por ubicación geográfica
- Dashboard de KPIs de negocio por tenant 