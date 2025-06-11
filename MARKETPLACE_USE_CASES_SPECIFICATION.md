# 📋 ESPECIFICACIÓN DE CASOS DE USO - MARKETPLACE MULTI-TENANT

## 🎯 Introducción

Este documento especifica los **6 casos de uso principales** implementados para el módulo marketplace del sistema SaaS multi-tenant. Cada caso de uso resuelve un problema específico en la gestión de taxonomías híbridas entre categorías globales del marketplace y configuraciones personalizadas por tenant.

## 🏗️ Arquitectura de Casos de Uso

### Patrón Implementado: **Arquitectura Hexagonal**
- **Application Layer**: Casos de uso, requests, responses
- **Domain Layer**: Entidades, puertos (interfaces)
- **Infrastructure Layer**: Repositorios, controladores (pendientes)

### Flujo de Datos
```
HTTP Request → Controller → UseCase → Repository → Database
                    ↓
Response ← Mapper ← Domain Entity ← Query Result
```

---

## 📚 CASOS DE USO IMPLEMENTADOS

### 1. 🏷️ **CREATE MARKETPLACE CATEGORY**

#### **Propósito**
Crear categorías globales del marketplace que servirán como base para todos los tenants.

#### **Importancia en el Marketplace Multi-Tenant**
- **Estandarización**: Establece una taxonomía global consistente
- **Escalabilidad**: Permite agregar nuevas categorías sin afectar tenants existentes
- **Governance**: Control centralizado de la estructura de categorías
- **SEO**: URLs y slugs consistentes para el marketplace público

#### **Funcionalidades Clave**
- ✅ Validación de slugs únicos globalmente
- ✅ Jerarquía automática (máximo 3 niveles)
- ✅ Generación automática de slugs desde nombres
- ✅ Validación de nombres duplicados por nivel
- ✅ Soft delete para mantener integridad referencial

#### **Request/Response**
```go
// Request
type CreateMarketplaceCategoryRequest struct {
    Name        string  `json:"name" binding:"required"`
    Slug        string  `json:"slug,omitempty"`
    Description string  `json:"description"`
    ParentID    *string `json:"parent_id,omitempty"`
}

// Response
type MarketplaceCategoryResponse struct {
    ID          string    `json:"id"`
    Name        string    `json:"name"`
    Slug        string    `json:"slug"`
    Description string    `json:"description"`
    ParentID    *string   `json:"parent_id"`
    Level       int       `json:"level"`
    IsActive    bool      `json:"is_active"`
    CreatedAt   time.Time `json:"created_at"`
    UpdatedAt   time.Time `json:"updated_at"`
}
```

#### **Casos de Uso del Negocio**
1. **Admin del Marketplace**: Crear nueva categoría "Electrónicos"
2. **Expansión de Catálogo**: Agregar subcategoría "Smartphones" bajo "Electrónicos"
3. **Reorganización**: Mover categorías manteniendo jerarquía

---

### 2. 🔗 **MAP TENANT CATEGORY**

#### **Propósito**
Mapear categorías existentes del tenant a categorías globales del marketplace, permitiendo personalización sin perder la estructura global.

#### **Importancia en el Marketplace Multi-Tenant**
- **Flexibilidad**: Cada tenant mantiene su estructura existente
- **Migración Gradual**: No requiere reestructurar categorías existentes
- **Personalización**: Nombres custom por tenant
- **Compatibilidad**: Integración con sistemas PIM existentes

#### **Funcionalidades Clave**
- ✅ Mapeo 1:1 entre categoría tenant y marketplace
- ✅ Nombres personalizados por tenant
- ✅ Validación de duplicados por tenant
- ✅ Preservación de jerarquía tenant original
- ✅ Auditoría completa de cambios

#### **Request/Response**
```go
// Request
type MapTenantCategoryRequest struct {
    CategoryID            string  `json:"category_id" binding:"required"`
    MarketplaceCategoryID string  `json:"marketplace_category_id" binding:"required"`
    CustomName            *string `json:"custom_name,omitempty"`
}

// Response
type TenantCategoryMappingResponse struct {
    ID                    string    `json:"id"`
    TenantID              string    `json:"tenant_id"`
    CategoryID            string    `json:"category_id"`
    MarketplaceCategoryID string    `json:"marketplace_category_id"`
    CustomName            *string   `json:"custom_name"`
    CreatedAt             time.Time `json:"created_at"`
    UpdatedAt             time.Time `json:"updated_at"`
}
```

#### **Casos de Uso del Negocio**
1. **Tenant "TechStore"**: Mapea su categoría "Gadgets" → marketplace "Electrónicos"
2. **Tenant "FashionHub"**: Mapea "Ropa Mujer" → marketplace "Moda Femenina"
3. **Personalización**: Tenant usa nombre "Tecnología" para categoría marketplace "Electrónicos"

---

### 3. 🎨 **EXTEND TENANT ATTRIBUTES**

#### **Propósito**
Permitir que cada tenant extienda las categorías con atributos completamente personalizados según sus necesidades específicas.

#### **Importancia en el Marketplace Multi-Tenant**
- **Diferenciación**: Cada tenant puede capturar datos únicos
- **Competitividad**: Atributos específicos del nicho de negocio
- **Flexibilidad Total**: Sin limitaciones de esquema fijo
- **Evolución**: Agregar atributos sin migraciones de BD

#### **Funcionalidades Clave**
- ✅ 5 tipos de atributos: text, number, boolean, select, multi_select
- ✅ Validaciones personalizadas por tipo
- ✅ Slugs únicos por tenant y categoría
- ✅ Atributos globales del tenant (sin categoría específica)
- ✅ Reglas de validación configurables

#### **Request/Response**
```go
// Request
type ExtendTenantAttributesRequest struct {
    CategoryID       string                    `json:"category_id" binding:"required"`
    CustomAttributes []CustomAttributeRequest  `json:"custom_attributes" binding:"required"`
}

type CustomAttributeRequest struct {
    Name            string                 `json:"name" binding:"required"`
    Slug            string                 `json:"slug,omitempty"`
    Type            string                 `json:"type" binding:"required"`
    IsFilterable    bool                   `json:"is_filterable"`
    IsSearchable    bool                   `json:"is_searchable"`
    ValidationRules map[string]interface{} `json:"validation_rules,omitempty"`
}

// Response
type TenantAttributeExtensionResponse struct {
    CategoryID       string                        `json:"category_id"`
    TotalAttributes  int                           `json:"total_attributes"`
    CreatedAttributes []TenantCustomAttributeInfo  `json:"created_attributes"`
    Errors           []AttributeCreationError      `json:"errors,omitempty"`
}
```

#### **Casos de Uso del Negocio**
1. **Tenant Joyería**: Agrega atributos "Quilates", "Tipo de Piedra", "Certificación"
2. **Tenant Automotriz**: Agrega "Año", "Kilometraje", "Tipo de Combustible"
3. **Tenant Inmobiliario**: Agrega "Metros Cuadrados", "Número de Habitaciones", "Estrato"

---

### 4. ✅ **VALIDATE CATEGORY HIERARCHY**

#### **Propósito**
Validar la integridad de jerarquías de categorías antes de realizar cambios, previniendo referencias circulares y violaciones de profundidad.

#### **Importancia en el Marketplace Multi-Tenant**
- **Integridad de Datos**: Previene corrupción de jerarquías
- **Performance**: Evita queries infinitos por ciclos
- **Experiencia de Usuario**: Validación previa a cambios
- **Mantenimiento**: Detección temprana de problemas

#### **Funcionalidades Clave**
- ✅ Detección de referencias circulares
- ✅ Validación de profundidad máxima
- ✅ Análisis de impacto en categorías hijas
- ✅ Simulación de cambios (dry-run)
- ✅ Reportes detallados de validación

#### **Request/Response**
```go
// Request
type ValidateCategoryHierarchyRequest struct {
    CategoryID       string  `json:"category_id" binding:"required"`
    NewParentID      *string `json:"new_parent_id,omitempty"`
    MaxDepth         int     `json:"max_depth,omitempty"`
    ValidateChildren bool    `json:"validate_children"`
}

// Response
type CategoryHierarchyValidationResponse struct {
    IsValid          bool                        `json:"is_valid"`
    CategoryID       string                      `json:"category_id"`
    CurrentLevel     int                         `json:"current_level"`
    NewLevel         int                         `json:"new_level"`
    ValidationErrors []ValidationError           `json:"validation_errors,omitempty"`
    AffectedChildren []AffectedChildCategory     `json:"affected_children,omitempty"`
    CategoryPath     []CategoryPathItem          `json:"category_path"`
}
```

#### **Casos de Uso del Negocio**
1. **Reorganización**: Validar mover "Smartphones" de "Electrónicos" a "Accesorios"
2. **Prevención de Ciclos**: Evitar que "Electrónicos" sea hijo de "Smartphones"
3. **Límites de Profundidad**: Validar que no se excedan 3 niveles de jerarquía

---

### 5. 🔄 **SYNC MARKETPLACE CHANGES**

#### **Propósito**
Sincronizar cambios en categorías marketplace hacia las configuraciones de todos los tenants afectados.

#### **Importancia en el Marketplace Multi-Tenant**
- **Consistencia Global**: Cambios marketplace se propagan automáticamente
- **Mantenimiento**: Actualizaciones masivas sin intervención manual
- **Auditoría**: Trazabilidad completa de sincronizaciones
- **Flexibilidad**: Sincronización selectiva por tenant o categoría

#### **Funcionalidades Clave**
- ✅ Sincronización masiva o selectiva
- ✅ Dry-run para simular cambios
- ✅ Creación automática de mapeos faltantes
- ✅ Limpieza de mapeos huérfanos
- ✅ Reportes detallados de sincronización

#### **Request/Response**
```go
// Request
type SyncMarketplaceChangesRequest struct {
    TenantID              *string           `json:"tenant_id,omitempty"`
    MarketplaceCategoryID *string           `json:"marketplace_category_id,omitempty"`
    ChangeTypes           []string          `json:"change_types,omitempty"`
    ForceSync             bool              `json:"force_sync"`
    DryRun                bool              `json:"dry_run"`
    SyncOptions           SyncOptionsConfig `json:"sync_options"`
}

// Response
type SyncMarketplaceChangesResponse struct {
    SyncID               string                 `json:"sync_id"`
    SyncStatus           string                 `json:"sync_status"`
    TotalChanges         int                    `json:"total_changes"`
    AppliedChanges       int                    `json:"applied_changes"`
    FailedChanges        int                    `json:"failed_changes"`
    SyncResults          []SyncResult           `json:"sync_results"`
    AffectedTenants      []string               `json:"affected_tenants"`
    ExecutionTime        time.Duration          `json:"execution_time"`
}
```

#### **Casos de Uso del Negocio**
1. **Cambio Global**: Renombrar "Electrónicos" → "Tecnología" en todos los tenants
2. **Nueva Categoría**: Propagar "Sustentabilidad" a tenants que vendan productos eco-friendly
3. **Limpieza**: Remover mapeos a categorías marketplace eliminadas

---

### 6. 📊 **GET TENANT TAXONOMY**

#### **Propósito**
Obtener la taxonomía completa de un tenant, combinando sus categorías, mapeos marketplace y atributos personalizados.

#### **Importancia en el Marketplace Multi-Tenant**
- **Vista Unificada**: Taxonomía completa del tenant en una sola consulta
- **Performance**: Datos pre-agregados para UIs complejas
- **Flexibilidad**: Múltiples formatos de salida (tree, flat, hierarchical)
- **Integración**: API para sistemas externos del tenant

#### **Funcionalidades Clave**
- ✅ 3 formatos de salida: tree, flat, hierarchical
- ✅ Inclusión opcional de datos marketplace
- ✅ Inclusión opcional de atributos personalizados
- ✅ Filtrado por categorías específicas
- ✅ Metadatos y estadísticas de taxonomía

#### **Request/Response**
```go
// Request
type GetTenantTaxonomyRequest struct {
    TenantID              string   `json:"tenant_id" binding:"required"`
    IncludeCustomAttributes bool     `json:"include_custom_attributes"`
    IncludeMarketplaceData  bool     `json:"include_marketplace_data"`
    CategoryIDs           []string `json:"category_ids,omitempty"`
    MaxDepth              *int     `json:"max_depth,omitempty"`
    Format                string   `json:"format"`
}

// Response
type TenantTaxonomyResponse struct {
    TenantID            string                    `json:"tenant_id"`
    Format              string                    `json:"format"`
    TotalCategories     int                       `json:"total_categories"`
    TotalMappings       int                       `json:"total_mappings"`
    Categories          []TenantCategoryNode      `json:"categories"`
    CustomAttributes    []TenantCustomAttributeInfo `json:"custom_attributes,omitempty"`
    Metadata            TaxonomyMetadata          `json:"metadata"`
}
```

#### **Casos de Uso del Negocio**
1. **Dashboard Tenant**: Mostrar árbol completo de categorías con estadísticas
2. **Formulario Producto**: Cargar categorías disponibles para clasificación
3. **Exportación**: Generar reporte de taxonomía para auditoría
4. **Integración**: Sincronizar con ERP externo del tenant

---

## 🔄 FLUJOS DE INTEGRACIÓN

### **Flujo 1: Onboarding de Nuevo Tenant**
```
1. Admin crea categorías marketplace (CREATE_MARKETPLACE_CATEGORY)
2. Tenant mapea sus categorías existentes (MAP_TENANT_CATEGORY)
3. Tenant agrega atributos específicos (EXTEND_TENANT_ATTRIBUTES)
4. Sistema valida integridad (VALIDATE_CATEGORY_HIERARCHY)
5. Tenant obtiene taxonomía final (GET_TENANT_TAXONOMY)
```

### **Flujo 2: Evolución del Marketplace**
```
1. Admin agrega nueva categoría marketplace (CREATE_MARKETPLACE_CATEGORY)
2. Sistema sincroniza a tenants relevantes (SYNC_MARKETPLACE_CHANGES)
3. Tenants mapean nuevas categorías (MAP_TENANT_CATEGORY)
4. Tenants extienden con atributos (EXTEND_TENANT_ATTRIBUTES)
```

### **Flujo 3: Reorganización de Categorías**
```
1. Admin valida cambios propuestos (VALIDATE_CATEGORY_HIERARCHY)
2. Admin aplica cambios si válidos (CREATE_MARKETPLACE_CATEGORY)
3. Sistema sincroniza cambios (SYNC_MARKETPLACE_CHANGES)
4. Tenants verifican taxonomía actualizada (GET_TENANT_TAXONOMY)
```

---

## 📈 BENEFICIOS DE LA IMPLEMENTACIÓN

### **Para el Marketplace**
- ✅ **Escalabilidad**: Agregar tenants sin reestructurar
- ✅ **Consistencia**: Taxonomía global estandarizada
- ✅ **Flexibilidad**: Personalización por tenant
- ✅ **Mantenimiento**: Cambios centralizados

### **Para los Tenants**
- ✅ **Migración Suave**: Mantienen estructura existente
- ✅ **Personalización**: Atributos específicos del negocio
- ✅ **Autonomía**: Control sobre su taxonomía
- ✅ **Integración**: APIs para sistemas externos

### **Para el Desarrollo**
- ✅ **Arquitectura Limpia**: Separación clara de responsabilidades
- ✅ **Testabilidad**: Casos de uso aislados y testeable
- ✅ **Mantenibilidad**: Código organizado y documentado
- ✅ **Extensibilidad**: Fácil agregar nuevos casos de uso

---

## 🚀 PRÓXIMOS PASOS

### **Infraestructura Pendiente**
1. **Repositorios PostgreSQL**: Implementar persistencia
2. **Controladores HTTP**: Exponer APIs REST
3. **Tests Unitarios**: Validar lógica de negocio
4. **Tests de Integración**: Validar flujos completos

### **Funcionalidades Futuras**
1. **Cache**: Redis para taxonomías frecuentes
2. **Events**: Notificaciones de cambios
3. **Versionado**: Historial de cambios de taxonomía
4. **Analytics**: Métricas de uso de categorías

---

*Documento generado: 2025-06-11*  
*Versión: 1.0*  
*Estado: Casos de uso implementados, infraestructura pendiente* 