# Convenciones OpenAPI Multi-Proyecto

## 🎯 Objetivo

Este documento establece las convenciones estándar para la documentación OpenAPI en todos los proyectos del ecosistema multi-tenant (IAM, PIM, y futuros proyectos).

## 📁 Estructura de Archivos

### Ubicación Estándar
Todos los proyectos deben seguir esta estructura:

```
{proyecto}/
├── api-docs/
│   └── openapi.yaml          # ✅ Especificación OpenAPI del proyecto
├── documentation/            # 📋 Documentación específica del proyecto
│   └── [archivos .md]
└── README.md                # 📖 Documentación principal del proyecto
```

### Nomenclatura
- **Archivo**: `openapi.yaml` (NO `swagger.yaml`)
- **Ubicación**: `{proyecto}/api-docs/openapi.yaml`
- **Razones**:
  - OpenAPI es el nombre oficial del estándar
  - Carpeta `api-docs/` organiza claramente la documentación de API
  - Facilita automatización (generadores, validadores)
  - Estructura predecible entre proyectos

## 📖 Estructura OpenAPI Estándar

### Información Base
```yaml
openapi: 3.1.0
info:
  title: Multi-Tenant {PROYECTO} API
  version: 1.0.0
  description: API para gestión de {dominio} multi-tenant
  contact:
    name: Equipo {PROYECTO}
    email: team@company.com

servers:
  - url: /{proyecto}/api/v1
    description: Base URL para la API v1
```

### Componentes Obligatorios

#### Autenticación
```yaml
components:
  securitySchemes:
    BearerAuth:
      type: http
      scheme: bearer
      bearerFormat: JWT
    tenantId:
      type: apiKey
      in: header
      name: X-Tenant-ID

security:
  - BearerAuth: []
  - tenantId: []
```

#### Parámetros de Paginación Base
```yaml
components:
  parameters:
    PageParam:
      name: page
      in: query
      description: Número de página (default: 1)
      schema:
        type: integer
        minimum: 1
        default: 1
    
    PageSizeParam:
      name: page_size
      in: query
      description: Elementos por página (1-100, default: 10)
      schema:
        type: integer
        minimum: 1
        maximum: 100
        default: 10
    
    SortByParam:
      name: sort_by
      in: query
      description: Campo de ordenamiento
      schema:
        type: string
        default: "created_at"
    
    SortDirParam:
      name: sort_dir
      in: query
      description: Dirección de ordenamiento
      schema:
        type: string
        enum: [asc, desc]
        default: "desc"
```

#### Schema de Respuesta de Lista
```yaml
components:
  schemas:
    # Schema genérico para listas paginadas
    ListResponseBase:
      type: object
      properties:
        total_count:
          type: integer
          description: Total de elementos
        page:
          type: integer
          description: Página actual
        page_size:
          type: integer
          description: Elementos por página
        total_pages:
          type: integer
          description: Total de páginas
      required: [total_count, page, page_size, total_pages]

    # Ejemplo de uso específico
    UserListResponse:
      allOf:
        - $ref: '#/components/schemas/ListResponseBase'
        - type: object
          properties:
            items:
              type: array
              items:
                $ref: '#/components/schemas/User'
```

## 🔍 Estándares de Endpoints

### Endpoints de Listado
Todos los endpoints de listado deben incluir:

```yaml
/entities:
  get:
    summary: Listar {entidades} con filtros y paginación
    tags: ['{Entidades}']
    parameters:
      - $ref: '#/components/parameters/PageParam'
      - $ref: '#/components/parameters/PageSizeParam'
      - $ref: '#/components/parameters/SortByParam'
      - $ref: '#/components/parameters/SortDirParam'
      # Filtros específicos de la entidad
      - name: status
        in: query
        description: Estado de la entidad
        schema:
          type: string
          enum: [ACTIVE, INACTIVE, DELETED]
    responses:
      '200':
        description: Lista de {entidades}
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/{Entity}ListResponse'
      '400':
        $ref: '#/components/responses/BadRequest'
      '401':
        $ref: '#/components/responses/Unauthorized'
```

### Respuestas de Error Estándar
```yaml
components:
  responses:
    BadRequest:
      description: Solicitud inválida
      content:
        application/json:
          schema:
            $ref: '#/components/schemas/ErrorResponse'
    
    Unauthorized:
      description: No autorizado
      content:
        application/json:
          schema:
            $ref: '#/components/schemas/ErrorResponse'
    
    NotFound:
      description: Recurso no encontrado
      content:
        application/json:
          schema:
            $ref: '#/components/schemas/ErrorResponse'

  schemas:
    ErrorResponse:
      type: object
      properties:
        error:
          type: string
          description: Mensaje de error
        code:
          type: string
          description: Código de error
        details:
          type: object
          description: Detalles adicionales del error
      required: [error]
```

## 🏷️ Convenciones de Tags

### Tags por Módulo
Organizar endpoints por entidades de dominio:

```yaml
tags:
  - name: Users
    description: Gestión de usuarios
  - name: Roles  
    description: Gestión de roles y permisos
  - name: Tenants
    description: Gestión de tenants
  - name: Health
    description: Endpoints de salud y monitoreo
```

## 🔧 Filtros por Tipo de Entidad

### Filtros Comunes (Todos los proyectos)
- `status`: Estado de la entidad (ACTIVE, INACTIVE, etc.)
- `created_after`: Filtro por fecha de creación (ISO 8601)
- `created_before`: Filtro por fecha de creación (ISO 8601)
- `search`: Búsqueda general por texto

### Filtros Multi-Tenant (Aplicables)
- `tenant_id`: UUID del tenant (obligatorio en contextos multi-tenant)

### Ejemplos por Proyecto

#### IAM - Usuarios
```yaml
parameters:
  - name: tenant_id
    description: UUID del tenant
  - name: status
    description: Estado del usuario
    enum: [ACTIVE, INACTIVE, SUSPENDED, DELETED]
  - name: role_id
    description: UUID del rol
  - name: email
    description: Búsqueda por email (LIKE)
  - name: provider
    description: Proveedor de autenticación
    enum: [LOCAL, GOOGLE, GITHUB]
```

#### PIM - Productos
```yaml
parameters:
  - name: category_id
    description: UUID de la categoría
  - name: status
    description: Estado del producto
    enum: [DRAFT, ACTIVE, INACTIVE, DISCONTINUED]
  - name: sku
    description: Búsqueda por SKU
  - name: name
    description: Búsqueda por nombre (LIKE)
  - name: min_price
    description: Precio mínimo
    schema:
      type: number
      format: decimal
```

## 🚀 Herramientas y Automatización

### Generación de Código
Los archivos OpenAPI deben ser compatibles con:
- **Swagger Codegen**: Para generar clientes SDK
- **OpenAPI Generator**: Para generar modelos TypeScript
- **Postman**: Para importar colecciones automáticamente

### Validación CI/CD
```yaml
# Ejemplo para GitHub Actions
- name: Validate OpenAPI
  run: |
    npx swagger-cli validate api-docs/openapi.yaml
    
- name: Generate TypeScript Types
  run: |
    npx openapi-typescript api-docs/openapi.yaml --output types/api.ts
```

### Configuración de Servidor
```go
// En Go, configuración estándar
type APIConfig struct {
    OpenAPIPath string
    SwaggerUIPath string
}

config := APIConfig{
    OpenAPIPath: "./api-docs/openapi.yaml",
    SwaggerUIPath: "/swagger-ui/",
}
```

## 📋 Checklist de Implementación

### Para Nuevos Proyectos
- [ ] Crear carpeta `api-docs/`
- [ ] Crear archivo `openapi.yaml` con estructura base
- [ ] Implementar autenticación estándar
- [ ] Agregar parámetros de paginación base
- [ ] Definir schemas de respuesta estándar
- [ ] Configurar tags por módulo
- [ ] Implementar filtros específicos del dominio
- [ ] Configurar Swagger UI en el servidor
- [ ] Agregar validación en CI/CD

### Para Proyectos Existentes
- [ ] Migrar archivo a `api-docs/openapi.yaml`
- [ ] Actualizar referencias en código
- [ ] Estandarizar estructura de respuestas
- [ ] Unificar parámetros de paginación
- [ ] Agregar filtros faltantes
- [ ] Actualizar configuración del servidor

## 🔗 Integración con Kong Gateway

### Configuración Estándar
```yaml
services:
  - name: iam-service
    url: http://iam:8080
    routes:
      - name: iam-api
        paths: ['/iam/api/v1']
  
  - name: pim-service  
    url: http://pim:8080
    routes:
      - name: pim-api
        paths: ['/pim/api/v1']
```

## 📚 Referencias

- [OpenAPI Specification](https://spec.openapis.org/oas/v3.1.0)
- [Swagger Editor](https://editor.swagger.io/)
- [OpenAPI Generator](https://openapi-generator.tech/)
- [Best Practices for API Design](https://swagger.io/resources/articles/best-practices-in-api-design/)

---

**Resultado:** Convenciones unificadas que garantizan consistencia, mantenibilidad y automatización en todos los proyectos del ecosistema multi-tenant. 