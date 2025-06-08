# Sistema Unificado de Filtros y Paginación - Patrón Criteria

## 🎯 Objetivo

Unificar y estandarizar el manejo de filtros, ordenamiento y paginación en todos los endpoints de listado de entidades, eliminando código duplicado y facilitando la migración a otros proyectos como PIM.

## 📁 Estructura Implementada

```
src/shared/domain/criteria/
├── criteria.go          # Estructuras principales del patrón criteria  
├── request.go           # DTOs y builders para request parameters base
└── repository.go        # Interfaces genéricas para repositorios

src/shared/infrastructure/criteria/
└── sql_criteria.go      # Conversor de criteria a SQL
└── controller_helper.go # Helper base para controllers (funciones genéricas)

# Builders específicos en cada módulo:
src/user/infrastructure/criteria/
└── user_criteria_builder.go

src/tenant/infrastructure/criteria/
└── tenant_criteria_builder.go

src/role/infrastructure/criteria/
└── role_criteria_builder.go

src/plan/infrastructure/criteria/
└── plan_criteria_builder.go
```

## 🔧 Componentes Principales

### 1. Estructura de Criteria (Shared)

```go
type Criteria struct {
    Filters    Filters      // Colección de filtros
    Order      Order        // Criterio de ordenamiento
    Pagination Pagination   // Configuración de paginación
}

type Filter struct {
    Field    string      // Campo a filtrar
    Operator string      // Operador (=, !=, >, <, LIKE, IN, etc.)
    Value    interface{} // Valor del filtro
}
```

### 2. Builder Pattern Base (Shared)

```go
builder := criteria.NewCriteriaBuilder()
    .FromURLValues(c.Request.URL.Query())
    .AddUUIDFilter("tenant_id", tenantID)
    .AddEqualFilter("status", "ACTIVE")
    .AddLikeFilter("name", searchTerm)

criteria := builder.Build()
```

### 3. Builders Específicos por Módulo

Cada módulo implementa su propio builder que extiende la funcionalidad base:

```go
// En el módulo User
userBuilder := userCriteria.NewUserCriteriaBuilder()
validCriteria := userBuilder.BuildValidated(c)

// En el módulo Tenant  
tenantBuilder := tenantCriteria.NewTenantCriteriaBuilder()
validCriteria := tenantBuilder.BuildValidated(c)
```

### 4. Conversor SQL (Shared)

```go
converter := criteria.NewSQLCriteriaConverter()

// Para SELECT con filtros, order y paginación
query, params := converter.ToSelectSQL(baseQuery, criteria)

// Para COUNT solo con filtros
countQuery, countParams := converter.ToCountSQL(baseCountQuery, criteria)
```

### 5. Respuesta Genérica (Shared)

```go
type ListResponse[T any] struct {
    Items      []*T `json:"items"`
    TotalCount int  `json:"total_count"`
    Page       int  `json:"page"`
    PageSize   int  `json:"page_size"`
    TotalPages int  `json:"total_pages"`
}
```

## 🚀 Cómo Migrar un Endpoint Existente

### Paso 1: Actualizar la Interfaz del Repositorio

```go
// Antes
type UserRepository interface {
    List(ctx context.Context, limit, offset int) ([]*entity.User, error)
    Count(ctx context.Context) (int, error)
    // otros métodos...
}

// Después
type UserCriteriaRepository interface {
    UserRepository
    criteria.CriteriaRepository[entity.User]
}
```

### Paso 2: Implementar Métodos de Criteria en el Repositorio

```go
func (r *PostgresUserRepository) SearchByCriteria(ctx context.Context, crit criteria.Criteria) ([]*entity.User, error) {
    baseQuery := `
        SELECT id, email, first_name, last_name, tenant_id, role_id, 
               status, provider, created_at, updated_at 
        FROM users
    `
    
    converter := criteria.NewSQLCriteriaConverter()
    query, params := converter.ToSelectSQL(baseQuery, crit)
    
    rows, err := r.db.QueryContext(ctx, query, params...)
    if err != nil {
        return nil, err
    }
    defer rows.Close()
    
    return r.scanUsers(rows)
}

func (r *PostgresUserRepository) CountByCriteria(ctx context.Context, crit criteria.Criteria) (int, error) {
    baseQuery := "SELECT COUNT(*) FROM users"
    
    converter := criteria.NewSQLCriteriaConverter()
    query, params := converter.ToCountSQL(baseQuery, crit)
    
    var count int
    err := r.db.QueryRowContext(ctx, query, params...).Scan(&count)
    return count, err
}
```

### Paso 3: Crear el Builder Específico del Módulo

```go
// En user/infrastructure/criteria/user_criteria_builder.go
type UserCriteriaBuilder struct {
    helper  *sharedCriteria.EntityCriteriaHelper
    builder *criteria.CriteriaBuilder
}

func (b *UserCriteriaBuilder) FromContext(c *gin.Context) *UserCriteriaBuilder {
    b.builder = b.helper.BuildBaseFromContext(c)
    
    // Filtros específicos de usuarios
    b.builder.AddUUIDFilter("tenant_id", c.Query("tenant_id"))
    b.builder.AddEqualFilter("status", c.Query("status"))
    b.builder.AddUUIDFilter("role_id", c.Query("role_id"))
    b.builder.AddLikeFilter("email", c.Query("email"))
    // ... más filtros específicos
    
    return b
}

func (b *UserCriteriaBuilder) BuildValidated(c *gin.Context) criteria.Criteria {
    searchCriteria := b.FromContext(c).Build()
    return b.helper.ValidateAndSanitizeCriteria(searchCriteria, b.GetAllowedFields())
}
```

### Paso 4: Crear el UseCase con Criteria

```go
type ListUsersByCriteriaUseCase struct {
    userRepo port.UserCriteriaRepository
}

func (uc *ListUsersByCriteriaUseCase) Execute(ctx context.Context, searchCriteria criteria.Criteria) (*criteria.ListResponse[entity.User], error) {
    users, err := uc.userRepo.SearchByCriteria(ctx, searchCriteria)
    if err != nil {
        return nil, err
    }

    total, err := uc.userRepo.CountByCriteria(ctx, searchCriteria)
    if err != nil {
        return nil, err
    }

    return criteria.NewListResponse(users, total, searchCriteria), nil
}
```

### Paso 5: Refactorizar el Controller

```go
type RefactoredUserHandler struct {
    // ... otros use cases
    listUsersByCriteriaUseCase *usecase.ListUsersByCriteriaUseCase
    criteriaBuilder            *userCriteria.UserCriteriaBuilder
}

func (h *RefactoredUserHandler) ListUsers(c *gin.Context) {
    // Una sola línea para construir criterios validados
    validCriteria := h.criteriaBuilder.BuildValidated(c)

    // Ejecutar búsqueda
    result, err := h.listUsersByCriteriaUseCase.Execute(c.Request.Context(), validCriteria)
    if err != nil {
        c.JSON(http.StatusBadRequest, gin.H{"error": err.Error()})
        return
    }

    c.JSON(http.StatusOK, result)
}
```

## 📝 Query Parameters Soportados

### Parámetros Básicos (todos los endpoints)
- `page`: Número de página (default: 1)
- `page_size`: Tamaño de página (default: 10, max: 100)
- `sort_by`: Campo de ordenamiento (default: "created_at")
- `sort_dir`: Dirección (asc/desc, default: "desc")

### Filtros por Entidad

#### Usuarios (`/users`)
- `tenant_id`: UUID del tenant
- `status`: Estado del usuario (ACTIVE, INACTIVE, SUSPENDED, DELETED)
- `role_id`: UUID del rol
- `email`: Búsqueda por email (LIKE)
- `first_name`: Búsqueda por nombre (LIKE)
- `last_name`: Búsqueda por apellido (LIKE)
- `provider`: Proveedor (LOCAL, GOOGLE)

#### Tenants (`/tenants`)
- `owner_id`: UUID del propietario
- `status`: Estado (ACTIVE, INACTIVE, SUSPENDED)
- `type`: Tipo (PERSONAL, STARTUP, BUSINESS, ENTERPRISE)
- `plan_id`: UUID del plan
- `name`: Búsqueda por nombre (LIKE)
- `slug`: Búsqueda por slug (LIKE)
- `domain`: Búsqueda por dominio (LIKE)
- `active`: Solo activos (true/false)

#### Roles (`/roles`)
- `tenant_id`: UUID del tenant
- `type`: Tipo (SYSTEM, TENANT)
- `status`: Estado (ACTIVE, INACTIVE)
- `name`: Búsqueda por nombre (LIKE)
- `system`: Solo roles de sistema (true/false)
- `active`: Solo activos (true/false)

#### Planes (`/plans`)
- `type`: Tipo (FREE, STARTER, PROFESSIONAL, ENTERPRISE)
- `status`: Estado (ACTIVE, INACTIVE)
- `name`: Búsqueda por nombre (LIKE)
- `currency`: Moneda
- `min_price`: Precio mínimo
- `max_price`: Precio máximo
- `active`: Solo activos (true/false)

## 🔍 Operadores de Filtro Soportados

```go
const (
    OpEqual              = "="
    OpNotEqual           = "!="
    OpGreaterThan        = ">"
    OpGreaterThanOrEqual = ">="
    OpLessThan           = "<"
    OpLessThanOrEqual    = "<="
    OpLike               = "LIKE"      // Para búsquedas de texto
    OpIn                 = "IN"        // Para arrays de valores
    OpIsNull             = "NULL"      // Para campos nulos
    OpIsNotNull          = "NOT NULL"  // Para campos no nulos
)
```

## 📋 Ejemplos de Uso

### Búsqueda Básica
```bash
GET /api/v1/users?page=1&page_size=20&sort_by=email&sort_dir=asc
```

### Filtros Múltiples
```bash
GET /api/v1/users?tenant_id=123e4567-e89b-12d3-a456-426614174000&status=ACTIVE&email=john
```

### Búsqueda Compleja
```bash
GET /api/v1/tenants?type=BUSINESS&active=true&name=acme&sort_by=created_at&sort_dir=desc&page=2&page_size=50
```

## ✅ Ventajas del Nuevo Sistema

1. **Separación de Responsabilidades**: Shared solo contiene abstracciones, cada módulo maneja sus propios criterios
2. **Código Reutilizable**: La estructura base se reutiliza, cada módulo extiende según sus necesidades
3. **Sin Duplicación**: Eliminación del parseo manual repetitivo
4. **Validación Automática**: Campos permitidos se validan automáticamente por módulo
5. **Seguridad**: Previene inyección SQL usando parámetros preparados
6. **Consistencia**: Todos los endpoints usan la misma estructura de respuesta
7. **Extensible**: Fácil agregar nuevos operadores o tipos de filtro
8. **Mantenible**: Cada módulo es independiente pero usa la misma base

## 🔄 Plan de Migración

### Fase 1: Implementación Base (Completada)
- ✅ Crear estructura de criteria en shared
- ✅ Implementar conversor SQL  
- ✅ Crear helpers base para controllers
- ✅ Crear builders específicos en cada módulo
- ✅ Ejemplo con usuario refactorizado

### Fase 2: Migración Gradual
1. **Users Module**: Reemplazar endpoint actual con criterios
2. **Tenants Module**: Migrar filtros complejos existentes
3. **Roles Module**: Unificar métodos de búsqueda
4. **Plans Module**: Agregar filtros faltantes

### Fase 3: Limpieza
1. Eliminar métodos específicos de filtrado en repositorios
2. Remover DTOs de request específicos obsoletos
3. Actualizar documentación de API

### Fase 4: Copia a Otros Proyectos
1. Copiar carpeta `shared/domain/criteria`
2. Copiar carpeta `shared/infrastructure/criteria`
3. Crear builders específicos para entidades del nuevo proyecto
4. Implementar interfaces en repositorios del nuevo proyecto

## 🛠️ Personalización por Proyecto

Para adaptar a PIM u otro proyecto:

1. **Copiar estructura base** de criteria desde shared
2. **Crear builders específicos** para cada entidad del nuevo proyecto
3. **Definir campos permitidos** específicos para cada entidad
4. **Implementar interfaces** en repositorios existentes
5. **Migrar controllers** usando el nuevo patrón

### Ejemplo para PIM

```go
// En pim/src/product/infrastructure/criteria/product_criteria_builder.go
type ProductCriteriaBuilder struct {
    helper  *sharedCriteria.EntityCriteriaHelper
    builder *criteria.CriteriaBuilder
}

func (b *ProductCriteriaBuilder) FromContext(c *gin.Context) *ProductCriteriaBuilder {
    b.builder = b.helper.BuildBaseFromContext(c)
    
    // Filtros específicos de productos PIM
    b.builder.AddLikeFilter("name", c.Query("name"))
    b.builder.AddLikeFilter("sku", c.Query("sku"))
    b.builder.AddEqualFilter("category_id", c.Query("category_id"))
    b.builder.AddEqualFilter("status", c.Query("status"))
    
    return b
}
```

El sistema es completamente agnóstico al dominio y se adapta fácilmente a cualquier entidad.

## 🔐 Consideraciones de Seguridad

1. **Whitelist de Campos**: Solo campos permitidos por módulo se pueden filtrar/ordenar
2. **Validación de Tipos**: UUIDs, enteros y strings se validan automáticamente  
3. **Parámetros Preparados**: Todas las consultas SQL usan parámetros preparados
4. **Límites de Paginación**: Se impone un máximo de 100 elementos por página
5. **Sanitización**: Los valores se sanitizan antes de usar en consultas
6. **Aislamiento por Módulo**: Cada módulo controla qué campos son accesibles

## 📊 Comparación: Antes vs Después

| Aspecto | Antes | Después |
|---------|-------|---------|
| **Líneas de código por endpoint** | ~50-80 líneas | ~5-10 líneas |
| **Parseo de parámetros** | Manual, repetitivo | Automático por módulo |
| **Validación** | Manual, inconsistente | Automática, por módulo |
| **Filtros soportados** | Limitados, hardcoded | Dinámicos, extensibles |
| **Respuesta** | Estructura específica | Genérica, consistente |
| **Reutilización** | Código duplicado | Base compartida + extensiones |
| **Mantenimiento** | Alto (cambios en N lugares) | Bajo (cambios centralizados o por módulo) |
| **Testing** | Múltiples casos por endpoint | Testing base + específico por módulo |
| **Separación de responsabilidades** | Baja | Alta (shared + módulos específicos) |

El nuevo sistema reduce significativamente la complejidad y mejora la mantenibilidad del código manteniendo una clara separación de responsabilidades. 