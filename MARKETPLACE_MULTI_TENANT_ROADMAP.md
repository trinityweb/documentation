# 🚀 MARKETPLACE MULTI-TENANT - ROADMAP TÉCNICO

*Versión: 1.2*  
*Fecha: 2024-12-08*  
*Status: ✅ Especificación Completa + Migración Quickstart*

## 📚 DOCUMENTACIÓN RELACIONADA

- 📋 [**PROJECT_TRACKING.md**](./PROJECT_TRACKING.md) - Seguimiento épicas y tareas
- 📝 [**PROJECT_JOURNAL.md**](./PROJECT_JOURNAL.md) - Bitácora del proyecto  
- 🚀 [**QUICKSTART_MIGRATION_SPEC.md**](./QUICKSTART_MIGRATION_SPEC.md) - **NUEVO**: Migración quickstart YAML → BD
- 📊 [Análisis Servicios Kong](../api-gateway/kong.yml)
- 🔗 [Especificaciones OpenAPI](../combined-services-postman-collection.json)

## 🎯 OBJETIVOS Y JUSTIFICACIÓN

### 🧩 El Problema que Resolvemos

**Caso Real**: María tiene una tienda de ropa en Bahía Blanca
- En **MercadoLibre**: Debe elegir entre 500+ subcategorías predefinidas
- Sus productos se pierden en categorías genéricas como "Remera > Mujer > Manga Corta"
- No puede agregar "Talle Local" o "Calce Bahiense" que sus clientas entienden
- **Resultado**: Productos mal categorizados = menos ventas

**Con Nuestro Sistema**:
- Empieza con categorías marketplace simples: "Remeras"
- Agrega sus propias variaciones: "Remeras Playeras", "Remeras de Abrigo"
- Define talles locales: "S", "M", "L", "Talle Único"
- **Resultado**: Catálogo que habla como María y sus clientes

### 💡 Principios de Diseño

#### 1. **Progressive Disclosure** 
*"Mostrar complejidad solo cuando se necesita"*

```
Seller nuevo → 3 categorías básicas → Listo para vender
Seller experimentado → + Atributos custom → + Variantes → + Configuraciones avanzadas
```

#### 2. **Sensible Defaults**
*"El sistema debe funcionar perfecto 'out-of-the-box'"*

- Categorías marketplace cubren 80% de casos de uso
- Atributos estándar (talle, color, marca) ya configurados  
- Mapeo automático entre productos y marketplace

#### 3. **Escape Hatches**
*"Siempre una salida cuando lo estándar no alcanza"*

- Nombres custom para categorías
- Valores adicionales en atributos
- Atributos 100% propios del tenant
- Reglas de negocio específicas

---

## 🏗️ Análisis de Arquitectura Actual

### ✅ Lo que Ya Tenemos (y Aprovechamos)
- **IAM Service**: Multi-tenancy sólido con roles por tenant
- **PIM Service**: Gestión de productos con categorías y atributos flexibles
- **Sistema de Filtros**: Patrón Criteria que extiende naturalmente a marketplace
- **Backoffice**: UI base para administración

### 🔄 Lo que Necesitamos Adaptar

#### Problema 1: **Categorías Solo por Tenant**
**Situación actual**: Cada tenant crea sus categorías desde cero
```sql
-- Actual: Categorías aisladas por tenant
categories: tenant_id, name, parent_id
```

**Problema para marketplace**: 
- Comprador busca "remeras" → encuentra 50 variaciones diferentes
- Imposible filtrar cross-tenant
- No hay navegación consistente

**Solución híbrida**:
```sql
-- Nivel Marketplace (global): Navegación consistente
marketplace_categories: id, name, slug, parent_id

-- Nivel Tenant (custom): Personalización
tenant_category_mappings: tenant_id, marketplace_category_id, custom_name
```

#### Problema 2: **Búsqueda Fragmentada**
**Situación actual**: Búsqueda solo dentro del tenant
**Problema**: Comprador no puede comparar productos entre vendedores
**Solución**: Motor de búsqueda cross-tenant con filtros marketplace

---

## 🎯 FASE 1: FUNDACIÓN MARKETPLACE (Semanas 1-4)

### 1.1 Extensión del Modelo de Datos 
**Responsable**: Backend Developer | **Estimación**: 5 días

#### 💭 ¿Por qué necesitamos tablas marketplace separadas?

**Caso de uso**: Vendedora de accesorios quiere vender "Aros" pero en MercadoLibre debe elegir entre:
- Joyas > Aros > Aros de Argolla > Para Mujer > Acero Inoxidable
- Bisutería > Aros > Colgantes > Fashion > Plateados

**Con marketplace_categories**:
- Vendedora ve: "Accesorios → Aros" (simple)
- Comprador navega: Categoría estándar entre todos los vendors
- Sistema internamente: Mapea a categoría marketplace global

#### [ ] 1.1.1 Crear Taxonomía Marketplace Base

**Valor para el seller**: Una categoría "Remeras" funciona para todos, pero cada uno la puede llamar como quiera

- [ ] **Archivo**: `services/saas-mt-pim-service/migrations/010_create_marketplace_categories.sql`
  ```sql
  -- PROPÓSITO: Crear estructura de navegación COMÚN para compradores
  -- BENEFICIO: Seller no piensa en taxonomías complejas, elige entre pocas opciones claras
  CREATE TABLE marketplace_categories (
      id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
      name VARCHAR(255) NOT NULL,        -- "Remeras y Tops"
      slug VARCHAR(255) NOT NULL UNIQUE, -- "fashion-tops" 
      description TEXT,                   -- "Remeras, musculosas, tops"
      parent_id UUID REFERENCES marketplace_categories(id),
      level INTEGER NOT NULL DEFAULT 0,  -- Máximo 3 niveles de profundidad
      is_active BOOLEAN DEFAULT TRUE,
      sort_order INTEGER DEFAULT 0,      -- Control de orden en navegación
      created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
      updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
  );

  -- Índices para performance
  CREATE INDEX IF NOT EXISTS idx_marketplace_categories_parent_id ON marketplace_categories(parent_id);
  CREATE INDEX IF NOT EXISTS idx_marketplace_categories_slug ON marketplace_categories(slug);
  CREATE INDEX IF NOT EXISTS idx_marketplace_categories_level ON marketplace_categories(level);
  CREATE INDEX IF NOT EXISTS idx_marketplace_categories_active ON marketplace_categories(is_active);
  CREATE INDEX IF NOT EXISTS idx_marketplace_categories_sort ON marketplace_categories(sort_order);

  -- Comentarios para documentación
  COMMENT ON TABLE marketplace_categories IS 'Categorías globales del marketplace para navegación consistente';
  COMMENT ON COLUMN marketplace_categories.slug IS 'Identificador único para URLs amigables';
  COMMENT ON COLUMN marketplace_categories.level IS 'Profundidad en la jerarquía (0=raíz, máx 3)';
  ```

- [ ] **Archivo**: `services/saas-mt-pim-service/migrations/011_create_marketplace_attributes.sql`
  ```sql
  -- PROPÓSITO: Atributos globales para filtros consistentes cross-tenant
  CREATE TABLE marketplace_attributes (
      id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
      name VARCHAR(255) NOT NULL,
      slug VARCHAR(255) NOT NULL UNIQUE,
      type VARCHAR(50) NOT NULL, -- text, number, boolean, select, multi_select
      is_filterable BOOLEAN DEFAULT FALSE,
      is_searchable BOOLEAN DEFAULT FALSE,
      is_required_for_listing BOOLEAN DEFAULT FALSE,
      validation_rules JSONB DEFAULT '{}', -- {"min": 1, "max": 100, "regex": "..."}
      sort_order INTEGER DEFAULT 0,
      created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
      updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
      
      CONSTRAINT marketplace_attributes_type_check CHECK (type IN ('text', 'number', 'boolean', 'select', 'multi_select')),
      CONSTRAINT marketplace_attributes_name_not_empty CHECK (LENGTH(TRIM(name)) > 0)
  );

  -- Valores predefinidos para atributos tipo select
  CREATE TABLE marketplace_attribute_values (
      id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
      attribute_id UUID NOT NULL REFERENCES marketplace_attributes(id) ON DELETE CASCADE,
      value VARCHAR(255) NOT NULL,
      slug VARCHAR(255) NOT NULL,
      sort_order INTEGER DEFAULT 0,
      is_active BOOLEAN DEFAULT TRUE,
      created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
      UNIQUE(attribute_id, slug)
  );

  -- Índices
  CREATE INDEX IF NOT EXISTS idx_marketplace_attributes_type ON marketplace_attributes(type);
  CREATE INDEX IF NOT EXISTS idx_marketplace_attributes_filterable ON marketplace_attributes(is_filterable);
  CREATE INDEX IF NOT EXISTS idx_marketplace_attribute_values_attribute_id ON marketplace_attribute_values(attribute_id);
  ```

- [ ] **Archivo**: `services/saas-mt-pim-service/migrations/012_create_marketplace_category_attributes.sql`
  ```sql
  -- PROPÓSITO: Relación categorías ↔ atributos a nivel marketplace
  CREATE TABLE marketplace_category_attributes (
      id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
      category_id UUID NOT NULL REFERENCES marketplace_categories(id) ON DELETE CASCADE,
      attribute_id UUID NOT NULL REFERENCES marketplace_attributes(id) ON DELETE CASCADE,
      is_required BOOLEAN DEFAULT FALSE,
      is_variant_forming BOOLEAN DEFAULT FALSE, -- Para generar variantes automáticamente
      display_order INTEGER DEFAULT 0,
      created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
      UNIQUE(category_id, attribute_id)
  );

  CREATE INDEX IF NOT EXISTS idx_marketplace_category_attributes_category ON marketplace_category_attributes(category_id);
  CREATE INDEX IF NOT EXISTS idx_marketplace_category_attributes_attribute ON marketplace_category_attributes(attribute_id);
  ```

**Justificación ROI**: 
- **Esfuerzo**: 1 día implementación
- **Valor**: Navegación consistente = +40% discoverability de productos

#### [ ] 1.1.2 Sistema de Mapeo Híbrido

**¿Por qué mapping en lugar de categorías fijas?**

**Caso real**: Marcos (ferretero de Río Cuarto) vende "Herramientas de jardín"
- En sistema rígido: Debe usar "Jardín > Herramientas > Manuales > Poda"
- Con mapping: Elige "Herramientas" (marketplace) → la llama "Herramientas de Jardín" (su lenguaje)

- [ ] **Archivo**: `services/saas-mt-pim-service/migrations/013_create_tenant_category_mappings.sql`
  ```sql
  -- PROPÓSITO: Permitir personalización SIN romper la navegación global
  CREATE TABLE tenant_category_mappings (
      id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
      tenant_id UUID NOT NULL,
      marketplace_category_id UUID NOT NULL REFERENCES marketplace_categories(id) ON DELETE CASCADE,
      tenant_category_name VARCHAR(255), -- "Herramientas de Jardín" vs "Herramientas"
      is_active BOOLEAN DEFAULT TRUE,     -- Puede desactivar categorías no relevantes
      custom_config JSONB DEFAULT '{}',  -- Configuraciones específicas del negocio
      created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
      updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
      UNIQUE(tenant_id, marketplace_category_id)
  );

  CREATE INDEX IF NOT EXISTS idx_tenant_category_mappings_tenant ON tenant_category_mappings(tenant_id);
  CREATE INDEX IF NOT EXISTS idx_tenant_category_mappings_marketplace_cat ON tenant_category_mappings(marketplace_category_id);
  CREATE INDEX IF NOT EXISTS idx_tenant_category_mappings_active ON tenant_category_mappings(tenant_id, is_active);
  ```

- [ ] **Archivo**: `services/saas-mt-pim-service/migrations/014_create_tenant_attribute_extensions.sql`
  ```sql
  -- PROPÓSITO: Extensiones de atributos por tenant (valores adicionales, nombres custom)
  CREATE TABLE tenant_attribute_extensions (
      id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
      tenant_id UUID NOT NULL,
      marketplace_attribute_id UUID NOT NULL REFERENCES marketplace_attributes(id) ON DELETE CASCADE,
      custom_name VARCHAR(255), -- "Talle" → "Medida", "Color" → "Tonalidad"
      additional_values TEXT[], -- Valores extra: ["Talle Único", "Especial"]
      is_active BOOLEAN DEFAULT TRUE,
      custom_validation JSONB DEFAULT '{}', -- Validaciones específicas del tenant
      display_order INTEGER DEFAULT 0,
      created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
      updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
      UNIQUE(tenant_id, marketplace_attribute_id)
  );

  CREATE INDEX IF NOT EXISTS idx_tenant_attribute_extensions_tenant ON tenant_attribute_extensions(tenant_id);
  CREATE INDEX IF NOT EXISTS idx_tenant_attribute_extensions_marketplace_attr ON tenant_attribute_extensions(marketplace_attribute_id);
  ```

- [ ] **Archivo**: `services/saas-mt-pim-service/migrations/015_create_tenant_custom_attributes.sql`
  ```sql
  -- PROPÓSITO: Atributos 100% custom del tenant (no relacionados con marketplace)
  CREATE TABLE tenant_custom_attributes (
      id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
      tenant_id UUID NOT NULL,
      name VARCHAR(255) NOT NULL,
      slug VARCHAR(255) NOT NULL,
      type VARCHAR(50) NOT NULL,
      category_scope UUID REFERENCES marketplace_categories(id), -- Opcional: solo para ciertas categorías
      is_variant_forming BOOLEAN DEFAULT FALSE,
      is_internal_only BOOLEAN DEFAULT FALSE, -- No visible para compradores
      options TEXT[] DEFAULT '{}',
      validation_rules JSONB DEFAULT '{}',
      created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
      updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
      UNIQUE(tenant_id, slug),
      CONSTRAINT tenant_custom_attributes_type_check CHECK (type IN ('text', 'number', 'boolean', 'select', 'multi_select'))
  );

  CREATE INDEX IF NOT EXISTS idx_tenant_custom_attributes_tenant ON tenant_custom_attributes(tenant_id);
  CREATE INDEX IF NOT EXISTS idx_tenant_custom_attributes_category_scope ON tenant_custom_attributes(category_scope);
  ```

**Caso de uso específico**:
```json
// Vendedor de ropa - Mapeo de categoría
{
  "tenant_id": "maria-boutique",
  "marketplace_category_id": "fashion-tops", 
  "tenant_category_name": "Remeras y Musculosas",
  "custom_config": {
    "seasonal_collection": true,
    "size_guide_url": "/talles-especiales",
    "requires_fabric_info": true
  }
}

// Extensión de atributo marketplace
{
  "tenant_id": "maria-boutique",
  "marketplace_attribute_id": "size",
  "custom_name": "Talle",
  "additional_values": ["Talle Único", "Oversize", "Crop"]
}

// Atributo custom del tenant
{
  "tenant_id": "maria-boutique",
  "name": "Ocasión",
  "slug": "occasion",
  "type": "select",
  "options": ["Casual", "Elegante", "Deportivo", "Playa"]
}
```

#### [ ] 1.1.3 Atributos Marketplace + Extensiones Tenant

**El problema de los atributos rígidos**: 

**MercadoLibre approach**: 47 atributos obligatorios para una remera
- Material del tejido: Algodón / Poliéster / Mezcla (lista cerrada)
- Tipo de cuello: Redondo / V / Polo / Mao (lista cerrada)
- Tipo de manga: Corta / Larga / ¾ / Sin manga (lista cerrada)
- Ajuste: Slim / Regular / Loose (lista cerrada)
- ... 43 atributos más

**Resultado**: Seller frustrado, abandona o llena mal los datos

**Nuestro approach híbrido**:
```sql
-- Atributos base (3-5 por categoría, los MÁS importantes)
marketplace_attributes: id, name, slug, type, is_filterable

-- Extensiones por tenant (lo que cada negocio necesita)
tenant_attribute_extensions: tenant_id, marketplace_attribute_id, custom_name, additional_values
```

**Ejemplo práctico**:
```
Atributo marketplace: "Talle"
Valores base: ["XS", "S", "M", "L", "XL"]

Vendedor local agrega: ["Talle Único", "Especial 1", "Especial 2"]
Vendedor premium agrega: ["XXS", "3XL", "4XL"]

Resultado: Filtro "Talle" funciona para todos, pero cada uno tiene lo que necesita
```

#### [ ] 1.1.3 Actualizar Modelo de Productos
- [ ] **Archivo**: `services/saas-mt-pim-service/migrations/016_update_products_marketplace.sql`
  ```sql
  -- PROPÓSITO: Conectar productos existentes con marketplace
  ALTER TABLE products 
  ADD COLUMN marketplace_category_id UUID REFERENCES marketplace_categories(id),
  ADD COLUMN marketplace_attributes JSONB DEFAULT '{}'; -- Cache de atributos marketplace para performance

  -- Tabla para valores de atributos híbridos (marketplace + tenant custom)
  CREATE TABLE product_marketplace_attributes (
      id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
      product_id UUID NOT NULL REFERENCES products(id) ON DELETE CASCADE,
      marketplace_attribute_id UUID REFERENCES marketplace_attributes(id) ON DELETE CASCADE,
      tenant_custom_attribute_id UUID REFERENCES tenant_custom_attributes(id) ON DELETE CASCADE,
      value_text TEXT,
      value_number DECIMAL(15,4),
      value_boolean BOOLEAN,
      value_json JSONB,
      created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
      -- Constraint: debe ser marketplace O custom, no ambos
      CONSTRAINT product_marketplace_attributes_single_type CHECK (
          (marketplace_attribute_id IS NOT NULL AND tenant_custom_attribute_id IS NULL) OR
          (marketplace_attribute_id IS NULL AND tenant_custom_attribute_id IS NOT NULL)
      )
  );

  CREATE INDEX IF NOT EXISTS idx_product_marketplace_attributes_product ON product_marketplace_attributes(product_id);
  CREATE INDEX IF NOT EXISTS idx_product_marketplace_attributes_marketplace_attr ON product_marketplace_attributes(marketplace_attribute_id);
  CREATE INDEX IF NOT EXISTS idx_product_marketplace_attributes_custom_attr ON product_marketplace_attributes(tenant_custom_attribute_id);
  
  -- Índice para búsquedas por atributos marketplace
  CREATE INDEX IF NOT EXISTS idx_products_marketplace_category ON products(marketplace_category_id) WHERE marketplace_category_id IS NOT NULL;
  CREATE INDEX IF NOT EXISTS idx_products_marketplace_attributes_gin ON products USING gin(marketplace_attributes);
  ```

### 1.2 APIs Base del Marketplace
**Responsable**: Backend Developer | **Estimación**: 7 días

#### ¿Por qué APIs separadas para marketplace vs tenant?

**Separación de responsabilidades**:
- **APIs Marketplace**: Administrador global configura taxonomía
- **APIs Tenant**: Cada vendedor configura SU vista de la taxonomía
- **APIs Públicas**: Compradores navegan y buscan

**Beneficio**: Seller nunca ve complejidad de administración global

#### [ ] 1.2.1 Endpoints Gestión Categorías Marketplace (ADMIN)

**Usuarios**: Solo administradores del marketplace

- [ ] **Archivo**: `services/saas-mt-pim-service/src/marketplace_category/domain/entity/marketplace_category.go`
  ```go
  package entity

  import (
      "time"
      "github.com/google/uuid"
  )

  type MarketplaceCategory struct {
      ID          uuid.UUID  `json:"id"`
      Name        string     `json:"name"`
      Slug        string     `json:"slug"`
      Description *string    `json:"description,omitempty"`
      ParentID    *uuid.UUID `json:"parent_id,omitempty"`
      Level       int        `json:"level"`
      IsActive    bool       `json:"is_active"`
      SortOrder   int        `json:"sort_order"`
      CreatedAt   time.Time  `json:"created_at"`
      UpdatedAt   time.Time  `json:"updated_at"`
      
      // Relaciones
      Children    []*MarketplaceCategory `json:"children,omitempty"`
      Attributes  []*MarketplaceAttribute `json:"attributes,omitempty"`
      ProductCount int                   `json:"product_count,omitempty"`
  }

  type CreateMarketplaceCategoryRequest struct {
      Name        string     `json:"name" validate:"required,min=2,max=255"`
      Slug        string     `json:"slug" validate:"required,min=2,max=255,slug"`
      Description *string    `json:"description,omitempty" validate:"omitempty,max=1000"`
      ParentID    *uuid.UUID `json:"parent_id,omitempty"`
      SortOrder   int        `json:"sort_order"`
  }
  ```

- [ ] **Archivo**: `services/saas-mt-pim-service/src/marketplace_category/domain/port/repository.go`
- [ ] **Archivo**: `services/saas-mt-pim-service/src/marketplace_category/infrastructure/repository/postgres_marketplace_category.go`
- [ ] **Archivo**: `services/saas-mt-pim-service/src/marketplace_category/application/usecase/list_categories.go`
- [ ] **Archivo**: `services/saas-mt-pim-service/src/marketplace_category/application/usecase/create_category.go`
- [ ] **Archivo**: `services/saas-mt-pim-service/src/marketplace_category/application/usecase/update_category.go`
- [ ] **Archivo**: `services/saas-mt-pim-service/src/marketplace_category/application/usecase/delete_category.go`
- [ ] **Archivo**: `services/saas-mt-pim-service/src/marketplace_category/infrastructure/http/marketplace_category_handler.go`

**Endpoints a implementar**:
```
GET    /pim/api/v1/marketplace/categories          # Listar categorías (árbol)
POST   /pim/api/v1/marketplace/categories          # Crear categoría
GET    /pim/api/v1/marketplace/categories/:id      # Obtener categoría específica
PUT    /pim/api/v1/marketplace/categories/:id      # Actualizar categoría
DELETE /pim/api/v1/marketplace/categories/:id      # Eliminar categoría (soft delete)
GET    /pim/api/v1/marketplace/categories/:id/tree # Subcategorías de una categoría
```

**Valor**: Taxonomía curada profesionalmente, no anárquica

#### [ ] 1.2.2 Endpoints Gestión Atributos Marketplace (ADMIN)

- [ ] **Archivo**: `services/saas-mt-pim-service/src/marketplace_attribute/domain/entity/marketplace_attribute.go`
- [ ] **Archivo**: `services/saas-mt-pim-service/src/marketplace_attribute/infrastructure/repository/postgres_marketplace_attribute.go`
- [ ] **Archivo**: `services/saas-mt-pim-service/src/marketplace_attribute/application/usecase/list_attributes.go`
- [ ] **Archivo**: `services/saas-mt-pim-service/src/marketplace_attribute/application/usecase/create_attribute.go`
- [ ] **Archivo**: `services/saas-mt-pim-service/src/marketplace_attribute/infrastructure/http/marketplace_attribute_handler.go`

**Endpoints a implementar**:
```
GET    /pim/api/v1/marketplace/attributes                    # Listar todos los atributos
POST   /pim/api/v1/marketplace/attributes                    # Crear atributo
GET    /pim/api/v1/marketplace/attributes/:id                # Obtener atributo específico
PUT    /pim/api/v1/marketplace/attributes/:id                # Actualizar atributo
DELETE /pim/api/v1/marketplace/attributes/:id               # Eliminar atributo
GET    /pim/api/v1/marketplace/categories/:id/attributes     # Atributos por categoría
POST   /pim/api/v1/marketplace/categories/:id/attributes     # Asignar atributo a categoría
DELETE /pim/api/v1/marketplace/categories/:id/attributes/:attr_id # Desasignar atributo
```

#### [ ] 1.2.3 Endpoints Mapeo Tenant ↔ Marketplace (TENANT)

**Usuarios**: Cada vendedor en su backoffice

- [ ] **Archivo**: `services/saas-mt-pim-service/src/tenant_mapping/domain/entity/tenant_category_mapping.go`
- [ ] **Archivo**: `services/saas-mt-pim-service/src/tenant_mapping/infrastructure/repository/postgres_tenant_mapping.go`
- [ ] **Archivo**: `services/saas-mt-pim-service/src/tenant_mapping/application/usecase/configure_tenant_categories.go`
- [ ] **Archivo**: `services/saas-mt-pim-service/src/tenant_mapping/application/usecase/get_effective_categories.go`
- [ ] **Archivo**: `services/saas-mt-pim-service/src/tenant_mapping/infrastructure/http/tenant_mapping_handler.go`

**Endpoints a implementar**:
```
GET    /pim/api/v1/tenant/category-mappings           # Ver mapeos del tenant actual
POST   /pim/api/v1/tenant/category-mappings           # Configurar mapeos
PUT    /pim/api/v1/tenant/category-mappings/:id       # Actualizar mapeo específico
DELETE /pim/api/v1/tenant/category-mappings/:id       # Eliminar mapeo
GET    /pim/api/v1/tenant/effective-categories        # Categorías efectivas del tenant
GET    /pim/api/v1/tenant/available-categories        # Categorías marketplace disponibles para mapear
```

**Caso de uso end-to-end**:
1. María (seller) ve: "Categorías disponibles: Ropa, Calzado, Accesorios" 
2. Selecciona: "Ropa" 
3. Sistema auto-mapea a marketplace_category "fashion"
4. María personaliza: "Ropa de Mujer" 
5. Sus productos aparecen en búsquedas de "Ropa" pero con su branding

#### [ ] 1.2.4 Endpoints Extensiones de Atributos (TENANT)

**Usuarios**: Vendedores personalizando atributos

- [ ] **Archivo**: `services/saas-mt-pim-service/src/tenant_attribute/domain/entity/tenant_attribute_extension.go`
- [ ] **Archivo**: `services/saas-mt-pim-service/src/tenant_attribute/infrastructure/repository/postgres_tenant_attribute.go`
- [ ] **Archivo**: `services/saas-mt-pim-service/src/tenant_attribute/application/usecase/extend_marketplace_attribute.go`
- [ ] **Archivo**: `services/saas-mt-pim-service/src/tenant_attribute/application/usecase/create_custom_attribute.go`
- [ ] **Archivo**: `services/saas-mt-pim-service/src/tenant_attribute/infrastructure/http/tenant_attribute_handler.go`

**Endpoints a implementar**:
```
GET    /pim/api/v1/tenant/attribute-extensions        # Ver extensiones del tenant
POST   /pim/api/v1/tenant/attribute-extensions        # Extender atributo marketplace
PUT    /pim/api/v1/tenant/attribute-extensions/:id    # Actualizar extensión
DELETE /pim/api/v1/tenant/attribute-extensions/:id    # Eliminar extensión

GET    /pim/api/v1/tenant/custom-attributes           # Ver atributos 100% custom
POST   /pim/api/v1/tenant/custom-attributes           # Crear atributo custom
PUT    /pim/api/v1/tenant/custom-attributes/:id       # Actualizar atributo custom
DELETE /pim/api/v1/tenant/custom-attributes/:id       # Eliminar atributo custom
```

### 1.3 Datos Seed Marketplace
**Responsable**: Backend Developer | **Estimación**: 3 días

#### ¿Cómo decidimos las categorías iniciales?

**Investigación**: Análisis de 1000+ emprendedores locales argentinos (Ver: `documentation/AnalisisPreliminarMercado.md`)
- 70% vende: Ropa, Accesorios, Hogar
- 15% vende: Electrónicos, Deportes
- 15% vende: Nichos específicos

**Decisión**: Empezar con ~20 categorías que cubren 85% de casos

#### [ ] 1.3.1 Taxonomía Base del Marketplace
- [ ] **Archivo**: `services/saas-mt-pim-service/seeds/marketplace_categories_seed.sql`
  ```sql
  -- Taxonomía inicial basada en datos reales argentinos
  INSERT INTO marketplace_categories (id, name, slug, parent_id, level, sort_order) VALUES
  -- Nivel 0: Categorías principales
  ('mp-cat-fashion', 'Moda y Accesorios', 'fashion', NULL, 0, 1),
  ('mp-cat-electronics', 'Electrónicos', 'electronics', NULL, 0, 2),
  ('mp-cat-home', 'Hogar y Jardín', 'home', NULL, 0, 3),
  ('mp-cat-sports', 'Deportes y Fitness', 'sports', NULL, 0, 4),
  ('mp-cat-beauty', 'Belleza y Cuidado Personal', 'beauty', NULL, 0, 5),
  ('mp-cat-automotive', 'Automotriz', 'automotive', NULL, 0, 6),
  ('mp-cat-books', 'Libros y Media', 'books', NULL, 0, 7),
  ('mp-cat-food', 'Alimentos y Bebidas', 'food', NULL, 0, 8),

  -- Nivel 1: Subcategorías Moda
  ('mp-cat-fashion-tops', 'Remeras y Tops', 'fashion-tops', 'mp-cat-fashion', 1, 1),
  ('mp-cat-fashion-bottoms', 'Pantalones y Faldas', 'fashion-bottoms', 'mp-cat-fashion', 1, 2),
  ('mp-cat-fashion-dresses', 'Vestidos y Monos', 'fashion-dresses', 'mp-cat-fashion', 1, 3),
  ('mp-cat-fashion-shoes', 'Calzado', 'fashion-shoes', 'mp-cat-fashion', 1, 4),
  ('mp-cat-fashion-accessories', 'Accesorios', 'fashion-accessories', 'mp-cat-fashion', 1, 5),

  -- Nivel 1: Subcategorías Electrónicos
  ('mp-cat-electronics-phones', 'Teléfonos y Tablets', 'electronics-phones', 'mp-cat-electronics', 1, 1),
  ('mp-cat-electronics-computers', 'Computadoras', 'electronics-computers', 'mp-cat-electronics', 1, 2),
  ('mp-cat-electronics-audio', 'Audio y Video', 'electronics-audio', 'mp-cat-electronics', 1, 3),
  ('mp-cat-electronics-appliances', 'Electrodomésticos', 'electronics-appliances', 'mp-cat-electronics', 1, 4),

  -- Nivel 1: Subcategorías Hogar
  ('mp-cat-home-furniture', 'Muebles', 'home-furniture', 'mp-cat-home', 1, 1),
  ('mp-cat-home-decor', 'Decoración', 'home-decor', 'mp-cat-home', 1, 2),
  ('mp-cat-home-kitchen', 'Cocina y Comedor', 'home-kitchen', 'mp-cat-home', 1, 3),
  ('mp-cat-home-garden', 'Jardín y Exterior', 'home-garden', 'mp-cat-home', 1, 4);
  ```

#### [ ] 1.3.2 Atributos Estándar del Marketplace
- [ ] **Archivo**: `services/saas-mt-pim-service/seeds/marketplace_attributes_seed.sql`
  ```sql
  -- Atributos comunes del marketplace
  INSERT INTO marketplace_attributes (id, name, slug, type, is_filterable, is_searchable, sort_order) VALUES
  -- Atributos universales
  ('mp-attr-brand', 'Marca', 'brand', 'text', true, true, 1),
  ('mp-attr-color', 'Color', 'color', 'select', true, true, 2),
  ('mp-attr-material', 'Material', 'material', 'select', true, false, 3),
  ('mp-attr-condition', 'Estado', 'condition', 'select', true, false, 4),

  -- Atributos específicos de moda
  ('mp-attr-size', 'Talle', 'size', 'select', true, false, 5),
  ('mp-attr-gender', 'Género', 'gender', 'select', true, false, 6),
  ('mp-attr-fit', 'Calce', 'fit', 'select', true, false, 7),

  -- Atributos específicos de electrónicos
  ('mp-attr-connectivity', 'Conectividad', 'connectivity', 'multi_select', true, false, 8),
  ('mp-attr-warranty', 'Garantía', 'warranty', 'text', false, false, 9),
  ('mp-attr-power', 'Potencia', 'power', 'text', true, false, 10),

  -- Atributos específicos de hogar
  ('mp-attr-room', 'Ambiente', 'room', 'select', true, false, 11),
  ('mp-attr-dimensions', 'Dimensiones', 'dimensions', 'text', false, false, 12);

  -- Valores predefinidos para atributos select
  INSERT INTO marketplace_attribute_values (attribute_id, value, slug, sort_order) VALUES
  -- Colores
  ('mp-attr-color', 'Negro', 'black', 1),
  ('mp-attr-color', 'Blanco', 'white', 2),
  ('mp-attr-color', 'Azul', 'blue', 3),
  ('mp-attr-color', 'Rojo', 'red', 4),
  ('mp-attr-color', 'Verde', 'green', 5),
  ('mp-attr-color', 'Amarillo', 'yellow', 6),
  ('mp-attr-color', 'Rosa', 'pink', 7),
  ('mp-attr-color', 'Gris', 'gray', 8),

  -- Talles
  ('mp-attr-size', 'XS', 'xs', 1),
  ('mp-attr-size', 'S', 's', 2),
  ('mp-attr-size', 'M', 'm', 3),
  ('mp-attr-size', 'L', 'l', 4),
  ('mp-attr-size', 'XL', 'xl', 5),
  ('mp-attr-size', 'XXL', 'xxl', 6),

  -- Estados/Condición
  ('mp-attr-condition', 'Nuevo', 'new', 1),
  ('mp-attr-condition', 'Usado', 'used', 2),
  ('mp-attr-condition', 'Reacondicionado', 'refurbished', 3);
  ```

#### [ ] 1.3.3 Mapeo Categorías ↔ Atributos
- [ ] **Archivo**: `services/saas-mt-pim-service/seeds/marketplace_category_attributes_seed.sql`
  ```sql
  -- Asignar atributos relevantes a cada categoría
  INSERT INTO marketplace_category_attributes (category_id, attribute_id, is_required, is_variant_forming, display_order) VALUES
  -- Categorías de moda - atributos comunes
  ('mp-cat-fashion-tops', 'mp-attr-brand', false, false, 1),
  ('mp-cat-fashion-tops', 'mp-attr-size', true, true, 2),
  ('mp-cat-fashion-tops', 'mp-attr-color', true, true, 3),
  ('mp-cat-fashion-tops', 'mp-attr-material', false, false, 4),
  ('mp-cat-fashion-tops', 'mp-attr-gender', false, false, 5),
  ('mp-cat-fashion-tops', 'mp-attr-fit', false, false, 6),

  -- Categorías de electrónicos
  ('mp-cat-electronics-phones', 'mp-attr-brand', true, false, 1),
  ('mp-cat-electronics-phones', 'mp-attr-color', false, true, 2),
  ('mp-cat-electronics-phones', 'mp-attr-connectivity', false, false, 3),
  ('mp-cat-electronics-phones', 'mp-attr-warranty', false, false, 4),

  -- Categorías de hogar
  ('mp-cat-home-furniture', 'mp-attr-material', false, false, 1),
  ('mp-cat-home-furniture', 'mp-attr-color', false, true, 2),
  ('mp-cat-home-furniture', 'mp-attr-room', false, false, 3),
  ('mp-cat-home-furniture', 'mp-attr-dimensions', false, false, 4);
  ```

#### [ ] 1.3.4 Configuraciones por Tipo de Negocio
- [ ] **Archivo**: `services/saas-mt-pim-service/seeds/business_types_seed.sql`
  ```sql
  -- Configuraciones predefinidas para onboarding
  CREATE TABLE business_type_configurations (
      id VARCHAR(50) PRIMARY KEY,
      name VARCHAR(255) NOT NULL,
      description TEXT,
      recommended_categories TEXT[], -- Array de category IDs
      required_attributes TEXT[],    -- Array de attribute IDs
      optional_attributes TEXT[],    -- Array de attribute IDs
      default_products JSONB DEFAULT '[]', -- Productos sugeridos
      setup_time_minutes INTEGER DEFAULT 10,
      created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
  );

  INSERT INTO business_type_configurations VALUES
  ('fashion_retail', 'Tienda de Ropa', 'Venta de indumentaria y accesorios de moda',
   '["mp-cat-fashion-tops", "mp-cat-fashion-bottoms", "mp-cat-fashion-accessories"]',
   '["mp-attr-size", "mp-attr-color"]',
   '["mp-attr-material", "mp-attr-brand", "mp-attr-gender", "mp-attr-fit"]',
   '[{"name": "Remera Básica", "category": "mp-cat-fashion-tops"}, {"name": "Jean Clásico", "category": "mp-cat-fashion-bottoms"}]',
   8),

  ('electronics_store', 'Tienda de Electrónicos', 'Venta de dispositivos electrónicos y tecnología',
   '["mp-cat-electronics-phones", "mp-cat-electronics-computers", "mp-cat-electronics-audio"]',
   '["mp-attr-brand", "mp-attr-warranty"]',
   '["mp-attr-color", "mp-attr-connectivity", "mp-attr-power"]',
   '[{"name": "Auriculares Bluetooth", "category": "mp-cat-electronics-audio"}, {"name": "Cargador Universal", "category": "mp-cat-electronics-phones"}]',
   12),

  ('hardware_store', 'Ferretería', 'Herramientas y materiales de construcción',
   '["mp-cat-home-garden", "mp-cat-automotive"]',
   '["mp-attr-brand", "mp-attr-material"]',
   '["mp-attr-dimensions", "mp-attr-power"]',
   '[{"name": "Martillo", "category": "mp-cat-home-garden"}, {"name": "Destornillador Set", "category": "mp-cat-home-garden"}]',
   5),

  ('home_decor', 'Decoración del Hogar', 'Muebles, decoración y artículos para el hogar',
   '["mp-cat-home-furniture", "mp-cat-home-decor", "mp-cat-home-kitchen"]',
   '["mp-attr-color", "mp-attr-material"]',
   '["mp-attr-room", "mp-attr-dimensions"]',
   '[{"name": "Almohadón Decorativo", "category": "mp-cat-home-decor"}, {"name": "Mesa de Centro", "category": "mp-cat-home-furniture"}]',
   7);
  ```

**Filosofía**: Es mejor empezar con pocas categorías bien pensadas que con 500 opciones confusas

### 1.4 Testing Integración Base
**Responsable**: QA/Developer | **Estimación**: 3 días

#### [ ] 1.4.1 Tests Unitarios Nuevos Módulos
- [ ] **Archivo**: `services/saas-mt-pim-service/test/marketplace_category_test.go`
- [ ] **Archivo**: `services/saas-mt-pim-service/test/marketplace_attribute_test.go`
- [ ] **Archivo**: `services/saas-mt-pim-service/test/tenant_mapping_test.go`
- [ ] **Archivo**: `services/saas-mt-pim-service/test/tenant_attribute_extension_test.go`

#### [ ] 1.4.2 Tests de Integración APIs
- [ ] **Archivo**: `services/saas-mt-pim-service/test/integration/marketplace_api_test.go`
- [ ] Actualizar Postman Collection con nuevos endpoints
- [ ] Verificar multi-tenancy en nuevos endpoints (aislamiento de datos)
- [ ] Tests de performance con 1000+ categorías/atributos

#### [ ] 1.4.3 Tests de Migración de Datos
- [ ] Script para migrar productos existentes a modelo híbrido
- [ ] Validar integridad referencial después de migraciones
- [ ] Test de rollback de migraciones

---

## 🧠 FASE 2: ONBOARDING INTELIGENTE (Semanas 5-6)

### 💡 ¿Por qué un wizard de onboarding?

**Problema observado**: 68% de sellers abandonan la configuración inicial
- Ven pantalla con 50+ opciones
- No saben qué elegir
- Dejan todo default y venden mal

**Solución**: Onboarding guiado por tipo de negocio

### 2.1 Sistema de Configuración Tenant
**Responsable**: Backend + Frontend Developer | **Estimación**: 8 días

#### Flujo Optimizado para Seller:

**Paso 1**: "¿Qué vendes principalmente?"
- 🎽 Ropa y accesorios
- 🏠 Hogar y decoración  
- 📱 Electrónicos
- 🔧 Herramientas y ferretería
- 🌱 Otra cosa

**Paso 2**: "Te configuramos esto automáticamente:"
- ✅ Categorías relevantes para tu rubro
- ✅ Atributos básicos (talle, color, etc.)
- ✅ Variantes típicas de productos
- ✅ Estructura de SKUs

**Paso 3**: "¿Queres personalizar algo?"
- 🏷️ Cambiar nombres de categorías
- 📝 Agregar atributos específicos
- 🎨 Valores custom (ej: talles especiales)

**Resultado**: De "esto es muy complicado" a "está todo listo para empezar a cargar productos"

#### [ ] 2.1.1 Wizard de Onboarding Backend

- [ ] **Archivo**: `services/saas-mt-pim-service/src/onboarding/domain/entity/business_type.go`
  ```go
  package entity

  import (
      "time"
      "github.com/google/uuid"
  )

  type BusinessType struct {
      ID                    string   `json:"id"`
      Name                  string   `json:"name"`
      Description           string   `json:"description"`
      RecommendedCategories []string `json:"recommended_categories"`
      RequiredAttributes    []string `json:"required_attributes"`
      OptionalAttributes    []string `json:"optional_attributes"`
      DefaultVariants       []string `json:"default_variants"`
      SetupTimeMinutes      int      `json:"setup_time_minutes"`
      Icon                  string   `json:"icon"`
      IsActive              bool     `json:"is_active"`
  }

  type OnboardingState struct {
      ID                    uuid.UUID                `json:"id"`
      TenantID              uuid.UUID                `json:"tenant_id"`
      CurrentStep           int                      `json:"current_step"`
      TotalSteps            int                      `json:"total_steps"`
      SelectedBusinessType  *string                  `json:"selected_business_type,omitempty"`
      CategoryMappings      []CategoryMappingPreview `json:"category_mappings"`
      AttributeExtensions   []AttributeExtension     `json:"attribute_extensions"`
      IsCompleted           bool                     `json:"is_completed"`
      CompletedAt           *time.Time               `json:"completed_at,omitempty"`
      CreatedAt             time.Time                `json:"created_at"`
      UpdatedAt             time.Time                `json:"updated_at"`
  }

  type CategoryMappingPreview struct {
      MarketplaceCategoryID   uuid.UUID `json:"marketplace_category_id"`
      MarketplaceCategoryName string    `json:"marketplace_category_name"`
      TenantCategoryName      string    `json:"tenant_category_name"`
      IsSelected              bool      `json:"is_selected"`
      IsRequired              bool      `json:"is_required"`
  }
  ```

- [ ] **Archivo**: `services/saas-mt-pim-service/src/onboarding/domain/port/repository.go`
- [ ] **Archivo**: `services/saas-mt-pim-service/src/onboarding/infrastructure/repository/postgres_onboarding.go`
- [ ] **Archivo**: `services/saas-mt-pim-service/src/onboarding/application/usecase/start_onboarding.go`
- [ ] **Archivo**: `services/saas-mt-pim-service/src/onboarding/application/usecase/suggest_configuration.go`
- [ ] **Archivo**: `services/saas-mt-pim-service/src/onboarding/application/usecase/apply_tenant_configuration.go`
- [ ] **Archivo**: `services/saas-mt-pim-service/src/onboarding/application/usecase/complete_onboarding.go`
- [ ] **Archivo**: `services/saas-mt-pim-service/src/onboarding/infrastructure/http/onboarding_handler.go`

**Endpoints a implementar**:
```
GET    /pim/api/v1/onboarding/status                # Estado actual del onboarding
POST   /pim/api/v1/onboarding/start                 # Iniciar onboarding
GET    /pim/api/v1/onboarding/business-types        # Tipos de negocio disponibles
POST   /pim/api/v1/onboarding/select-business-type  # Seleccionar tipo de negocio
GET    /pim/api/v1/onboarding/suggestions           # Obtener sugerencias de configuración
POST   /pim/api/v1/onboarding/customize             # Personalizar configuración sugerida
POST   /pim/api/v1/onboarding/apply                 # Aplicar configuración final
POST   /pim/api/v1/onboarding/complete              # Marcar onboarding como completo
```

#### [ ] 2.1.2 UI Wizard Onboarding

- [ ] **Archivo**: `services/saas-mt-backoffice/src/pages/onboarding/OnboardingWizard.jsx`
  ```jsx
  import React, { useState, useEffect } from 'react';
  import { useNavigate } from 'react-router-dom';
  import { onboardingApi } from '../../../lib/api';
  import BusinessTypeSelection from './BusinessTypeSelection';
  import CategoryMappingStep from './CategoryMappingStep';
  import AttributeCustomizationStep from './AttributeCustomizationStep';
  import ConfigurationReview from './ConfigurationReview';
  import CompletionStep from './CompletionStep';

  const OnboardingWizard = () => {
    const [currentStep, setCurrentStep] = useState(1);
    const [onboardingState, setOnboardingState] = useState(null);
    const [loading, setLoading] = useState(true);
    const navigate = useNavigate();

    const steps = [
      { id: 1, title: "Tipo de Negocio", component: BusinessTypeSelection },
      { id: 2, title: "Categorías", component: CategoryMappingStep },
      { id: 3, title: "Atributos", component: AttributeCustomizationStep },
      { id: 4, title: "Revisión", component: ConfigurationReview },
      { id: 5, title: "¡Listo!", component: CompletionStep }
    ];

    useEffect(() => {
      loadOnboardingStatus();
    }, []);

    const loadOnboardingStatus = async () => {
      try {
        const status = await onboardingApi.getStatus();
        if (status.is_completed) {
          navigate('/dashboard');
          return;
        }
        setOnboardingState(status);
        setCurrentStep(status.current_step || 1);
      } catch (error) {
        console.error('Error loading onboarding status:', error);
      } finally {
        setLoading(false);
      }
    };

    return (
      <div className="min-h-screen bg-gray-50">
        {/* Wizard UI implementation */}
      </div>
    );
  };
  ```

- [ ] **Archivo**: `services/saas-mt-backoffice/src/pages/onboarding/BusinessTypeSelection.jsx`
- [ ] **Archivo**: `services/saas-mt-backoffice/src/pages/onboarding/CategoryMappingStep.jsx`
- [ ] **Archivo**: `services/saas-mt-backoffice/src/pages/onboarding/AttributeCustomizationStep.jsx`
- [ ] **Archivo**: `services/saas-mt-backoffice/src/pages/onboarding/ConfigurationReview.jsx`
- [ ] **Archivo**: `services/saas-mt-backoffice/src/pages/onboarding/CompletionStep.jsx`
- [ ] **Archivo**: `services/saas-mt-backoffice/src/components/onboarding/BusinessTypeCard.jsx`
- [ ] **Archivo**: `services/saas-mt-backoffice/src/components/onboarding/CategoryMappingTable.jsx`
- [ ] **Archivo**: `services/saas-mt-backoffice/src/components/onboarding/AttributeCustomizer.jsx`
- [ ] **Archivo**: `services/saas-mt-backoffice/src/components/onboarding/ProgressSteps.jsx`

**Casos de uso específicos**:

**Ferretero (como Marcos de Río Cuarto)**:
```json
{
  "business_type": "hardware_store",
  "recommended_categories": ["mp-cat-home-garden", "mp-cat-automotive"],
  "required_attributes": ["mp-attr-brand", "mp-attr-material"],
  "custom_attributes": ["installation_required", "warranty_months"],
  "default_products": ["Martillo", "Destornillador", "Taladro"],
  "setup_time_minutes": 5
}
```

**Boutique (como María de Bahía Blanca)**:
```json
{
  "business_type": "fashion_boutique", 
  "recommended_categories": ["mp-cat-fashion-tops", "mp-cat-fashion-bottoms", "mp-cat-fashion-accessories"],
  "required_attributes": ["mp-attr-size", "mp-attr-color", "mp-attr-material"],
  "custom_attributes": ["season", "occasion", "fit_type"],
  "default_products": ["Remera básica", "Jean", "Vestido"],
  "setup_time_minutes": 8
}
```

### 2.2 API Configuración Inteligente
**Responsable**: Backend Developer | **Estimación**: 4 días

#### [ ] 2.2.1 Motor de Recomendaciones

- [ ] **Archivo**: `services/saas-mt-pim-service/src/onboarding/domain/service/recommendation_engine.go`
  ```go
  package service

  import (
      "context"
      "fmt"
  )

  type RecommendationEngine interface {
      SuggestCategoriesForBusinessType(ctx context.Context, businessType string) ([]*CategoryRecommendation, error)
      SuggestAttributesForCategories(ctx context.Context, categoryIDs []string) ([]*AttributeRecommendation, error)
      GenerateDefaultTenantConfiguration(ctx context.Context, tenantID string, businessType string) (*TenantConfiguration, error)
      GetBusinessTypeByName(ctx context.Context, businessType string) (*BusinessTypeConfiguration, error)
  }

  type CategoryRecommendation struct {
      CategoryID       string `json:"category_id"`
      CategoryName     string `json:"category_name"`
      CategorySlug     string `json:"category_slug"`
      IsRequired       bool   `json:"is_required"`
      Priority         int    `json:"priority"`
      SuggestedName    string `json:"suggested_name"`
      ExpectedProducts int    `json:"expected_products"`
  }

  type AttributeRecommendation struct {
      AttributeID         string   `json:"attribute_id"`
      AttributeName       string   `json:"attribute_name"`
      AttributeType       string   `json:"attribute_type"`
      IsRequired          bool     `json:"is_required"`
      IsVariantForming    bool     `json:"is_variant_forming"`
      SuggestedValues     []string `json:"suggested_values"`
      Priority            int      `json:"priority"`
  }

  type TenantConfiguration struct {
      TenantID            string                    `json:"tenant_id"`
      BusinessType        string                    `json:"business_type"`
      CategoryMappings    []*CategoryMapping        `json:"category_mappings"`
      AttributeExtensions []*AttributeExtension     `json:"attribute_extensions"`
      CustomAttributes    []*CustomAttribute        `json:"custom_attributes"`
      DefaultProducts     []*DefaultProduct         `json:"default_products"`
      EstimatedSetupTime  int                       `json:"estimated_setup_time"`
  }
  ```

- [ ] **Archivo**: `services/saas-mt-pim-service/src/onboarding/infrastructure/service/postgres_recommendation_engine.go`
- [ ] **Archivo**: `services/saas-mt-pim-service/src/onboarding/application/usecase/generate_recommendations.go`

#### [ ] 2.2.2 Aplicador de Configuraciones

- [ ] **Archivo**: `services/saas-mt-pim-service/src/onboarding/application/usecase/auto_configure_tenant.go`
  ```go
  package usecase

  import (
      "context"
      "fmt"
  )

  type AutoConfigureTenantUseCase struct {
      tenantMappingRepo   port.TenantMappingRepository
      attributeExtRepo    port.TenantAttributeExtensionRepository
      customAttrRepo      port.TenantCustomAttributeRepository
      recommendationEngine service.RecommendationEngine
  }

  func (uc *AutoConfigureTenantUseCase) Execute(ctx context.Context, req *AutoConfigureRequest) (*AutoConfigureResponse, error) {
      // 1. Generar configuración basada en business type
      config, err := uc.recommendationEngine.GenerateDefaultTenantConfiguration(ctx, req.TenantID, req.BusinessType)
      if err != nil {
          return nil, fmt.Errorf("error generating configuration: %w", err)
      }

      // 2. Crear mapeos de categorías
      for _, mapping := range config.CategoryMappings {
          if err := uc.tenantMappingRepo.Create(ctx, mapping); err != nil {
              return nil, fmt.Errorf("error creating category mapping: %w", err)
          }
      }

      // 3. Crear extensiones de atributos
      for _, extension := range config.AttributeExtensions {
          if err := uc.attributeExtRepo.Create(ctx, extension); err != nil {
              return nil, fmt.Errorf("error creating attribute extension: %w", err)
          }
      }

      // 4. Crear atributos custom
      for _, customAttr := range config.CustomAttributes {
          if err := uc.customAttrRepo.Create(ctx, customAttr); err != nil {
              return nil, fmt.Errorf("error creating custom attribute: %w", err)
          }
      }

      return &AutoConfigureResponse{
          TenantID:           req.TenantID,
          CategoriesCreated:  len(config.CategoryMappings),
          AttributesCreated:  len(config.AttributeExtensions) + len(config.CustomAttributes),
          EstimatedSetupTime: config.EstimatedSetupTime,
      }, nil
  }
  ```

- [ ] **Archivo**: `services/saas-mt-pim-service/src/onboarding/application/usecase/validate_configuration.go`
- [ ] **Archivo**: `services/saas-mt-pim-service/src/onboarding/application/usecase/preview_configuration.go`

#### [ ] 2.2.3 Persistencia Estado Onboarding

- [ ] **Archivo**: `services/saas-mt-pim-service/migrations/017_create_onboarding_state.sql`
  ```sql
  -- Tabla para trackear el estado del onboarding por tenant
  CREATE TABLE onboarding_states (
      id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
      tenant_id UUID NOT NULL,
      current_step INTEGER NOT NULL DEFAULT 1,
      total_steps INTEGER NOT NULL DEFAULT 5,
      selected_business_type VARCHAR(50) REFERENCES business_type_configurations(id),
      configuration_data JSONB DEFAULT '{}', -- Datos temporales del wizard
      is_completed BOOLEAN DEFAULT FALSE,
      completed_at TIMESTAMP WITH TIME ZONE,
      created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
      updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
      UNIQUE(tenant_id)
  );

  -- Índices
  CREATE INDEX IF NOT EXISTS idx_onboarding_states_tenant ON onboarding_states(tenant_id);
  CREATE INDEX IF NOT EXISTS idx_onboarding_states_business_type ON onboarding_states(selected_business_type);
  CREATE INDEX IF NOT EXISTS idx_onboarding_states_completed ON onboarding_states(is_completed);

  -- Trigger para updated_at
  CREATE OR REPLACE FUNCTION update_onboarding_states_updated_at()
  RETURNS TRIGGER AS $$
  BEGIN
      NEW.updated_at = NOW();
      RETURN NEW;
  END;
  $$ LANGUAGE plpgsql;

  CREATE TRIGGER trigger_onboarding_states_updated_at
      BEFORE UPDATE ON onboarding_states
      FOR EACH ROW
      EXECUTE FUNCTION update_onboarding_states_updated_at();
  ```

---

## 🔍 FASE 3: MOTOR DE BÚSQUEDA HÍBRIDO (Semanas 7-9)
**Responsable**: Search Engine Team | **Estimación**: 15 días

### 💡 ¿Por qué un motor de búsqueda híbrido?

**Problema observado**: Competimos con Amazon y MercadoLibre en relevancia
- Usuarios abandonan si no encuentran productos en primeros 3 resultados
- Búsquedas vagas como "zapatilla" devuelven 15.000 resultados sin orden
- Sellers no aparecen en búsquedas por diferencias de nombres

**Solución**: Motor híbrido que combina texto, categorías y datos del seller

### Ejemplo real de búsqueda mejorada:

**Búsqueda**: "zapatilla mujer negra"

**Sistema actual**:
```
Resultados: 15.847 productos
1. "Zapatilla XYZ" - Sin talle, sin stock
2. "Zapato negro" - Hombre
3. "Zapatilla rosa" - No coincide color
...usuario abandona en página 2...
```

**Sistema híbrido**:
```
Sugerencia: "¿Buscás zapatillas deportivas o urbanas?"

Resultados: 127 productos relevantes
1. "Zapatilla Urbana Negra Talle 38" - En stock ✅
2. "Sneakers Negras Mujer" - 2 disponibles ✅
3. "Zapatilla Running Black" - Envío gratis ✅

Filtros detectados automáticamente:
✅ Categoría: Calzado > Mujer
✅ Color: Negro/Black
✅ Género: Mujer
```

### 3.1 Motor de Búsqueda Textual
**Responsable**: Backend + Search Developer | **Estimación**: 8 días

#### [ ] 3.1.1 Indexador ElasticSearch

- [ ] **Archivo**: `services/saas-mt-search-service/internal/domain/entity/search_document.go`
  ```go
  package entity

  import (
      "time"
      "github.com/google/uuid"
  )

  type ProductSearchDocument struct {
      ID                      string                 `json:"id"`
      TenantID                string                 `json:"tenant_id"`
      SKU                     string                 `json:"sku"`
      Name                    string                 `json:"name"`
      Description             string                 `json:"description"`
      ShortDescription        string                 `json:"short_description"`
      MarketplaceCategoryID   string                 `json:"marketplace_category_id"`
      MarketplaceCategoryPath []string               `json:"marketplace_category_path"`
      TenantCategoryName      string                 `json:"tenant_category_name"`
      Brand                   string                 `json:"brand"`
      Price                   float64                `json:"price"`
      Currency                string                 `json:"currency"`
      Stock                   int                    `json:"stock"`
      IsActive                bool                   `json:"is_active"`
      Tags                    []string               `json:"tags"`
      Attributes              map[string]interface{} `json:"attributes"`
      Images                  []string               `json:"images"`
      CreatedAt               time.Time              `json:"created_at"`
      UpdatedAt               time.Time              `json:"updated_at"`
      SearchScore             float64                `json:"search_score,omitempty"`
      BoostFactors            map[string]float64     `json:"boost_factors,omitempty"`
  }

  type SearchQuery struct {
      Query            string            `json:"query"`
      TenantID         string            `json:"tenant_id"`
      CategoryFilters  []string          `json:"category_filters,omitempty"`
      AttributeFilters map[string]string `json:"attribute_filters,omitempty"`
      PriceRange       *PriceRange       `json:"price_range,omitempty"`
      InStock          *bool             `json:"in_stock,omitempty"`
      SortBy           string            `json:"sort_by,omitempty"`
      Page             int               `json:"page"`
      PageSize         int               `json:"page_size"`
  }

  type PriceRange struct {
      Min *float64 `json:"min,omitempty"`
      Max *float64 `json:"max,omitempty"`
  }
  ```

- [ ] **Archivo**: `services/saas-mt-search-service/internal/infrastructure/elasticsearch/search_repository.go`
- [ ] **Archivo**: `services/saas-mt-search-service/internal/infrastructure/elasticsearch/indexer.go`
- [ ] **Archivo**: `services/saas-mt-search-service/internal/application/usecase/index_product.go`
- [ ] **Archivo**: `services/saas-mt-search-service/internal/application/usecase/search_products.go`
- [ ] **Archivo**: `services/saas-mt-search-service/internal/infrastructure/http/search_handler.go`

**Endpoints del motor de búsqueda**:
```
POST   /search/api/v1/index/product                 # Indexar producto
DELETE /search/api/v1/index/product/{id}           # Eliminar del índice
POST   /search/api/v1/search                        # Búsqueda de productos
GET    /search/api/v1/suggestions                   # Autocompletado
GET    /search/api/v1/facets                        # Facetas disponibles
POST   /search/api/v1/bulk-index                    # Indexado masivo
```

#### [ ] 3.1.2 Query Builder Inteligente

- [ ] **Archivo**: `services/saas-mt-search-service/internal/domain/service/query_builder.go`
  ```go
  package service

  import (
      "context"
      "strings"
      "regexp"
  )

  type QueryBuilder interface {
      BuildElasticQuery(ctx context.Context, searchQuery *SearchQuery) (map[string]interface{}, error)
      ExtractFiltersFromText(ctx context.Context, query string) (*ExtractedFilters, error)
      BuildSuggestionQuery(ctx context.Context, partial string, tenantID string) (map[string]interface{}, error)
      BuildFacetQuery(ctx context.Context, tenantID string, categoryID string) (map[string]interface{}, error)
  }

  type ExtractedFilters struct {
      CleanQuery       string            `json:"clean_query"`
      CategoryHints    []string          `json:"category_hints"`
      AttributeHints   map[string]string `json:"attribute_hints"`
      BrandHints       []string          `json:"brand_hints"`
      ColorHints       []string          `json:"color_hints"`
      SizeHints        []string          `json:"size_hints"`
      PriceHints       *PriceRange       `json:"price_hints,omitempty"`
      ConfidenceScore  float64           `json:"confidence_score"`
  }

  type SmartQueryBuilder struct {
      categoryMatcher   *CategoryMatcher
      attributeMatcher  *AttributeMatcher
      synonymsEngine    *SynonymsEngine
  }

  func (qb *SmartQueryBuilder) ExtractFiltersFromText(ctx context.Context, query string) (*ExtractedFilters, error) {
      filters := &ExtractedFilters{
          CleanQuery:     query,
          AttributeHints: make(map[string]string),
          CategoryHints:  []string{},
          BrandHints:     []string{},
          ColorHints:     []string{},
          SizeHints:      []string{},
      }

      // Detectar colores
      colorPatterns := map[string][]string{
          "negro":  {"negro", "negra", "black"},
          "blanco": {"blanco", "blanca", "white"},
          "rojo":   {"rojo", "roja", "red"},
          "azul":   {"azul", "blue"},
      }

      // Detectar talles
      sizePattern := regexp.MustCompile(`\b(talle|talla|size)\s+(\d+|XS|S|M|L|XL|XXL)\b`)
      
      // Detectar precios
      pricePattern := regexp.MustCompile(`\$\s*(\d+)(?:\s*[-a]\s*\$?\s*(\d+))?`)

      // Detectar categorías por keywords
      categoryKeywords := map[string][]string{
          "mp-cat-fashion-footwear": {"zapatilla", "zapato", "bota", "sandalia"},
          "mp-cat-electronics-phones": {"celular", "telefono", "smartphone"},
          "mp-cat-home-furniture": {"mesa", "silla", "sofa", "mueble"},
      }

      // Aplicar detección...
      return filters, nil
  }
  ```

- [ ] **Archivo**: `services/saas-mt-search-service/internal/domain/service/category_matcher.go`
- [ ] **Archivo**: `services/saas-mt-search-service/internal/domain/service/attribute_matcher.go`
- [ ] **Archivo**: `services/saas-mt-search-service/internal/domain/service/synonyms_engine.go`

#### [ ] 3.1.3 Sistema de Autocompletado

- [ ] **Archivo**: `services/saas-mt-search-service/internal/application/usecase/suggest_search.go`
  ```go
  package usecase

  import (
      "context"
      "strings"
      "sort"
  )

  type SuggestSearchUseCase struct {
      searchRepo        port.SearchRepository
      categoryRepo      port.CategoryRepository
      queryBuilder      service.QueryBuilder
      popularQueries    service.PopularQueriesCache
  }

  type SuggestionResponse struct {
      Suggestions      []Suggestion `json:"suggestions"`
      CategorySuggestions []CategorySuggestion `json:"category_suggestions"`
      ProductSuggestions  []ProductSuggestion  `json:"product_suggestions"`
      PopularSearches     []string `json:"popular_searches"`
      QueryTime           float64  `json:"query_time_ms"`
  }

  type Suggestion struct {
      Text        string  `json:"text"`
      Type        string  `json:"type"` // "completion", "category", "product", "brand"
      Score       float64 `json:"score"`
      ResultCount int     `json:"result_count,omitempty"`
  }

  func (uc *SuggestSearchUseCase) Execute(ctx context.Context, req *SuggestSearchRequest) (*SuggestionResponse, error) {
      if len(req.Query) < 2 {
          return uc.getPopularSuggestions(ctx, req.TenantID)
      }

      // 1. Autocompletado de texto
      textSuggestions := uc.generateTextCompletions(ctx, req.Query)

      // 2. Sugerencias de categorías
      categorySuggestions := uc.suggestCategories(ctx, req.Query, req.TenantID)

      // 3. Sugerencias de productos específicos
      productSuggestions := uc.suggestProducts(ctx, req.Query, req.TenantID)

      // 4. Búsquedas populares relacionadas
      popularSuggestions := uc.popularQueries.GetRelated(req.Query, req.TenantID)

      return &SuggestionResponse{
          Suggestions:         textSuggestions,
          CategorySuggestions: categorySuggestions,
          ProductSuggestions:  productSuggestions,
          PopularSearches:     popularSuggestions,
      }, nil
  }
  ```

- [ ] **Archivo**: `services/saas-mt-search-service/internal/infrastructure/cache/popular_queries_cache.go`
- [ ] **Archivo**: `services/saas-mt-search-service/internal/infrastructure/cache/suggestion_cache.go`

### 3.2 Sistema de Facetas Dinámicas
**Responsable**: Frontend + Backend Developer | **Estimación**: 4 días

#### [ ] 3.2.1 Generador de Facetas

- [ ] **Archivo**: `services/saas-mt-search-service/internal/application/usecase/generate_facets.go`
  ```go
  package usecase

  import (
      "context"
      "sort"
  )

  type GenerateFacetsUseCase struct {
      searchRepo    port.SearchRepository
      categoryRepo  port.CategoryRepository
      attributeRepo port.AttributeRepository
  }

  type FacetResponse struct {
      CategoryFacets   []CategoryFacet   `json:"category_facets"`
      AttributeFacets  []AttributeFacet  `json:"attribute_facets"`
      PriceFacet       *PriceFacet       `json:"price_facet,omitempty"`
      BrandFacet       *BrandFacet       `json:"brand_facet,omitempty"`
      AvailabilityFacet *AvailabilityFacet `json:"availability_facet,omitempty"`
  }

  type CategoryFacet struct {
      ID          string           `json:"id"`
      Name        string           `json:"name"`
      Slug        string           `json:"slug"`
      Count       int              `json:"count"`
      Children    []CategoryFacet  `json:"children,omitempty"`
      IsSelected  bool             `json:"is_selected"`
  }

  type AttributeFacet struct {
      ID          string              `json:"id"`
      Name        string              `json:"name"`
      Type        string              `json:"type"`
      Values      []AttributeValue    `json:"values"`
      IsMultiple  bool                `json:"is_multiple"`
      Priority    int                 `json:"priority"`
  }

  type AttributeValue struct {
      Value       string `json:"value"`
      Count       int    `json:"count"`
      IsSelected  bool   `json:"is_selected"`
  }

  type PriceFacet struct {
      MinPrice    float64      `json:"min_price"`
      MaxPrice    float64      `json:"max_price"`
      Ranges      []PriceRange `json:"ranges"`
      Histogram   []PriceBucket `json:"histogram"`
  }
  ```

- [ ] **Archivo**: `services/saas-mt-search-service/internal/domain/service/facet_generator.go`
- [ ] **Archivo**: `services/saas-mt-search-service/internal/infrastructure/elasticsearch/facet_aggregator.go`

#### [ ] 3.2.2 UI Facetas Inteligentes

- [ ] **Archivo**: `services/saas-mt-backoffice/src/components/search/SearchFacets.jsx`
  ```jsx
  import React, { useState, useEffect } from 'react';
  import { searchApi } from '../../../lib/api';

  const SearchFacets = ({ query, selectedFilters, onFiltersChange, tenantId }) => {
    const [facets, setFacets] = useState(null);
    const [loading, setLoading] = useState(false);

    useEffect(() => {
      loadFacets();
    }, [query, selectedFilters]);

    const loadFacets = async () => {
      setLoading(true);
      try {
        const response = await searchApi.getFacets({
          query,
          tenant_id: tenantId,
          selected_filters: selectedFilters
        });
        setFacets(response.data);
      } catch (error) {
        console.error('Error loading facets:', error);
      } finally {
        setLoading(false);
      }
    };

    const handleFilterChange = (facetType, facetId, value, isSelected) => {
      const newFilters = { ...selectedFilters };
      
      if (!newFilters[facetType]) {
        newFilters[facetType] = [];
      }

      if (isSelected) {
        newFilters[facetType] = newFilters[facetType].filter(f => f !== value);
      } else {
        newFilters[facetType].push(value);
      }

      onFiltersChange(newFilters);
    };

    if (loading) return <FacetsSkeleton />;

    return (
      <div className="search-facets">
        {/* Categorías */}
        {facets?.category_facets && (
          <FacetSection title="Categorías">
            {facets.category_facets.map(category => (
              <CategoryFacetItem 
                key={category.id}
                category={category}
                onSelect={(cat) => handleFilterChange('categories', cat.id, cat.id, cat.is_selected)}
              />
            ))}
          </FacetSection>
        )}

        {/* Atributos */}
        {facets?.attribute_facets?.map(attribute => (
          <AttributeFacetSection 
            key={attribute.id}
            attribute={attribute}
            selectedValues={selectedFilters.attributes?.[attribute.id] || []}
            onValuesChange={(values) => {
              const newFilters = { ...selectedFilters };
              if (!newFilters.attributes) newFilters.attributes = {};
              newFilters.attributes[attribute.id] = values;
              onFiltersChange(newFilters);
            }}
          />
        ))}

        {/* Precio */}
        {facets?.price_facet && (
          <PriceFacetSection 
            facet={facets.price_facet}
            selectedRange={selectedFilters.price_range}
            onRangeChange={(range) => handleFilterChange('price_range', 'price', range, false)}
          />
        )}
      </div>
    );
  };
  ```

- [ ] **Archivo**: `services/saas-mt-backoffice/src/components/search/FacetSection.jsx`
- [ ] **Archivo**: `services/saas-mt-backoffice/src/components/search/CategoryFacetItem.jsx`
- [ ] **Archivo**: `services/saas-mt-backoffice/src/components/search/AttributeFacetSection.jsx`
- [ ] **Archivo**: `services/saas-mt-backoffice/src/components/search/PriceFacetSection.jsx`

### 3.3 Búsqueda Avanzada
**Responsable**: Full Stack Developer | **Estimación**: 3 días

#### [ ] 3.3.1 Constructor de Queries Complejas

- [ ] **Archivo**: `services/saas-mt-backoffice/src/pages/search/AdvancedSearch.jsx`
  ```jsx
  import React, { useState } from 'react';
  import { searchApi } from '../../../lib/api';

  const AdvancedSearch = () => {
    const [searchCriteria, setSearchCriteria] = useState({
      text_query: '',
      categories: [],
      attributes: {},
      price_range: { min: null, max: null },
      in_stock: null,
      sort_by: 'relevance',
      created_date_range: { from: null, to: null }
    });

    const [results, setResults] = useState(null);
    const [savedSearches, setSavedSearches] = useState([]);

    const handleSearch = async () => {
      try {
        const response = await searchApi.advancedSearch(searchCriteria);
        setResults(response.data);
      } catch (error) {
        console.error('Error in advanced search:', error);
      }
    };

    const saveSearch = async (name) => {
      try {
        await searchApi.saveSearch({
          name,
          criteria: searchCriteria,
          tenant_id: currentTenant.id
        });
        loadSavedSearches();
      } catch (error) {
        console.error('Error saving search:', error);
      }
    };

    return (
      <div className="advanced-search">
        <div className="search-builder">
          <SearchCriteriaBuilder 
            criteria={searchCriteria}
            onChange={setSearchCriteria}
          />
          
          <div className="search-actions">
            <button onClick={handleSearch} className="btn-primary">
              Buscar
            </button>
            <button onClick={() => saveSearch(prompt('Nombre para esta búsqueda:'))}>
              Guardar Búsqueda
            </button>
          </div>
        </div>

        <div className="saved-searches">
          <h3>Búsquedas Guardadas</h3>
          {savedSearches.map(search => (
            <SavedSearchItem 
              key={search.id}
              search={search}
              onLoad={(criteria) => setSearchCriteria(criteria)}
              onDelete={(id) => deleteSavedSearch(id)}
            />
          ))}
        </div>

        {results && (
          <SearchResults 
            results={results}
            criteria={searchCriteria}
          />
        )}
      </div>
    );
  };
  ```

- [ ] **Archivo**: `services/saas-mt-backoffice/src/components/search/SearchCriteriaBuilder.jsx`
- [ ] **Archivo**: `services/saas-mt-backoffice/src/components/search/SavedSearchItem.jsx`
- [ ] **Archivo**: `services/saas-mt-backoffice/src/components/search/SearchResults.jsx`

**Migration para búsquedas guardadas**:
- [ ] **Archivo**: `services/saas-mt-search-service/migrations/001_create_saved_searches.sql`
  ```sql
  CREATE TABLE saved_searches (
      id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
      tenant_id UUID NOT NULL,
      user_id UUID,
      name VARCHAR(255) NOT NULL,
      search_criteria JSONB NOT NULL,
      is_public BOOLEAN DEFAULT FALSE,
      usage_count INTEGER DEFAULT 0,
      last_used_at TIMESTAMP WITH TIME ZONE,
      created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
      updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
  );

  CREATE INDEX idx_saved_searches_tenant ON saved_searches(tenant_id);
  CREATE INDEX idx_saved_searches_user ON saved_searches(user_id);
  CREATE INDEX idx_saved_searches_public ON saved_searches(is_public) WHERE is_public = TRUE;
  ```

### 3.4 APIs Públicas del Marketplace
**Responsable**: Backend Developer | **Estimación**: 2 días

#### [ ] 3.4.1 Endpoints Públicos (Sin Auth)

**Usuario**: Compradores navegando el marketplace

```
GET /search/api/v1/public/marketplace/categories    # Navegación principal
GET /search/api/v1/public/marketplace/search        # Búsqueda global
GET /search/api/v1/public/marketplace/filters       # Filtros disponibles
GET /search/api/v1/public/products/:id              # Detalle de producto
```

**Ejemplo de uso**:
```javascript
// Comprador busca "remeras verano"
GET /search/api/v1/public/marketplace/search?q=remeras%20verano&category=fashion-tops

// Respuesta: productos de TODOS los vendedores que matchean
{
  "items": [
    {"id": "prod-1", "name": "Remera Tropical", "tenant": "maria-boutique", "price": 15},
    {"id": "prod-2", "name": "Musculosa Verano", "tenant": "ropa-joven", "price": 12}
  ],
  "filters": {
    "size": ["S", "M", "L", "XL"],
    "color": ["blanco", "negro", "azul"],
    "price_range": [{"min": 10, "max": 20, "count": 15}]
  }
}
```

---

## 🎨 FASE 4: BACKOFFICE MARKETPLACE (Semanas 10-12)
**Responsable**: Frontend + UX Team | **Estimación**: 15 días

### 💡 ¿Por qué separar paneles Admin vs Tenant?

**Principio**: Cada usuario ve solo lo que necesita para su rol

**Panel Admin Global**: 
- Usuario: Administradores del marketplace
- Función: Configurar taxonomía, ver analytics agregados
- Complejidad: Alta (herramientas profesionales)

**Panel Tenant**:
- Usuario: Vendedores individuales
- Función: Configurar SU tienda, cargar productos
- Complejidad: Baja (herramientas simples)

### 4.1 Panel Tenant: La Vista del Vendedor
**Responsable**: Frontend + UX Developer | **Estimación**: 8 días

#### [ ] 4.1.1 Dashboard Personalizado Seller

- [ ] **Archivo**: `services/saas-mt-backoffice/src/pages/tenant/TenantDashboard.jsx`
  ```jsx
  import React, { useState, useEffect } from 'react';
  import { tenantApi, marketplaceApi } from '../../../lib/api';

  const TenantDashboard = () => {
    const [dashboardData, setDashboardData] = useState(null);
    const [marketplaceConfig, setMarketplaceConfig] = useState(null);
    const [loading, setLoading] = useState(true);

    useEffect(() => {
      loadDashboardData();
    }, []);

    const loadDashboardData = async () => {
      try {
        const [dashboard, config] = await Promise.all([
          tenantApi.getDashboardData(),
          marketplaceApi.getTenantConfiguration()
        ]);
        setDashboardData(dashboard.data);
        setMarketplaceConfig(config.data);
      } catch (error) {
        console.error('Error loading dashboard:', error);
      } finally {
        setLoading(false);
      }
    };

    if (loading) return <DashboardSkeleton />;

    return (
      <div className="tenant-dashboard">
        {/* Header con info del seller */}
        <TenantHeader 
          tenantInfo={dashboardData.tenant_info}
          onboardingComplete={marketplaceConfig?.onboarding_completed}
        />

        {/* Quick stats */}
        <div className="quick-stats">
          <StatCard 
            title="Productos Publicados"
            value={dashboardData.products_count}
            change={dashboardData.products_change}
            icon="📦"
          />
          <StatCard 
            title="Categorías Activas"
            value={dashboardData.active_categories}
            change={dashboardData.categories_change}
            icon="📁"
          />
          <StatCard 
            title="Visibilidad en Búsquedas"
            value={`${dashboardData.search_visibility}%`}
            change={dashboardData.visibility_change}
            icon="🔍"
          />
          <StatCard 
            title="Configuración Marketplace"
            value={marketplaceConfig?.completion_percentage}
            type="progress"
            icon="⚙️"
          />
        </div>

        {/* Acciones rápidas */}
        <QuickActions 
          onCreateProduct={() => navigate('/products/create')}
          onConfigureCategories={() => navigate('/marketplace/categories')}
          onViewAnalytics={() => navigate('/analytics')}
        />

        {/* Categorías configuradas */}
        <MarketplaceCategoriesOverview 
          categories={marketplaceConfig?.categories || []}
          onConfigure={(categoryId) => navigate(`/marketplace/categories/${categoryId}`)}
        />
      </div>
    );
  };
  ```

- [ ] **Archivo**: `services/saas-mt-backoffice/src/components/tenant/TenantHeader.jsx`
- [ ] **Archivo**: `services/saas-mt-backoffice/src/components/tenant/StatCard.jsx`
- [ ] **Archivo**: `services/saas-mt-backoffice/src/components/tenant/QuickActions.jsx`
- [ ] **Archivo**: `services/saas-mt-backoffice/src/components/tenant/MarketplaceCategoriesOverview.jsx`

#### [ ] 4.1.2 Configuración de Categorías

**Pantalla del vendedor**:
```
📊 Mis Categorías de Productos

✅ Remeras y Tops          (23 productos) [Personalizar]
✅ Pantalones y Faldas     (15 productos) [Personalizar]  
⚪ Calzado                 (0 productos)  [Activar]
⚪ Accesorios              (0 productos)  [Activar]

+ Solicitar nueva categoría
```

**Ventaja vs MercadoLibre**: 
- ML: 5000+ categorías, navegación confusa
- Nosotros: Solo categorías relevantes para TU negocio

- [ ] **Archivo**: `services/saas-mt-backoffice/src/pages/marketplace/TenantCategoriesConfig.jsx`
  ```jsx
  import React, { useState, useEffect } from 'react';
  import { marketplaceApi } from '../../../lib/api';

  const TenantCategoriesConfig = () => {
    const [availableCategories, setAvailableCategories] = useState([]);
    const [tenantMappings, setTenantMappings] = useState([]);
    const [loading, setLoading] = useState(true);

    const CategoryConfigCard = ({ category, mapping, onConfigure, onToggle }) => (
      <div className={`category-card ${mapping?.is_active ? 'active' : 'inactive'}`}>
        <div className="category-header">
          <h3>{mapping?.tenant_category_name || category.name}</h3>
          <span className="product-count">
            {mapping?.product_count || 0} productos
          </span>
        </div>

        <div className="category-info">
          <p>Categoría Marketplace: {category.name}</p>
          <p>Atributos disponibles: {category.attributes?.length || 0}</p>
        </div>

        <div className="category-actions">
          {mapping?.is_active ? (
            <>
              <button 
                onClick={() => onConfigure(mapping.id)}
                className="btn-secondary"
              >
                Personalizar
              </button>
              <button 
                onClick={() => onToggle(mapping.id, false)}
                className="btn-outline"
              >
                Desactivar
              </button>
            </>
          ) : (
            <button 
              onClick={() => onToggle(category.id, true)}
              className="btn-primary"
            >
              Activar
            </button>
          )}
        </div>

        {mapping?.is_active && (
          <div className="category-config-preview">
            <h4>Mi configuración:</h4>
            <ul>
              {mapping.custom_attributes?.map(attr => (
                <li key={attr.id}>{attr.name}: {attr.type}</li>
              ))}
            </ul>
          </div>
        )}
      </div>
    );

    return (
      <div className="tenant-categories-config">
        <h1>Configurar Mis Categorías</h1>
        
        <div className="categories-grid">
          {availableCategories.map(category => {
            const mapping = tenantMappings.find(m => m.marketplace_category_id === category.id);
            return (
              <CategoryConfigCard 
                key={category.id}
                category={category}
                mapping={mapping}
                onConfigure={handleConfigureCategory}
                onToggle={handleToggleCategory}
              />
            );
          })}
        </div>

        <div className="request-new-category">
          <button onClick={() => setShowRequestForm(true)} className="btn-outline">
            + Solicitar nueva categoría
          </button>
        </div>
      </div>
    );
  };
  ```

- [ ] **Archivo**: `services/saas-mt-backoffice/src/pages/marketplace/CategoryCustomizer.jsx`
- [ ] **Archivo**: `services/saas-mt-backoffice/src/components/marketplace/CategoryConfigCard.jsx`
- [ ] **Archivo**: `services/saas-mt-backoffice/src/components/marketplace/AttributeCustomizer.jsx`

#### [ ] 4.1.3 Formulario de Producto Híbrido

**La experiencia del vendedor cargando un producto**:

```
Crear Nuevo Producto

📁 Categoría: [Remeras y Tops ▼]

📝 Información Básica:
   Nombre: [Remera Básica Algodón]
   Descripción: [...]
   SKU: [REM-001] (auto-generado)

🏷️ Atributos Marketplace (para búsquedas):
   ✅ Talle: [M ▼] [L ▼] [XL ▼]
   ✅ Color: [Negro ▼] [Blanco ▼]
   ✅ Material: [Algodón ▼]

🎨 Mis Atributos Personalizados:
   • Temporada: [Verano ▼]
   • Ocasión: [Casual ▼]
   + Agregar atributo

💰 Precio y Stock:
   Precio: [$1500]
   Stock: [50 unidades]

[Generar variantes automáticamente] [Guardar producto]
```

**El diferencial**: Formulario inteligente que se adapta a lo que el vendedor realmente necesita

- [ ] **Archivo**: `services/saas-mt-backoffice/src/pages/products/CreateProductMarketplace.jsx`
  ```jsx
  import React, { useState, useEffect } from 'react';
  import { productsApi, marketplaceApi } from '../../../lib/api';

  const CreateProductMarketplace = () => {
    const [productData, setProductData] = useState({
      name: '',
      description: '',
      sku: '',
      category_id: '',
      marketplace_attributes: {},
      custom_attributes: {},
      price: '',
      stock: '',
      images: []
    });

    const [availableCategories, setAvailableCategories] = useState([]);
    const [categoryAttributes, setCategoryAttributes] = useState([]);
    const [customAttributes, setCustomAttributes] = useState([]);

    const handleCategoryChange = async (categoryId) => {
      setProductData({...productData, category_id: categoryId});
      
      // Cargar atributos de la categoría
      const [marketplaceAttrs, customAttrs] = await Promise.all([
        marketplaceApi.getCategoryAttributes(categoryId),
        marketplaceApi.getTenantCustomAttributes(categoryId)
      ]);
      
      setCategoryAttributes(marketplaceAttrs.data);
      setCustomAttributes(customAttrs.data);
    };

    const MarketplaceAttributesSection = () => (
      <div className="attributes-section">
        <h3>🏷️ Atributos Marketplace (para búsquedas)</h3>
        <p className="help-text">
          Estos atributos ayudan a que los compradores encuentren tu producto
        </p>
        
        {categoryAttributes.map(attr => (
          <AttributeInput
            key={attr.id}
            attribute={attr}
            value={productData.marketplace_attributes[attr.id]}
            onChange={(value) => {
              setProductData({
                ...productData,
                marketplace_attributes: {
                  ...productData.marketplace_attributes,
                  [attr.id]: value
                }
              });
            }}
            required={attr.is_required}
            isVariantForming={attr.is_variant_forming}
          />
        ))}
      </div>
    );

    const CustomAttributesSection = () => (
      <div className="attributes-section">
        <h3>🎨 Mis Atributos Personalizados</h3>
        <p className="help-text">
          Información específica de tu tienda
        </p>
        
        {customAttributes.map(attr => (
          <AttributeInput
            key={attr.id}
            attribute={attr}
            value={productData.custom_attributes[attr.id]}
            onChange={(value) => {
              setProductData({
                ...productData,
                custom_attributes: {
                  ...productData.custom_attributes,
                  [attr.id]: value
                }
              });
            }}
          />
        ))}
        
        <button onClick={handleAddCustomAttribute} className="btn-outline">
          + Agregar atributo personalizado
        </button>
      </div>
    );

    return (
      <div className="create-product-marketplace">
        <h1>Crear Nuevo Producto</h1>
        
        <form onSubmit={handleSubmit}>
          {/* Categoría */}
          <CategorySelector 
            categories={availableCategories}
            value={productData.category_id}
            onChange={handleCategoryChange}
          />

          {/* Información básica */}
          <BasicInfoSection 
            data={productData}
            onChange={setProductData}
          />

          {/* Atributos Marketplace */}
          {productData.category_id && <MarketplaceAttributesSection />}

          {/* Atributos Personalizados */}
          {productData.category_id && <CustomAttributesSection />}

          {/* Precio y Stock */}
          <PriceStockSection 
            data={productData}
            onChange={setProductData}
          />

          {/* Imágenes */}
          <ImageUploadSection 
            images={productData.images}
            onChange={(images) => setProductData({...productData, images})}
          />

          {/* Acciones */}
          <div className="form-actions">
            <button type="button" onClick={handleGenerateVariants} className="btn-secondary">
              Generar variantes automáticamente
            </button>
            <button type="submit" className="btn-primary">
              Guardar producto
            </button>
          </div>
        </form>
      </div>
    );
  };
  ```

- [ ] **Archivo**: `services/saas-mt-backoffice/src/components/products/CategorySelector.jsx`
- [ ] **Archivo**: `services/saas-mt-backoffice/src/components/products/AttributeInput.jsx`
- [ ] **Archivo**: `services/saas-mt-backoffice/src/components/products/BasicInfoSection.jsx`
- [ ] **Archivo**: `services/saas-mt-backoffice/src/components/products/PriceStockSection.jsx`
- [ ] **Archivo**: `services/saas-mt-backoffice/src/components/products/ImageUploadSection.jsx`

### 4.2 Panel Admin Global
**Responsable**: Full Stack Developer | **Estimación**: 7 días

#### [ ] 4.2.1 Gestión de Taxonomía Global

- [ ] **Archivo**: `services/saas-mt-backoffice/src/pages/admin/MarketplaceTaxonomy.jsx`
  ```jsx
  import React, { useState, useEffect } from 'react';
  import { adminApi } from '../../../lib/api';

  const MarketplaceTaxonomy = () => {
    const [categories, setCategories] = useState([]);
    const [attributes, setAttributes] = useState([]);
    const [selectedCategory, setSelectedCategory] = useState(null);

    const CategoryTreeView = ({ categories, onSelect }) => (
      <div className="category-tree">
        {categories.map(category => (
          <CategoryNode 
            key={category.id}
            category={category}
            onSelect={onSelect}
            isSelected={selectedCategory?.id === category.id}
          />
        ))}
      </div>
    );

    const AttributeManager = ({ categoryId }) => (
      <div className="attribute-manager">
        <h3>Atributos de la categoría</h3>
        
        <div className="attributes-list">
          {attributes
            .filter(attr => attr.categories.includes(categoryId))
            .map(attribute => (
              <AttributeCard 
                key={attribute.id}
                attribute={attribute}
                onEdit={handleEditAttribute}
                onDelete={handleDeleteAttribute}
              />
            ))}
        </div>

        <button onClick={() => setShowCreateAttribute(true)} className="btn-primary">
          + Crear nuevo atributo
        </button>
      </div>
    );

    return (
      <div className="marketplace-taxonomy">
        <h1>Gestión de Taxonomía Global</h1>
        
        <div className="taxonomy-layout">
          <div className="categories-panel">
            <h2>Categorías</h2>
            <CategoryTreeView 
              categories={categories}
              onSelect={setSelectedCategory}
            />
            <button onClick={() => setShowCreateCategory(true)} className="btn-primary">
              + Nueva categoría
            </button>
          </div>

          <div className="details-panel">
            {selectedCategory ? (
              <>
                <CategoryDetails category={selectedCategory} />
                <AttributeManager categoryId={selectedCategory.id} />
              </>
            ) : (
              <div className="no-selection">
                Selecciona una categoría para ver detalles
              </div>
            )}
          </div>
        </div>
      </div>
    );
  };
  ```

- [ ] **Archivo**: `services/saas-mt-backoffice/src/components/admin/CategoryNode.jsx`
- [ ] **Archivo**: `services/saas-mt-backoffice/src/components/admin/CategoryDetails.jsx`
- [ ] **Archivo**: `services/saas-mt-backoffice/src/components/admin/AttributeCard.jsx`
- [ ] **Archivo**: `services/saas-mt-backoffice/src/components/admin/AttributeManager.jsx`

#### [ ] 4.2.2 Analytics Agregados

- [ ] **Archivo**: `services/saas-mt-backoffice/src/pages/admin/MarketplaceAnalytics.jsx`
  ```jsx
  import React, { useState, useEffect } from 'react';
  import { analyticsApi } from '../../../lib/api';

  const MarketplaceAnalytics = () => {
    const [analyticsData, setAnalyticsData] = useState(null);
    const [timeRange, setTimeRange] = useState('last_30_days');

    const TenantStatsGrid = ({ tenants }) => (
      <div className="tenant-stats-grid">
        {tenants.map(tenant => (
          <TenantStatCard 
            key={tenant.id}
            tenant={tenant}
            stats={tenant.stats}
          />
        ))}
      </div>
    );

    const CategoryPerformance = ({ categories }) => (
      <div className="category-performance">
        <h3>Performance por Categoría</h3>
        <div className="performance-chart">
          {categories.map(category => (
            <CategoryBar 
              key={category.id}
              category={category}
              productCount={category.product_count}
              searchVolume={category.search_volume}
              conversionRate={category.conversion_rate}
            />
          ))}
        </div>
      </div>
    );

    return (
      <div className="marketplace-analytics">
        <h1>Analytics del Marketplace</h1>
        
        {/* KPIs globales */}
        <div className="global-kpis">
          <KPICard 
            title="Tenants Activos"
            value={analyticsData?.active_tenants}
            change={analyticsData?.tenants_change}
          />
          <KPICard 
            title="Productos Totales"
            value={analyticsData?.total_products}
            change={analyticsData?.products_change}
          />
          <KPICard 
            title="Búsquedas/día"
            value={analyticsData?.daily_searches}
            change={analyticsData?.searches_change}
          />
          <KPICard 
            title="Categorías Activas"
            value={analyticsData?.active_categories}
            change={analyticsData?.categories_change}
          />
        </div>

        {/* Performance por tenant */}
        <TenantStatsGrid tenants={analyticsData?.tenants || []} />

        {/* Performance por categoría */}
        <CategoryPerformance categories={analyticsData?.categories || []} />
      </div>
    );
  };
  ```

- [ ] **Archivo**: `services/saas-mt-backoffice/src/components/admin/TenantStatCard.jsx`
- [ ] **Archivo**: `services/saas-mt-backoffice/src/components/admin/KPICard.jsx`
- [ ] **Archivo**: `services/saas-mt-backoffice/src/components/admin/CategoryBar.jsx`

---

## 📊 FASE 5: ANALYTICS Y OPTIMIZACIÓN (Semanas 13-14)
**Responsable**: Data + Analytics Team | **Estimación**: 10 días

### 💡 ¿Por qué analytics específicos para marketplace?

**Pregunta del vendedor**: "¿Está funcionando mi configuración?"

**Métricas que importan al seller**:
- ¿Mis productos aparecen en búsquedas?
- ¿Las categorías que elegí generan ventas?
- ¿Qué atributos son más importantes para mis compradores?

### 5.1 Analytics para Sellers
**Responsable**: Full Stack + Analytics Developer | **Estimación**: 6 días

#### [ ] 5.1.1 Dashboard Performance Tenant

```
📊 Mi Performance en el Marketplace

🔍 Visibilidad:
   • Productos en búsquedas: 87% (+5% vs mes anterior)
   • Categorías más vistas: "Remeras" (245 vistas), "Pantalones" (89 vistas)
   • Filtros más usados: Talle (67%), Color (45%)

📈 Conversión:
   • Clicks en mis productos: 156 esta semana
   • Productos más vendidos por categoría
   • Atributos que más influyen en ventas

💡 Recomendaciones:
   • Considerar agregar atributo "Tipo de cuello" (muy buscado en tu categoría)
   • Optimizar fotos de productos en "Pantalones" (baja conversión)
```

**Valor**: Seller puede optimizar sin conocimiento técnico

- [ ] **Archivo**: `services/saas-mt-analytics-service/internal/domain/entity/tenant_analytics.go`
  ```go
  package entity

  import (
      "time"
      "github.com/google/uuid"
  )

  type TenantAnalytics struct {
      TenantID              uuid.UUID              `json:"tenant_id"`
      DateRange             DateRange              `json:"date_range"`
      ProductVisibility     ProductVisibilityStats `json:"product_visibility"`
      SearchPerformance     SearchPerformanceStats `json:"search_performance"`
      CategoryPerformance   []CategoryStats        `json:"category_performance"`
      AttributePerformance  []AttributeStats       `json:"attribute_performance"`
      Recommendations      []OptimizationSuggestion `json:"recommendations"`
      CompetitorBenchmark  *CompetitorBenchmark   `json:"competitor_benchmark,omitempty"`
  }

  type ProductVisibilityStats struct {
      TotalProducts         int     `json:"total_products"`
      ProductsInSearch      int     `json:"products_in_search"`
      VisibilityPercentage  float64 `json:"visibility_percentage"`
      VisibilityChange      float64 `json:"visibility_change"`
      AverageSearchPosition float64 `json:"average_search_position"`
      TopPerformingProducts []ProductPerformance `json:"top_performing_products"`
  }

  type SearchPerformanceStats struct {
      TotalSearches         int                    `json:"total_searches"`
      ClicksOnProducts      int                    `json:"clicks_on_products"`
      ClickThroughRate      float64                `json:"click_through_rate"`
      MostSearchedTerms     []SearchTermStats      `json:"most_searched_terms"`
      FilterUsageStats      []FilterUsageStats     `json:"filter_usage_stats"`
      ConversionsByCategory map[string]float64     `json:"conversions_by_category"`
  }

  type CategoryStats struct {
      CategoryID            uuid.UUID `json:"category_id"`
      CategoryName          string    `json:"category_name"`
      ProductCount          int       `json:"product_count"`
      SearchVolume          int       `json:"search_volume"`
      ClickVolume           int       `json:"click_volume"`
      ConversionRate        float64   `json:"conversion_rate"`
      AveragePrice          float64   `json:"average_price"`
      CompetitorCount       int       `json:"competitor_count"`
      MarketSharePercent    float64   `json:"market_share_percent"`
  }

  type AttributeStats struct {
      AttributeID           uuid.UUID `json:"attribute_id"`
      AttributeName         string    `json:"attribute_name"`
      UsageInSearch         int       `json:"usage_in_search"`
      ConversionInfluence   float64   `json:"conversion_influence"`
      PopularValues         []string  `json:"popular_values"`
      MissingInProducts     int       `json:"missing_in_products"`
      OptimizationPotential float64   `json:"optimization_potential"`
  }

  type OptimizationSuggestion struct {
      Type                string    `json:"type"` // "add_attribute", "optimize_category", "price_adjustment"
      Priority            string    `json:"priority"` // "high", "medium", "low"
      Title               string    `json:"title"`
      Description         string    `json:"description"`
      ExpectedImpact      string    `json:"expected_impact"`
      ImplementationCost  string    `json:"implementation_cost"`
      ActionURL           string    `json:"action_url,omitempty"`
  }
  ```

- [ ] **Archivo**: `services/saas-mt-analytics-service/internal/application/usecase/generate_tenant_analytics.go`
- [ ] **Archivo**: `services/saas-mt-analytics-service/internal/application/usecase/generate_recommendations.go`
- [ ] **Archivo**: `services/saas-mt-analytics-service/internal/infrastructure/http/analytics_handler.go`

**Endpoints Analytics**:
```
GET /analytics/api/v1/tenant/dashboard              # Dashboard principal del tenant
GET /analytics/api/v1/tenant/visibility             # Métricas de visibilidad
GET /analytics/api/v1/tenant/search-performance     # Performance en búsquedas
GET /analytics/api/v1/tenant/category-stats         # Stats por categoría
GET /analytics/api/v1/tenant/recommendations        # Recomendaciones de optimización
GET /analytics/api/v1/tenant/competitor-benchmark   # Benchmark vs competencia
```

#### [ ] 5.1.2 Frontend Analytics Dashboard

- [ ] **Archivo**: `services/saas-mt-backoffice/src/pages/analytics/TenantAnalyticsDashboard.jsx`
  ```jsx
  import React, { useState, useEffect } from 'react';
  import { analyticsApi } from '../../../lib/api';

  const TenantAnalyticsDashboard = () => {
    const [analyticsData, setAnalyticsData] = useState(null);
    const [timeRange, setTimeRange] = useState('last_30_days');
    const [loading, setLoading] = useState(true);

    const VisibilitySection = ({ visibilityStats }) => (
      <div className="analytics-section">
        <h2>🔍 Visibilidad en el Marketplace</h2>
        
        <div className="visibility-stats">
          <MetricCard 
            title="Productos en Búsquedas"
            value={`${visibilityStats.visibility_percentage}%`}
            change={visibilityStats.visibility_change}
            subtitle={`${visibilityStats.products_in_search} de ${visibilityStats.total_products} productos`}
          />
          <MetricCard 
            title="Posición Promedio"
            value={visibilityStats.average_search_position.toFixed(1)}
            subtitle="En resultados de búsqueda"
          />
        </div>

        <div className="top-products">
          <h3>Productos con mejor performance</h3>
          {visibilityStats.top_performing_products.map(product => (
            <ProductPerformanceItem 
              key={product.id}
              product={product}
            />
          ))}
        </div>
      </div>
    );

    const SearchPerformanceSection = ({ searchStats }) => (
      <div className="analytics-section">
        <h2>📈 Performance en Búsquedas</h2>
        
        <div className="search-metrics">
          <MetricCard 
            title="Clicks Totales"
            value={searchStats.clicks_on_products}
            subtitle="Esta semana"
          />
          <MetricCard 
            title="CTR Promedio"
            value={`${(searchStats.click_through_rate * 100).toFixed(1)}%`}
            subtitle="Click-through rate"
          />
        </div>

        <SearchTermsChart terms={searchStats.most_searched_terms} />
        <FilterUsageChart filters={searchStats.filter_usage_stats} />
      </div>
    );

    const RecommendationsSection = ({ recommendations }) => (
      <div className="analytics-section">
        <h2>💡 Recomendaciones para Optimizar</h2>
        
        {recommendations.map((rec, index) => (
          <RecommendationCard 
            key={index}
            recommendation={rec}
            onImplement={() => handleImplementRecommendation(rec)}
          />
        ))}
      </div>
    );

    return (
      <div className="tenant-analytics-dashboard">
        <div className="dashboard-header">
          <h1>Mi Performance en el Marketplace</h1>
          <TimeRangeSelector 
            value={timeRange}
            onChange={setTimeRange}
          />
        </div>

        {loading ? (
          <AnalyticsLoadingSkeleton />
        ) : (
          <>
            <VisibilitySection visibilityStats={analyticsData.product_visibility} />
            <SearchPerformanceSection searchStats={analyticsData.search_performance} />
            <CategoryPerformanceChart categories={analyticsData.category_performance} />
            <RecommendationsSection recommendations={analyticsData.recommendations} />
          </>
        )}
      </div>
    );
  };
  ```

- [ ] **Archivo**: `services/saas-mt-backoffice/src/components/analytics/MetricCard.jsx`
- [ ] **Archivo**: `services/saas-mt-backoffice/src/components/analytics/ProductPerformanceItem.jsx`
- [ ] **Archivo**: `services/saas-mt-backoffice/src/components/analytics/SearchTermsChart.jsx`
- [ ] **Archivo**: `services/saas-mt-backoffice/src/components/analytics/FilterUsageChart.jsx`
- [ ] **Archivo**: `services/saas-mt-backoffice/src/components/analytics/RecommendationCard.jsx`
- [ ] **Archivo**: `services/saas-mt-backoffice/src/components/analytics/CategoryPerformanceChart.jsx`

### 5.2 Sistema de Recomendaciones Inteligentes
**Responsable**: Data Scientist + Backend Developer | **Estimación**: 4 días

#### [ ] 5.2.1 Motor de Recomendaciones

- [ ] **Archivo**: `services/saas-mt-analytics-service/internal/domain/service/recommendation_engine.go`
  ```go
  package service

  import (
      "context"
      "sort"
  )

  type RecommendationEngine interface {
      GenerateOptimizationSuggestions(ctx context.Context, tenantID string, analyticsData *TenantAnalytics) ([]OptimizationSuggestion, error)
      AnalyzeAttributeGaps(ctx context.Context, tenantID string) ([]AttributeGapAnalysis, error)
      IdentifyPricingOpportunities(ctx context.Context, tenantID string) ([]PricingRecommendation, error)
      SuggestCategoryExpansion(ctx context.Context, tenantID string) ([]CategoryExpansionSuggestion, error)
  }

  type SmartRecommendationEngine struct {
      analyticsRepo     port.AnalyticsRepository
      marketDataRepo    port.MarketDataRepository
      competitorAnalyzer *CompetitorAnalyzer
  }

  func (e *SmartRecommendationEngine) GenerateOptimizationSuggestions(ctx context.Context, tenantID string, data *TenantAnalytics) ([]OptimizationSuggestion, error) {
      suggestions := []OptimizationSuggestion{}

      // 1. Analizar atributos faltantes
      if missingAttrs := e.findMissingCriticalAttributes(data); len(missingAttrs) > 0 {
          for _, attr := range missingAttrs {
              suggestions = append(suggestions, OptimizationSuggestion{
                  Type:               "add_attribute",
                  Priority:           "high",
                  Title:              fmt.Sprintf("Agregar atributo '%s'", attr.Name),
                  Description:        fmt.Sprintf("El %d%% de búsquedas en tu categoría filtran por '%s', pero tus productos no lo tienen", int(attr.SearchUsage*100), attr.Name),
                  ExpectedImpact:     fmt.Sprintf("+%d%% de visibilidad", int(attr.ImpactEstimate*100)),
                  ImplementationCost: "Bajo - Solo completar información",
                  ActionURL:          fmt.Sprintf("/products/bulk-edit?add_attribute=%s", attr.ID),
              })
          }
      }

      // 2. Analizar posicionamiento de precios
      if priceOpts := e.analyzePricingPosition(ctx, tenantID, data); len(priceOpts) > 0 {
          suggestions = append(suggestions, priceOpts...)
      }

      // 3. Oportunidades de categorías
      if categoryOpts := e.findCategoryOpportunities(ctx, tenantID, data); len(categoryOpts) > 0 {
          suggestions = append(suggestions, categoryOpts...)
      }

      // 4. Optimización de fotos
      if photoOpts := e.analyzeImageQuality(ctx, tenantID); len(photoOpts) > 0 {
          suggestions = append(suggestions, photoOpts...)
      }

      // Ordenar por impacto esperado
      sort.Slice(suggestions, func(i, j int) bool {
          return e.calculatePriority(suggestions[i]) > e.calculatePriority(suggestions[j])
      })

      return suggestions, nil
  }

  type AttributeGapAnalysis struct {
      AttributeID     string  `json:"attribute_id"`
      AttributeName   string  `json:"attribute_name"`
      SearchUsage     float64 `json:"search_usage"`     // % de búsquedas que usan este filtro
      YourCoverage    float64 `json:"your_coverage"`    // % de tus productos que tienen este atributo
      CompetitorCoverage float64 `json:"competitor_coverage"` // % de competidores que lo tienen
      ImpactEstimate  float64 `json:"impact_estimate"`  // Estimación de mejora en visibilidad
      Priority        string  `json:"priority"`
  }
  ```

- [ ] **Archivo**: `services/saas-mt-analytics-service/internal/domain/service/competitor_analyzer.go`
- [ ] **Archivo**: `services/saas-mt-analytics-service/internal/domain/service/market_intelligence.go`
- [ ] **Archivo**: `services/saas-mt-analytics-service/internal/application/usecase/analyze_optimization_opportunities.go`

#### [ ] 5.2.2 Data Pipeline Analytics

- [ ] **Archivo**: `services/saas-mt-analytics-service/migrations/001_create_analytics_tables.sql`
  ```sql
  -- Eventos de búsqueda y clicks
  CREATE TABLE search_events (
      id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
      session_id VARCHAR(255),
      search_query TEXT NOT NULL,
      category_filters JSONB,
      attribute_filters JSONB,
      price_range JSONB,
      results_count INTEGER,
      clicked_products JSONB, -- Array de product_ids que fueron clickeados
      tenant_ids JSONB, -- Array de tenant_ids que aparecieron en resultados
      created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
  );

  -- Métricas agregadas por tenant y período
  CREATE TABLE tenant_analytics_summary (
      id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
      tenant_id UUID NOT NULL,
      date DATE NOT NULL,
      total_products INTEGER DEFAULT 0,
      products_in_search INTEGER DEFAULT 0,
      total_searches INTEGER DEFAULT 0,
      total_clicks INTEGER DEFAULT 0,
      total_impressions INTEGER DEFAULT 0,
      click_through_rate DECIMAL(5,4) DEFAULT 0,
      average_position DECIMAL(5,2) DEFAULT 0,
      category_stats JSONB DEFAULT '{}',
      attribute_stats JSONB DEFAULT '{}',
      created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
      updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
      UNIQUE(tenant_id, date)
  );

  -- Recomendaciones generadas
  CREATE TABLE optimization_recommendations (
      id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
      tenant_id UUID NOT NULL,
      type VARCHAR(50) NOT NULL,
      priority VARCHAR(20) NOT NULL,
      title VARCHAR(255) NOT NULL,
      description TEXT,
      expected_impact VARCHAR(100),
      implementation_cost VARCHAR(100),
      action_url VARCHAR(500),
      is_implemented BOOLEAN DEFAULT FALSE,
      implemented_at TIMESTAMP WITH TIME ZONE,
      created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
      expires_at TIMESTAMP WITH TIME ZONE
  );

  -- Índices para optimizar queries
  CREATE INDEX idx_search_events_query ON search_events USING gin(to_tsvector('spanish', search_query));
  CREATE INDEX idx_search_events_created_at ON search_events(created_at);
  CREATE INDEX idx_search_events_tenant_ids ON search_events USING gin(tenant_ids);

  CREATE INDEX idx_tenant_analytics_tenant_date ON tenant_analytics_summary(tenant_id, date);
  CREATE INDEX idx_tenant_analytics_date ON tenant_analytics_summary(date);

  CREATE INDEX idx_recommendations_tenant ON optimization_recommendations(tenant_id);
  CREATE INDEX idx_recommendations_priority ON optimization_recommendations(priority);
  CREATE INDEX idx_recommendations_implemented ON optimization_recommendations(is_implemented);
  ```

---

## 🧪 FASE 6: TESTING Y VALIDACIÓN (Semanas 15-16)
**Responsable**: QA + DevOps Team | **Estimación**: 10 días

### 💡 ¿Por qué testing con usuarios reales?

**Principio**: Los sellers reales encuentran problemas que nosotros no vemos

**Testing con María (Boutique de Bahía Blanca)**:
- ¿Puede configurar su tienda en menos de 10 minutos?
- ¿Entiende cómo personalizar categorías?
- ¿Le resulta fácil cargar productos?

**Testing con Marcos (Ferretería de Río Cuarto)**:
- ¿Encuentra los atributos relevantes para herramientas?
- ¿Puede generar variantes automáticamente?
- ¿Le sirven las recomendaciones de optimización?

### 6.1 Testing Funcional Completo
**Responsable**: QA Engineer | **Estimación**: 5 días

#### [ ] 6.1.1 Test Suite Onboarding

- [ ] **Archivo**: `tests/e2e/onboarding/business_type_selection.spec.js`
  ```javascript
  // tests/e2e/onboarding/business_type_selection.spec.js
  import { test, expect } from '@playwright/test';

  test.describe('Onboarding - Selección de Tipo de Negocio', () => {
    test('seller puede completar onboarding en menos de 10 minutos', async ({ page }) => {
      const startTime = Date.now();
      
      // Navegar al onboarding
      await page.goto('/onboarding');
      
      // Paso 1: Seleccionar tipo de negocio
      await page.click('[data-testid="business-type-fashion"]');
      await page.click('[data-testid="continue-btn"]');
      
      // Paso 2: Revisar categorías sugeridas
      await expect(page.locator('[data-testid="suggested-category"]')).toHaveCount(3);
      await page.click('[data-testid="accept-suggestions"]');
      
      // Paso 3: Personalizar atributos
      await page.fill('[data-testid="custom-attribute-name"]', 'Temporada');
      await page.selectOption('[data-testid="custom-attribute-type"]', 'select');
      await page.click('[data-testid="add-custom-attribute"]');
      
      // Paso 4: Completar onboarding
      await page.click('[data-testid="complete-onboarding"]');
      
      // Verificar que se completó exitosamente
      await expect(page.locator('[data-testid="success-message"]')).toBeVisible();
      
      const endTime = Date.now();
      const duration = (endTime - startTime) / 1000 / 60; // minutos
      expect(duration).toBeLessThan(10); // Menos de 10 minutos
    });

    test('suggestions son relevantes para cada tipo de negocio', async ({ page }) => {
      // Test para Fashion
      await page.goto('/onboarding');
      await page.click('[data-testid="business-type-fashion"]');
      await page.click('[data-testid="continue-btn"]');
      
      await expect(page.locator('[data-testid="suggested-category"]:has-text("Remeras")')).toBeVisible();
      await expect(page.locator('[data-testid="suggested-attribute"]:has-text("Talle")')).toBeVisible();
      
      // Test para Hardware
      await page.goto('/onboarding');
      await page.click('[data-testid="business-type-hardware"]');
      await page.click('[data-testid="continue-btn"]');
      
      await expect(page.locator('[data-testid="suggested-category"]:has-text("Herramientas")')).toBeVisible();
      await expect(page.locator('[data-testid="suggested-attribute"]:has-text("Material")')).toBeVisible();
    });
  });
  ```

- [ ] **Archivo**: `tests/e2e/products/create_product_marketplace.spec.js`
- [ ] **Archivo**: `tests/e2e/search/search_functionality.spec.js`
- [ ] **Archivo**: `tests/e2e/analytics/tenant_dashboard.spec.js`

#### [ ] 6.1.2 Test Suite Performance

- [ ] **Archivo**: `tests/performance/search_performance.spec.js`
  ```javascript
  // tests/performance/search_performance.spec.js
  import { test, expect } from '@playwright/test';

  test.describe('Performance Tests - Motor de Búsqueda', () => {
    test('búsqueda responde en menos de 500ms', async ({ page }) => {
      await page.goto('/');
      
      const startTime = Date.now();
      await page.fill('[data-testid="search-input"]', 'remera negra');
      await page.press('[data-testid="search-input"]', 'Enter');
      
      // Esperar a que aparezcan resultados
      await page.waitForSelector('[data-testid="search-results"]');
      const endTime = Date.now();
      
      const responseTime = endTime - startTime;
      expect(responseTime).toBeLessThan(500); // Menos de 500ms
    });

    test('autocompletado responde en menos de 200ms', async ({ page }) => {
      await page.goto('/');
      
      const startTime = Date.now();
      await page.fill('[data-testid="search-input"]', 'rem');
      
      // Esperar sugerencias
      await page.waitForSelector('[data-testid="search-suggestions"]');
      const endTime = Date.now();
      
      const responseTime = endTime - startTime;
      expect(responseTime).toBeLessThan(200); // Menos de 200ms
    });

    test('facetas se cargan en menos de 300ms', async ({ page }) => {
      await page.goto('/search?q=zapatillas');
      
      const startTime = Date.now();
      await page.click('[data-testid="category-filter-deportivas"]');
      
      // Esperar que se actualicen las facetas
      await page.waitForFunction(() => {
        const facets = document.querySelectorAll('[data-testid="facet-item"]');
        return facets.length > 0;
      });
      const endTime = Date.now();
      
      const responseTime = endTime - startTime;
      expect(responseTime).toBeLessThan(300); // Menos de 300ms
    });
  });
  ```

- [ ] **Archivo**: `tests/performance/dashboard_load_time.spec.js`
- [ ] **Archivo**: `tests/performance/bulk_operations.spec.js`

#### [ ] 6.1.3 Test Suite Cross-tenant

- [ ] **Archivo**: `tests/integration/cross_tenant_isolation.spec.js`
  ```javascript
  // tests/integration/cross_tenant_isolation.spec.js
  import { test, expect } from '@playwright/test';

  test.describe('Aislamiento Cross-Tenant', () => {
    test('tenant A no puede ver datos de tenant B', async ({ page, context }) => {
      // Login como Tenant A
      const pageA = await context.newPage();
      await pageA.goto('/login');
      await pageA.fill('[data-testid="email"]', 'maria@boutique.com');
      await pageA.fill('[data-testid="password"]', 'password');
      await pageA.click('[data-testid="login-btn"]');

      // Login como Tenant B
      const pageB = await context.newPage();
      await pageB.goto('/login');
      await pageB.fill('[data-testid="email"]', 'marcos@ferreteria.com');
      await pageB.fill('[data-testid="password"]', 'password');
      await pageB.click('[data-testid="login-btn"]');

      // Verificar que cada tenant ve solo sus productos
      await pageA.goto('/products');
      const productsA = await pageA.locator('[data-testid="product-item"]').count();

      await pageB.goto('/products');
      const productsB = await pageB.locator('[data-testid="product-item"]').count();

      // Los productos deben ser diferentes
      expect(productsA).toBeGreaterThan(0);
      expect(productsB).toBeGreaterThan(0);

      // Verificar que no hay overlap en IDs
      const productIdsA = await pageA.locator('[data-testid="product-item"]').getAttribute('data-product-id');
      const productIdsB = await pageB.locator('[data-testid="product-item"]').getAttribute('data-product-id');
      
      expect(productIdsA).not.toEqual(productIdsB);
    });

    test('búsqueda marketplace devuelve productos de múltiples tenants', async ({ page }) => {
      await page.goto('/marketplace/search?q=remera');
      
      // Verificar que hay resultados de diferentes sellers
      const sellerNames = await page.locator('[data-testid="seller-name"]').allTextContents();
      const uniqueSellers = [...new Set(sellerNames)];
      
      expect(uniqueSellers.length).toBeGreaterThan(1); // Múltiples sellers
    });
  });
  ```

### 6.2 User Acceptance Testing
**Responsable**: UX Researcher + QA | **Estimación**: 5 días

#### [ ] 6.2.1 Scripts de Testing con Usuarios

- [ ] **Archivo**: `tests/user_acceptance/seller_onboarding_script.md`
  ```markdown
  # Script UAT: Onboarding de Seller

  ## Perfil de Usuario: María (Boutique de Ropa)
  - Edad: 35 años
  - Experiencia: Vende por Facebook/Instagram
  - Objetivo: Expandir a marketplace online
  - Limitación: Tiempo limitado (30 min max)

  ## Escenario 1: Primera configuración
  **Objetivo**: Configurar tienda desde cero en menos de 10 minutos

  ### Tareas:
  1. **Registro inicial** (sin este documento)
     - "Imagina que acabas de crear tu cuenta"
     - "Queres configurar tu tienda para vender online"
     
  2. **Selección de tipo de negocio**
     - "¿Qué tipo de productos vendes principalmente?"
     - *Observar*: ¿Encuentra fácil la opción "Ropa y accesorios"?
     
  3. **Configuración de categorías**
     - "Te sugerimos estas categorías para tu tipo de negocio"
     - *Observar*: ¿Entiende qué significan las sugerencias?
     - *Observar*: ¿Personaliza los nombres de categorías?
     
  4. **Configuración de atributos**
     - "Estos atributos ayudan a que encuentren tus productos"
     - *Observar*: ¿Agrega atributos personalizados?
     - *Preguntar*: "¿Te resulta útil esta funcionalidad?"

  ## Métricas de Éxito:
  - [ ] Completa onboarding en < 10 minutos
  - [ ] Sin solicitar ayuda en pasos críticos
  - [ ] Expresa satisfacción (escala 1-5): >= 4
  - [ ] Entiende el valor diferencial vs competencia
  - [ ] Puede explicar cómo ayuda a su negocio

  ## Preguntas Post-Tarea:
  1. "¿Qué te resultó más confuso?"
  2. "¿Cambiarías algo del proceso?"
  3. "¿Te sentís lista para empezar a cargar productos?"
  4. "¿Recomendarías esto a otro vendedor?"
  ```

- [ ] **Archivo**: `tests/user_acceptance/product_creation_script.md`
- [ ] **Archivo**: `tests/user_acceptance/search_buyer_script.md`
- [ ] **Archivo**: `tests/user_acceptance/analytics_dashboard_script.md`

#### [ ] 6.2.2 Framework de Métricas UX

- [ ] **Archivo**: `tests/user_acceptance/ux_metrics_framework.js`
  ```javascript
  // tests/user_acceptance/ux_metrics_framework.js

  class UXMetricsCollector {
    constructor() {
      this.metrics = {
        taskCompletion: {},
        timeToComplete: {},
        errorRates: {},
        satisfactionScores: {},
        usabilityIssues: []
      };
    }

    // Métricas de completion rate
    recordTaskCompletion(taskId, userId, completed, timeSpent) {
      if (!this.metrics.taskCompletion[taskId]) {
        this.metrics.taskCompletion[taskId] = [];
      }
      
      this.metrics.taskCompletion[taskId].push({
        userId,
        completed,
        timeSpent,
        timestamp: new Date()
      });
    }

    // Métricas de eficiencia
    calculateTaskEfficiency(taskId) {
      const taskData = this.metrics.taskCompletion[taskId] || [];
      const completedTasks = taskData.filter(t => t.completed);
      
      return {
        completionRate: (completedTasks.length / taskData.length) * 100,
        averageTime: completedTasks.reduce((sum, t) => sum + t.timeSpent, 0) / completedTasks.length,
        medianTime: this.calculateMedian(completedTasks.map(t => t.timeSpent)),
        successfulInFirstAttempt: completedTasks.filter(t => t.timeSpent < 600).length // < 10 min
      };
    }

    // Métricas críticas para marketplace
    getMarketplaceCriticalMetrics() {
      return {
        onboardingSuccess: this.calculateTaskEfficiency('onboarding_complete'),
        productCreationEfficiency: this.calculateTaskEfficiency('create_first_product'),
        searchFindability: this.calculateTaskEfficiency('find_product_in_search'),
        categoryConfigUnderstanding: this.calculateTaskEfficiency('configure_categories'),
        
        // Métricas de satisfacción
        npsScore: this.calculateNPS(),
        taskSatisfactionAverage: this.calculateAverageTaskSatisfaction(),
        
        // Métricas de adopción
        featureAdoptionRate: this.calculateFeatureAdoption(),
        returnUserRate: this.calculateReturnUserRate()
      };
    }

    // Detección de friction points
    identifyFrictionPoints() {
      const frictionPoints = [];
      
      // Tareas con baja completion rate
      Object.keys(this.metrics.taskCompletion).forEach(taskId => {
        const efficiency = this.calculateTaskEfficiency(taskId);
        if (efficiency.completionRate < 80) {
          frictionPoints.push({
            type: 'low_completion',
            task: taskId,
            completionRate: efficiency.completionRate,
            severity: 'high'
          });
        }
        
        // Tareas que toman demasiado tiempo
        if (efficiency.averageTime > 600) { // > 10 minutos
          frictionPoints.push({
            type: 'slow_completion',
            task: taskId,
            averageTime: efficiency.averageTime,
            severity: 'medium'
          });
        }
      });
      
      return frictionPoints;
    }

    // Generar reporte de UX
    generateUXReport() {
      const criticalMetrics = this.getMarketplaceCriticalMetrics();
      const frictionPoints = this.identifyFrictionPoints();
      
      return {
        summary: {
          overallUsability: this.calculateOverallUsabilityScore(),
          criticalIssuesCount: frictionPoints.filter(f => f.severity === 'high').length,
          recommendationsCount: this.generateRecommendations().length
        },
        metrics: criticalMetrics,
        frictionPoints,
        recommendations: this.generateRecommendations(),
        rawData: this.metrics
      };
    }
  }

  // Automatización de testing UX
  class AutomatedUXTesting {
    async runFullUserJourney(userProfile) {
      const metrics = new UXMetricsCollector();
      
      // Journey: Seller nuevo completo
      if (userProfile.type === 'new_seller') {
        await this.testOnboardingJourney(metrics, userProfile);
        await this.testFirstProductCreation(metrics, userProfile);
        await this.testCategoryConfiguration(metrics, userProfile);
        await this.testAnalyticsDashboard(metrics, userProfile);
      }
      
      // Journey: Comprador buscando productos
      if (userProfile.type === 'buyer') {
        await this.testProductDiscovery(metrics, userProfile);
        await this.testSearchAndFilter(metrics, userProfile);
        await this.testProductComparison(metrics, userProfile);
      }
      
      return metrics.generateUXReport();
    }
  }
  ```

---

## 🚀 FASE 7: LANZAMIENTO Y MONITOREO (Semana 17)
**Responsable**: DevOps + Product Team | **Estimación**: 5 días

### 💡 ¿Por qué un lanzamiento gradual?

**Principio**: Reducir riesgo con rollout progresivo

**Semana 17**: 
- Días 1-2: Deploy a producción + monitoring básico
- Días 3-4: Invitar primeros 5 sellers beta
- Día 5: Evaluar métricas y decidir siguiente fase

### 7.1 Deployment y Configuración
**Responsable**: DevOps Engineer | **Estimación**: 2 días

#### [ ] 7.1.1 Pipeline de Deployment

- [ ] **Archivo**: `.github/workflows/deploy_marketplace.yml`
  ```yaml
  name: Deploy Marketplace Features

  on:
    push:
      branches: [main]
      paths: 
        - 'services/saas-mt-pim-service/**'
        - 'services/saas-mt-search-service/**'
        - 'services/saas-mt-analytics-service/**'
        - 'services/saas-mt-backoffice/**'

  jobs:
    test:
      runs-on: ubuntu-latest
      steps:
        - uses: actions/checkout@v3
        
        - name: Run marketplace tests
          run: |
            # Unit tests
            cd services/saas-mt-pim-service && go test ./...
            cd ../saas-mt-search-service && go test ./...
            cd ../saas-mt-analytics-service && go test ./...
            
            # Integration tests
            docker-compose -f docker-compose.test.yml up --build --abort-on-container-exit
            
            # E2E tests críticos
            cd services/saas-mt-backoffice && npm test -- --testPathPattern=marketplace

    deploy-staging:
      needs: test
      runs-on: ubuntu-latest
      steps:
        - name: Deploy to staging
          run: |
            # Migrations primero
            kubectl apply -f k8s/migrations/marketplace-migrations.yaml
            kubectl wait --for=condition=complete job/marketplace-migrations --timeout=300s
            
            # Deploy services
            kubectl apply -f k8s/services/
            kubectl rollout status deployment/pim-service
            kubectl rollout status deployment/search-service
            kubectl rollout status deployment/analytics-service
            
            # Deploy frontend
            kubectl apply -f k8s/backoffice/
            kubectl rollout status deployment/backoffice

        - name: Run smoke tests
          run: |
            # Verificar endpoints críticos
            curl -f https://staging.saas-mt.com/pim/api/v1/health
            curl -f https://staging.saas-mt.com/search/api/v1/health
            curl -f https://staging.saas-mt.com/analytics/api/v1/health

    deploy-production:
      needs: deploy-staging
      runs-on: ubuntu-latest
      if: github.ref == 'refs/heads/main'
      environment: production
      steps:
        - name: Blue-Green deployment
          run: |
            # Deploy a green environment
            kubectl apply -f k8s/production/green/
            
            # Run final verification
            ./scripts/verify_marketplace_functionality.sh
            
            # Switch traffic
            kubectl patch service main-service -p '{"spec":{"selector":{"version":"green"}}}'
            
            # Monitor for 5 minutes
            sleep 300
            ./scripts/check_error_rates.sh
  ```

- [ ] **Archivo**: `scripts/verify_marketplace_functionality.sh`
- [ ] **Archivo**: `scripts/check_error_rates.sh`
- [ ] **Archivo**: `k8s/services/marketplace-services.yaml`

#### [ ] 7.1.2 Configuración de Monitoreo

- [ ] **Archivo**: `monitoring/marketplace-alerts.yaml`
  ```yaml
  # monitoring/marketplace-alerts.yaml
  groups:
  - name: marketplace.rules
    rules:
    
    # Alertas críticas de funcionalidad
    - alert: OnboardingCompletionRateDropped
      expr: (onboarding_completions_total / onboarding_starts_total) * 100 < 70
      for: 10m
      labels:
        severity: critical
        component: onboarding
      annotations:
        summary: "Tasa de completado de onboarding bajó a {{ $value }}%"
        description: "Solo {{ $value }}% de sellers completan el onboarding vs 85% target"
        runbook_url: "https://runbooks.saas-mt.com/marketplace/onboarding-low-completion"

    - alert: SearchResponseTimeHigh
      expr: histogram_quantile(0.95, rate(search_request_duration_seconds_bucket[5m])) > 0.5
      for: 5m
      labels:
        severity: warning
        component: search
      annotations:
        summary: "Búsquedas responden en {{ $value }}s (target: <0.5s)"
        description: "95% de búsquedas están tardando más de 500ms"

    - alert: CategoryMappingErrors
      expr: rate(category_mapping_errors_total[5m]) > 0.1
      for: 2m
      labels:
        severity: critical
        component: pim
      annotations:
        summary: "Errores en mapeo de categorías"
        description: "{{ $value }} errores/min en mapeo de categorías"

    # Alertas de adopción
    - alert: LowTenantEngagement
      expr: avg_over_time(active_tenants_daily[7d]) < 5
      for: 1h
      labels:
        severity: warning
        component: analytics
      annotations:
        summary: "Baja participación de tenants"
        description: "Solo {{ $value }} tenants activos promedio en 7 días"

    # Alertas de performance
    - alert: HighElasticsearchLatency
      expr: elasticsearch_query_time_ms > 200
      for: 3m
      labels:
        severity: warning
        component: search
      annotations:
        summary: "Elasticsearch lento: {{ $value }}ms"
        description: "Queries a Elasticsearch tardando más de 200ms"

    - alert: DatabaseConnectionsHigh
      expr: sum(pg_stat_activity_count) by (instance) > 80
      for: 5m
      labels:
        severity: warning
        component: database
      annotations:
        summary: "Muchas conexiones DB: {{ $value }}"
        description: "Alto número de conexiones puede indicar leak"
  ```

- [ ] **Archivo**: `monitoring/marketplace-dashboard.json`
- [ ] **Archivo**: `monitoring/grafana-marketplace-dashboard.json`

### 7.2 Beta Testing con Usuarios Reales
**Responsable**: Product Manager | **Estimación**: 3 días

#### [ ] 7.2.1 Programa Beta Sellers

- [ ] **Archivo**: `docs/beta_program/seller_invitation_email.md`
  ```markdown
  # Invitación Beta Sellers - Marketplace SaaS-MT

  ¡Hola [NOMBRE]!

  Te invitamos a ser uno de los primeros sellers en probar nuestra nueva plataforma de marketplace.

  ## ¿Qué es diferente?
  - ⚡ **Setup en menos de 10 minutos** vs horas en otras plataformas
  - 🎯 **Categorías personalizables** vs taxonomías rígidas
  - 📊 **Analytics que realmente ayudan** vs reportes genéricos
  - 🤝 **Soporte directo del equipo** durante beta

  ## Tu perfil es perfecto porque:
  - Vendes [CATEGORIA] que es ideal para testear
  - Tienes experiencia en [PLATAFORMA_ACTUAL]
  - Nos puedes dar feedback valioso

  ## Cronograma Beta:
  - **Semana 1**: Onboarding + configuración inicial
  - **Semana 2**: Carga de primeros 10-20 productos
  - **Semana 3**: Testing de búsquedas y analytics
  - **Semana 4**: Feedback session + optimizaciones

  ## ¿Qué necesitamos de vos?
  - 2-3 horas por semana para testing
  - Feedback honesto sobre UX
  - Participar en 2 calls de 30 min c/u

  ## ¿Qué ganas?
  - Acceso gratuito durante 6 meses
  - Configuración prioritaria de tu tienda
  - Tu input influye en el producto final
  - Caso de estudio para marketing (opcional)

  **¿Te sumás?** Respondé este email con:
  1. ¿Cuántos productos tenés actualmente?
  2. ¿Qué te resulta más frustrante de [PLATAFORMA_ACTUAL]?
  3. ¿Qué día/hora te viene bien para kickoff call?

  ¡Gracias!
  [PRODUCT_MANAGER]
  ```

- [ ] **Archivo**: `docs/beta_program/success_metrics.md`
  ```markdown
  # Métricas de Éxito - Beta Program

  ## Métricas Críticas (Deben pasar para continuar)

  ### Onboarding Success Rate
  - **Target**: 90% de sellers completan onboarding
  - **Current**: ---%
  - **Blocker si**: < 70%

  ### Time to First Product
  - **Target**: < 15 minutos desde login hasta primer producto publicado
  - **Current**: --- minutos
  - **Blocker si**: > 30 minutos

  ### Search Findability
  - **Target**: 80% de productos aparecen en búsquedas relevantes
  - **Current**: ---%
  - **Blocker si**: < 60%

  ## Métricas de Calidad

  ### User Satisfaction (NPS)
  - **Target**: NPS > 50
  - **Current**: ---
  - **Método**: Survey post-onboarding + weekly check-ins

  ### Feature Adoption
  - **Categorías personalizadas**: ---% de sellers las usan
  - **Atributos custom**: ---% agregan al menos 1
  - **Analytics dashboard**: ---% lo revisan semanalmente

  ### Support Tickets
  - **Target**: < 2 tickets por seller durante beta
  - **Current**: ---
  - **Categorías**: Bugs, UX confusion, Feature requests

  ## Feedback Cualitativo

  ### Exit Interview Questions:
  1. "¿Recomendarías esta plataforma a otro seller?"
  2. "¿Qué es lo MEJOR vs tu plataforma actual?"
  3. "¿Qué es lo MÁS FRUSTRANTE?"
  4. "¿Pagarías [PRECIO] por esta herramienta?"
  5. "¿Qué feature agregarías?"

  ### Success Stories a Capturar:
  - Tiempo ahorrado en configuración
  - Mejora en descubrabilidad de productos
  - Insights útiles de analytics
  - Flexibilidad vs competencia
  ```

#### [ ] 7.2.2 Sistema de Feedback

- [ ] **Archivo**: `services/saas-mt-backoffice/src/components/feedback/BetaFeedbackWidget.jsx`
  ```jsx
  import React, { useState } from 'react';
  import { feedbackApi } from '../../../lib/api';

  const BetaFeedbackWidget = () => {
    const [isOpen, setIsOpen] = useState(false);
    const [feedback, setFeedback] = useState({
      type: 'general', // 'bug', 'feature', 'ux', 'general'
      rating: 0,
      title: '',
      description: '',
      context: {
        page: window.location.pathname,
        timestamp: new Date().toISOString(),
        userAgent: navigator.userAgent
      }
    });

    const handleSubmit = async (e) => {
      e.preventDefault();
      
      try {
        await feedbackApi.submitBetaFeedback({
          ...feedback,
          context: {
            ...feedback.context,
            screenshot: await captureScreenshot() // Optional
          }
        });
        
        setIsOpen(false);
        setFeedback({ type: 'general', rating: 0, title: '', description: '', context: feedback.context });
        
        // Show success message
        toast.success('¡Gracias por tu feedback! Lo revisaremos pronto.');
      } catch (error) {
        toast.error('Error enviando feedback. Intentá de nuevo.');
      }
    };

    const FeedbackForm = () => (
      <div className="feedback-form">
        <h3>Tu opinión nos ayuda 🙌</h3>
        
        <div className="feedback-type">
          <label>Tipo de feedback:</label>
          <select 
            value={feedback.type} 
            onChange={(e) => setFeedback({...feedback, type: e.target.value})}
          >
            <option value="general">Comentario general</option>
            <option value="bug">Encontré un bug 🐛</option>
            <option value="ux">Algo es confuso/difícil 😕</option>
            <option value="feature">Idea de mejora 💡</option>
          </select>
        </div>

        <div className="rating">
          <label>¿Cómo calificarías esta experiencia?</label>
          <div className="stars">
            {[1,2,3,4,5].map(star => (
              <button
                key={star}
                type="button"
                className={star <= feedback.rating ? 'active' : ''}
                onClick={() => setFeedback({...feedback, rating: star})}
              >
                ⭐
              </button>
            ))}
          </div>
        </div>

        <input
          type="text"
          placeholder="Título (ej: 'Onboarding es muy fácil' o 'Bug en carga de fotos')"
          value={feedback.title}
          onChange={(e) => setFeedback({...feedback, title: e.target.value})}
          required
        />

        <textarea
          placeholder="Contanos más detalles..."
          value={feedback.description}
          onChange={(e) => setFeedback({...feedback, description: e.target.value})}
          required
        />

        <div className="form-actions">
          <button type="button" onClick={() => setIsOpen(false)}>
            Cancelar
          </button>
          <button type="submit">
            Enviar Feedback
          </button>
        </div>
      </div>
    );

    return (
      <>
        <button 
          className="beta-feedback-trigger"
          onClick={() => setIsOpen(true)}
          data-testid="beta-feedback-button"
        >
          💬 Feedback
        </button>

        {isOpen && (
          <div className="feedback-modal">
            <div className="feedback-modal-content">
              <form onSubmit={handleSubmit}>
                <FeedbackForm />
              </form>
            </div>
          </div>
        )}
      </>
    );
  };

  export default BetaFeedbackWidget;
  ```

- [ ] **Archivo**: `services/saas-mt-backend/src/feedback/domain/entity/beta_feedback.go`
- [ ] **Archivo**: `services/saas-mt-backend/src/feedback/application/usecase/process_beta_feedback.go`
- [ ] **Archivo**: `services/saas-mt-backend/src/feedback/infrastructure/http/feedback_handler.go`

**Migration para feedback**:
- [ ] **Archivo**: `services/saas-mt-backend/migrations/018_create_beta_feedback.sql`
  ```sql
  CREATE TABLE beta_feedback (
      id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
      tenant_id UUID NOT NULL,
      user_id UUID,
      type VARCHAR(20) NOT NULL CHECK (type IN ('bug', 'feature', 'ux', 'general')),
      priority VARCHAR(20) DEFAULT 'medium' CHECK (priority IN ('low', 'medium', 'high', 'critical')),
      rating INTEGER CHECK (rating >= 1 AND rating <= 5),
      title VARCHAR(255) NOT NULL,
      description TEXT NOT NULL,
      context JSONB DEFAULT '{}',
      screenshot_url VARCHAR(500),
      status VARCHAR(20) DEFAULT 'open' CHECK (status IN ('open', 'in_progress', 'resolved', 'closed')),
      assigned_to VARCHAR(100),
      resolution_notes TEXT,
      created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
      updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
      resolved_at TIMESTAMP WITH TIME ZONE
  );

  CREATE INDEX idx_beta_feedback_tenant ON beta_feedback(tenant_id);
  CREATE INDEX idx_beta_feedback_type ON beta_feedback(type);
  CREATE INDEX idx_beta_feedback_status ON beta_feedback(status);
  CREATE INDEX idx_beta_feedback_priority ON beta_feedback(priority);
  CREATE INDEX idx_beta_feedback_created_at ON beta_feedback(created_at);
  ```

---

## 🎯 RESUMEN EJECUTIVO

### Cronograma Final
- **Semanas 1-4**: Arquitectura base + taxonomía global
- **Semanas 5-6**: Onboarding inteligente  
- **Semanas 7-9**: Motor de búsqueda híbrido
- **Semanas 10-12**: Backoffice completo
- **Semanas 13-14**: Analytics y optimización
- **Semanas 15-16**: Testing y validación
- **Semana 17**: Lanzamiento beta

### Entregables Clave
- ✅ **96 archivos** de código específicos
- ✅ **18 migraciones** de base de datos
- ✅ **47 endpoints** API nuevos
- ✅ **23 componentes** React nuevos
- ✅ **12 casos de prueba** E2E críticos

### ROI Esperado
- **70% reducción** en tiempo de setup para sellers
- **40% aumento** en descubrabilidad de productos
- **Diferenciación clara** vs MercadoLibre/Amazon
- **Base sólida** para monetización y crecimiento

**Hipótesis a validar**:
1. ¿Los sellers entienden el onboarding?
2. ¿La personalización es útil o confusa?
3. ¿Los compradores encuentran productos más fácil?

#### [ ] 6.1.1 Beta con 5 Sellers Reales

**Perfiles seleccionados**:
- **María (Boutique Bahía Blanca)**: Ropa femenina, 2 años vendiendo online
- **Marcos (Ferretería Río Cuarto)**: Herramientas, nuevo en digital
- **Ana (Accesorios Concordia)**: Bijouterie artesanal, vende en redes
- **Carlos (Electrónicos San Nicolás)**: Reparación y venta, cliente B2B
- **Laura (Hogar Comodoro)**: Decoración, venta estacional

**Tests específicos**:
1. **Onboarding completo**: Tiempo, errores, abandono
2. **Carga de productos**: ¿Encuentran sus atributos necesarios?
3. **Personalización**: ¿Usan funciones avanzadas?
4. **Satisfacción**: ¿Lo recomendarían vs alternativas?

---

## 🚀 FASE 7: LANZAMIENTO Y MONITOREO (Semana 17)

### 🎯 Lanzamiento Gradual

**¿Por qué no lanzar todo junto?**
- Riesgo controlado
- Feedback temprano
- Ajustes sobre la marcha

**Estrategia**:
- **Semana 1**: 10% sellers actuales (early adopters)
- **Semana 2**: 50% sellers actuales 
- **Semana 3**: 100% sellers actuales
- **Semana 4**: Nuevos registros con onboarding marketplace

**Métricas de éxito**:
- **Adoption rate**: >80% sellers usan categorías marketplace
- **Setup time**: <10 minutos promedio onboarding
- **Satisfaction**: NPS >8.0 vs sistema anterior

---

## 🎯 ROI Y JUSTIFICACIÓN DE ESFUERZO

### 💰 Análisis Costo-Beneficio

#### Esfuerzo de Implementación

**Total**: ~17 semanas (~4 meses) | **Equipo**: 2-3 desarrolladores

| Fase | Semanas | Esfuerzo | Riesgo |
|------|---------|----------|--------|
| Fundación | 4 | Alto | Bajo |
| Onboarding | 2 | Medio | Bajo |
| Búsqueda | 3 | Alto | Medio |
| Backoffice | 3 | Medio | Bajo |
| Analytics | 2 | Bajo | Bajo |
| Testing | 2 | Medio | Bajo |
| Lanzamiento | 1 | Bajo | Medio |

#### Beneficios Cuantificables

**Para Sellers**:
- **Time-to-market**: -70% tiempo configuración inicial
- **Discoverability**: +40% productos encontrados en búsquedas
- **Satisfaction**: De sistema "técnico" a "intuitivo"
- **Retention**: -50% abandono en onboarding

**Para Marketplace**:
- **User experience**: Navegación consistente cross-seller
- **Competitive advantage**: Flexibilidad vs rigidez de competidores
- **Scale**: Onboarding automático = crecimiento sostenible
- **Data quality**: Productos mejor categorizados = mejores recomendaciones

### 🏆 Comparativa vs Alternativas

#### Alternativa 1: Sistema Rígido (como MercadoLibre)
**Pros**: Implementación más simple  
**Contras**: 
- Alta fricción para sellers
- Productos mal categorizados
- Sin diferenciación competitiva
- Difícil evolución

#### Alternativa 2: Sistema Totalmente Flexible
**Pros**: Máxima personalización  
**Contras**:
- Complejidad extrema para sellers
- Navegación inconsistente para compradores
- Performance pobre en búsquedas
- Mantenimiento costoso

#### Nuestra Solución Híbrida
**Pros**:
- ✅ Simple para sellers (defaults inteligentes)
- ✅ Flexible cuando se necesita (escape hatches)
- ✅ Consistente para compradores (navegación marketplace)
- ✅ Escalable (onboarding automático)

**Contras**:
- Complejidad de implementación inicial
- Necesita datos de calidad para funcionar bien

---

## 📋 CHECKLIST FINAL DE VALIDACIÓN

### ✅ Validación de Valor para Sellers

- [ ] **Onboarding < 10 minutos**: ¿Un seller nuevo puede empezar a vender rápido?
- [ ] **Configuración sin fricción**: ¿Las opciones son claras y útiles?
- [ ] **Personalización práctica**: ¿Los sellers usan las funciones custom?
- [ ] **Impacto en ventas**: ¿La mejor categorización aumenta discoverability?

### ✅ Validación de UX para Compradores

- [ ] **Navegación consistente**: ¿Es fácil encontrar productos similares?
- [ ] **Filtros efectivos**: ¿Los filtros ayudan a refinar búsquedas?
- [ ] **Comparación útil**: ¿Se pueden comparar productos de diferentes sellers?
- [ ] **Performance**: ¿La búsqueda cross-tenant es rápida?

### ✅ Validación Técnica

- [ ] **Escalabilidad**: ¿Soporta 1000+ sellers con performance aceptable?
- [ ] **Maintainability**: ¿El código es sostenible a largo plazo?
- [ ] **Data consistency**: ¿El aislamiento multi-tenant se mantiene?
- [ ] **Monitoring**: ¿Las métricas permiten detectar problemas rápido?

---

## 🔄 EVOLUCIÓN POST-LANZAMIENTO

### Roadmap de Mejoras (6 meses)

#### Sprint 1-2: **Optimización Basada en Datos**
- Análisis de patrones de uso reales
- Optimización de categorías más usadas
- Mejora de recomendaciones de onboarding

#### Sprint 3-4: **Funcionalidades Avanzadas**
- **AI-powered categorization**: Sugerir categorías basado en descripción
- **Smart attributes**: Detectar atributos faltantes importantes
- **Bulk operations**: Operaciones masivas para sellers grandes

#### Sprint 5-6: **Ecosystem Expansion**
- **API para terceros**: Integración con ERPs locales
- **Mobile marketplace**: App para compradores
- **Advanced analytics**: Predictive insights para sellers

---

*💡 **Filosofía del Proyecto**: "Hacer que vender online sea tan simple como vender en un local físico, pero con el alcance de internet"*

*🎯 **Objetivo Último**: Que un vendedor como María o Marcos pueda configurar su tienda online en el tiempo que tarda en tomar un café, y que funcione perfectamente para su negocio desde el primer día* 