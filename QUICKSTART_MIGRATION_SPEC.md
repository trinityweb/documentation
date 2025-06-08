# 🚀 QUICKSTART MIGRATION SPECIFICATION

## 🎯 Objetivo
Migrar el sistema quickstart existente de archivos YAML estáticos a base de datos dinámica con administración completa desde backoffice marketplace.

## 📊 Estado Actual vs Target

### 🟠 Estado Actual (YAML-based)
```yaml
# Estructura actual:
services/saas-mt-pim-service/src/quickstart/data/
├── business-types.yaml          # 15 tipos hardcodeados
├── categories/retail.yaml       # ~800 categorías por tipo
├── attributes/retail.yaml       # ~200 atributos por tipo  
├── variants/retail.yaml         # Configuraciones variantes
├── products/retail.yaml         # Productos de ejemplo
└── brands/retail.yaml          # Marcas sugeridas
```

**❌ Limitaciones Actuales:**
- Requiere redeploy para cambios
- No analytics de uso
- No personalización por región
- Mantenimiento manual complejo
- No A/B testing de configuraciones

### 🟢 Target (BD + Admin)
```sql
-- Estructura target:
business_types                   # Tipos dinámicos
quickstart_templates            # Templates configurables
quickstart_categories          # Categorías por template
quickstart_attributes          # Atributos por template
tenant_quickstart_history      # Tracking de uso
quickstart_analytics           # Métricas de adopción
```

**✅ Beneficios Esperados:**
- Configuración sin redeploy
- Analytics y optimización
- Personalización avanzada
- A/B testing de templates
- Rollback instantáneo

---

## 🏗️ ARQUITECTURA TÉCNICA

### 📋 Entidades de Dominio

#### 1. BusinessType (Dinámico)
```go
// AI-TODO: services/saas-mt-pim-service/src/quickstart/domain/entity/business_type.go
type BusinessType struct {
    ID          string    `json:"id"`
    Name        string    `json:"name"`
    Description string    `json:"description"`
    Icon        string    `json:"icon"`
    Active      bool      `json:"active"`
    SortOrder   int       `json:"sort_order"`
    CreatedAt   time.Time `json:"created_at"`
    UpdatedAt   time.Time `json:"updated_at"`
}
```

#### 2. QuickstartTemplate 
```go
// AI-TODO: services/saas-mt-pim-service/src/quickstart/domain/entity/quickstart_template.go
type QuickstartTemplate struct {
    ID             string                 `json:"id"`
    BusinessTypeID string                 `json:"business_type_id"`
    Name           string                 `json:"name"`
    Version        string                 `json:"version"`
    Description    string                 `json:"description"`
    Categories     []TemplateCategory     `json:"categories"`
    Attributes     []TemplateAttribute    `json:"attributes"`
    Variants       []TemplateVariant      `json:"variants"`
    Products       []TemplateProduct      `json:"products"`
    Brands         []TemplateBrand        `json:"brands"`
    Active         bool                   `json:"active"`
    CreatedAt      time.Time              `json:"created_at"`
    UpdatedAt      time.Time              `json:"updated_at"`
}
```

#### 3. TenantQuickstartHistory (Analytics)
```go
// AI-TODO: services/saas-mt-pim-service/src/quickstart/domain/entity/tenant_quickstart_history.go
type TenantQuickstartHistory struct {
    ID               string               `json:"id"`
    TenantID         string               `json:"tenant_id"`
    BusinessTypeID   string               `json:"business_type_id"`
    TemplateID       string               `json:"template_id"`
    TemplateVersion  string               `json:"template_version"`
    SetupData        SetupData            `json:"setup_data"`
    Status           SetupStatus          `json:"status"`
    StartedAt        time.Time            `json:"started_at"`
    CompletedAt      *time.Time           `json:"completed_at"`
    SetupTimeSeconds int                  `json:"setup_time_seconds"`
    CreatedAt        time.Time            `json:"created_at"`
}
```

### 🗄️ Esquemas de Base de Datos

#### Migration 001: Business Types
```sql
-- AI-TODO: services/saas-mt-pim-service/migrations/020_create_business_types.sql
CREATE TABLE business_types (
    id VARCHAR(50) PRIMARY KEY,
    name VARCHAR(255) NOT NULL,
    description TEXT,
    icon VARCHAR(100),
    active BOOLEAN DEFAULT true,
    sort_order INTEGER DEFAULT 0,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

CREATE INDEX idx_business_types_active ON business_types(active);
CREATE INDEX idx_business_types_sort_order ON business_types(sort_order);
```

#### Migration 002: Quickstart Templates
```sql
-- AI-TODO: services/saas-mt-pim-service/migrations/021_create_quickstart_templates.sql
CREATE TABLE quickstart_templates (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    business_type_id VARCHAR(50) REFERENCES business_types(id),
    name VARCHAR(255) NOT NULL,
    version VARCHAR(20) DEFAULT '1.0.0',
    description TEXT,
    template_data JSONB NOT NULL, -- Categories, attributes, etc.
    active BOOLEAN DEFAULT true,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

CREATE INDEX idx_quickstart_templates_business_type ON quickstart_templates(business_type_id);
CREATE INDEX idx_quickstart_templates_active ON quickstart_templates(active);
```

#### Migration 003: Tenant History
```sql
-- AI-TODO: services/saas-mt-pim-service/migrations/022_create_tenant_quickstart_history.sql
CREATE TABLE tenant_quickstart_history (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID NOT NULL,
    business_type_id VARCHAR(50) REFERENCES business_types(id),
    template_id UUID REFERENCES quickstart_templates(id),
    template_version VARCHAR(20),
    setup_data JSONB NOT NULL,
    status VARCHAR(20) DEFAULT 'STARTED',
    started_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    completed_at TIMESTAMP WITH TIME ZONE,
    setup_time_seconds INTEGER,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

CREATE INDEX idx_tenant_quickstart_tenant ON tenant_quickstart_history(tenant_id);
CREATE INDEX idx_tenant_quickstart_status ON tenant_quickstart_history(status);
```

---

## 🔄 MIGRACIÓN DE DATOS

### 📥 Script de Migración YAML → BD
```go
// AI-TODO: services/saas-mt-pim-service/src/quickstart/infrastructure/migration/yaml_to_db_migrator.go
type YamlToDbMigrator struct {
    businessTypeRepo BusinessTypeRepository
    templateRepo     QuickstartTemplateRepository
    yamlLoader       YamlDataLoader
}

func (m *YamlToDbMigrator) MigrateAllData() error {
    // 1. Migrar business types desde business-types.yaml
    // 2. Para cada business type, migrar sus configuraciones
    // 3. Crear templates con versioning
    // 4. Validar integridad de datos
    // 5. Marcar migración como completada
}
```

### 📊 Comando de Migración
```bash
# AI-TODO: Script ejecutable desde CLI
go run cmd/migrate-quickstart/main.go \
  --source-path="src/quickstart/data" \
  --dry-run=false \
  --backup=true \
  --validate=true
```

---

## 🎨 ADMINISTRACIÓN EN BACKOFFICE

### 🖥️ Páginas del Admin Panel

#### 1. Business Types Management
```tsx
// AI-TODO: backoffice/src/pages/admin/quickstart/BusinessTypesAdmin.tsx
const BusinessTypesAdmin = () => {
  // CRUD completo para business types
  // - Lista con filtros y búsqueda
  // - Modal crear/editar con validaciones
  // - Drag & drop para reordering
  // - Toggle activar/desactivar
  // - Preview del tipo en wizard
};
```

#### 2. Template Editor
```tsx
// AI-TODO: backoffice/src/pages/admin/quickstart/TemplateEditor.tsx
const TemplateEditor = () => {
  // Editor visual de templates:
  // - Selector de business type
  // - Editor de categorías (tree view)
  // - Editor de atributos (con tipos y validaciones)
  // - Preview de productos sugeridos
  // - Sistema de versioning
  // - Validation en tiempo real
};
```

#### 3. Analytics Dashboard
```tsx
// AI-TODO: backoffice/src/pages/admin/quickstart/QuickstartAnalytics.tsx
const QuickstartAnalytics = () => {
  // Métricas de uso:
  // - Business types más usados
  // - Templates con mayor éxito
  // - Tiempo promedio de setup
  // - Tasa de abandono por paso
  // - Heat map de selecciones
};
```

### 🧩 Componentes Reutilizables

#### 1. Business Type Card
```tsx
// AI-TODO: backoffice/src/components/quickstart/BusinessTypeCard.tsx
interface BusinessTypeCardProps {
  businessType: BusinessType;
  onEdit: (id: string) => void;
  onToggleActive: (id: string) => void;
  onPreview: (id: string) => void;
}
```

#### 2. Template Preview
```tsx
// AI-TODO: backoffice/src/components/quickstart/TemplatePreview.tsx
interface TemplatePreviewProps {
  template: QuickstartTemplate;
  mode: 'admin' | 'customer'; // Vista admin vs customer
  onSimulateSetup?: () => void;
}
```

#### 3. Category Tree Editor
```tsx
// AI-TODO: backoffice/src/components/quickstart/CategoryTreeEditor.tsx
interface CategoryTreeEditorProps {
  categories: TemplateCategory[];
  onChange: (categories: TemplateCategory[]) => void;
  validation?: ValidationRules;
}
```

---

## 🔌 APIS Y ENDPOINTS

### 🌐 Admin APIs (Backend)

#### Business Types CRUD
```go
// AI-TODO: services/saas-mt-pim-service/src/quickstart/infrastructure/controller/admin_business_types_controller.go
// GET    /api/v1/admin/quickstart/business-types
// POST   /api/v1/admin/quickstart/business-types
// GET    /api/v1/admin/quickstart/business-types/:id
// PUT    /api/v1/admin/quickstart/business-types/:id
// DELETE /api/v1/admin/quickstart/business-types/:id
// POST   /api/v1/admin/quickstart/business-types/:id/toggle-active
```

#### Templates CRUD
```go
// AI-TODO: services/saas-mt-pim-service/src/quickstart/infrastructure/controller/admin_templates_controller.go
// GET    /api/v1/admin/quickstart/templates
// POST   /api/v1/admin/quickstart/templates
// GET    /api/v1/admin/quickstart/templates/:id
// PUT    /api/v1/admin/quickstart/templates/:id
// DELETE /api/v1/admin/quickstart/templates/:id
// POST   /api/v1/admin/quickstart/templates/:id/publish
// POST   /api/v1/admin/quickstart/templates/:id/rollback
```

#### Analytics APIs
```go
// AI-TODO: services/saas-mt-pim-service/src/quickstart/infrastructure/controller/admin_analytics_controller.go
// GET    /api/v1/admin/quickstart/analytics/usage
// GET    /api/v1/admin/quickstart/analytics/business-types
// GET    /api/v1/admin/quickstart/analytics/templates
// GET    /api/v1/admin/quickstart/analytics/setup-times
```

### 🛡️ Refactoring APIs Existentes

#### Compatibility Layer
```go
// AI-TODO: services/saas-mt-pim-service/src/quickstart/infrastructure/service/compatibility_service.go
type CompatibilityService struct {
    // Mantener APIs existentes funcionando
    // Redireccionar a nueva implementación BD
    // Cache para performance
}

// APIs que deben seguir funcionando:
// GET /api/v1/quickstart/business-types (now from DB)
// GET /api/v1/quickstart/categories/:businessType (now from DB)  
// GET /api/v1/quickstart/attributes/:businessType (now from DB)
// POST /api/v1/quickstart/setup (enhanced with history tracking)
```

---

## 🧪 TESTING Y VALIDACIÓN

### ✅ Test Coverage Requerido

#### 1. Unit Tests
```go
// AI-TODO: services/saas-mt-pim-service/src/quickstart/domain/entity/business_type_test.go
// AI-TODO: services/saas-mt-pim-service/src/quickstart/application/usecase/create_business_type_test.go
// AI-TODO: services/saas-mt-pim-service/src/quickstart/infrastructure/repository/business_type_repository_test.go
```

#### 2. Integration Tests
```go
// AI-TODO: services/saas-mt-pim-service/src/quickstart/integration/migration_test.go
// AI-TODO: services/saas-mt-pim-service/src/quickstart/integration/api_compatibility_test.go
```

#### 3. E2E Tests
```typescript
// AI-TODO: backoffice/tests/e2e/quickstart-admin.spec.ts
describe('Quickstart Admin Management', () => {
  it('should create and edit business types', () => {});
  it('should migrate YAML data correctly', () => {});
  it('should maintain API compatibility', () => {});
});
```

### 🎯 Scenarios de Testing

#### Migration Scenarios
```sql
-- AI-TODO: Test data verification
-- Verificar que todos los business types YAML se migraron
-- Verificar que todas las categorías se mantuvieron
-- Verificar que jerarquías están intactas
-- Verificar que APIs devuelven mismos datos
```

---

## 📋 CHECKLIST DE IMPLEMENTACIÓN

### 🏗️ Backend (Go Service)

#### Fase 2.0: Migración Quickstart a BD

- [ ] **AI-TODO** Crear entidades de dominio
  - [ ] `business_type.go` con validaciones
  - [ ] `quickstart_template.go` con versioning
  - [ ] `tenant_quickstart_history.go` para analytics

- [ ] **AI-TODO** Implementar repositorios
  - [ ] `business_type_repository.go` con CRUD
  - [ ] `quickstart_template_repository.go` con versioning
  - [ ] `tenant_history_repository.go` con agregaciones

- [ ] **AI-TODO** Crear migraciones de BD
  - [ ] `020_create_business_types.sql`
  - [ ] `021_create_quickstart_templates.sql`
  - [ ] `022_create_tenant_quickstart_history.sql`

- [ ] **AI-TODO** Script migración datos
  - [ ] `yaml_to_db_migrator.go` con validaciones
  - [ ] Command line tool para migración
  - [ ] Backup automático antes de migrar

- [ ] **AI-TODO** Refactor service layer
  - [ ] Mantener `QuickstartService` compatible
  - [ ] Nuevo `DatabaseQuickstartService`
  - [ ] Cache layer con Redis
  - [ ] Compatibility layer para APIs existentes

- [ ] **AI-TODO** Testing completo
  - [ ] Unit tests para todas las entidades
  - [ ] Integration tests para migración
  - [ ] Performance tests para cache
  - [ ] API compatibility tests

### 🎨 Frontend (Admin Backoffice)

#### Fase 4.0: Admin Panel Quickstart

- [ ] **AI-TODO** Páginas principales
  - [ ] `BusinessTypesAdmin.tsx` con CRUD completo
  - [ ] `TemplateEditor.tsx` con editor visual
  - [ ] `QuickstartAnalytics.tsx` con métricas

- [ ] **AI-TODO** Componentes reutilizables  
  - [ ] `BusinessTypeCard.tsx` para gestión
  - [ ] `TemplatePreview.tsx` para visualización
  - [ ] `CategoryTreeEditor.tsx` para edición jerárquica

- [ ] **AI-TODO** APIs y servicios frontend
  - [ ] `quickstart-admin.api.ts` con todas las operaciones
  - [ ] `template-editor.service.ts` para editor
  - [ ] `analytics.service.ts` para métricas

- [ ] **AI-TODO** UX/UI optimizations
  - [ ] Drag & drop para reordering
  - [ ] Real-time validation
  - [ ] Preview mode customer
  - [ ] Responsive design

- [ ] **AI-TODO** Testing frontend
  - [ ] Component tests con Jest
  - [ ] E2E tests con Playwright
  - [ ] Accessibility tests
  - [ ] Performance tests

---

## 🎯 ACCEPTANCE CRITERIA

### ✅ Criterios Técnicos

1. **Performance**: 
   - APIs responden <200ms (con cache)
   - Migración completa <5 minutos
   - Zero downtime durante migración

2. **Compatibility**:
   - APIs existentes funcionan sin cambios
   - Datos migrados 100% íntegros
   - Rollback disponible en <1 minuto

3. **Testing**:
   - Coverage >90% en nuevos componentes
   - E2E tests pasan 100%
   - Performance tests dentro de targets

### ✅ Criterios Funcionales

1. **Admin Experience**:
   - Product manager puede crear business type <10 min
   - Template editor intuitivo sin training
   - Preview mode refleja customer experience exacto

2. **Customer Experience**:
   - Setup experience idéntico (transparente)
   - Mismo performance pre/post migración
   - Mismos datos disponibles

3. **Analytics**:
   - Tracking completo de uso
   - Métricas de abandono por paso
   - Comparación A/B de templates

### ✅ Criterios de Negocio

1. **ROI**:
   - Tiempo setup nuevo tipo: 2 días → 30 min
   - Flexibilidad: 15 estáticos → ilimitados dinámicos
   - Maintenance: Dev required → PM self-service

2. **Risk Mitigation**:
   - Zero customer impact durante migración
   - Rollback automático si fallas
   - Monitoring completo post-migración

---

## 🔄 ROLLOUT PLAN

### 🚀 Plan de Despliegue

#### Fase 1: Preparación (Semana 1)
- [ ] **AI-TODO** Deploy migración BD en staging
- [ ] **AI-TODO** Testing exhaustivo en staging
- [ ] **AI-TODO** Backup completo de datos actuales
- [ ] **AI-TODO** Plan de rollback validado

#### Fase 2: Migración Datos (Semana 2)
- [ ] **AI-TODO** Ejecutar migración en horario de baja demanda
- [ ] **AI-TODO** Validación automática post-migración
- [ ] **AI-TODO** Switch gradual a APIs BD (feature flag)
- [ ] **AI-TODO** Monitoring continuo de performance

#### Fase 3: Admin Panel (Semana 3)
- [ ] **AI-TODO** Deploy admin panel para internal testing
- [ ] **AI-TODO** Training product management team
- [ ] **AI-TODO** Documentation y tutorials
- [ ] **AI-TODO** Go-live con business types admin

#### Fase 4: Validación (Semana 4)
- [ ] **AI-TODO** Crear primer business type nuevo desde admin
- [ ] **AI-TODO** A/B testing de templates optimizados
- [ ] **AI-TODO** Análisis de métricas y optimizaciones
- [ ] **AI-TODO** Documentation final y cierre

---

## 📊 SUCCESS METRICS

### 📈 Métricas de Adopción
- **Target**: 100% business types migrados sin pérdida de datos
- **Target**: <1% regresión en performance APIs quickstart
- **Target**: Admin puede crear business type completo <30 min

### 📊 Métricas Operacionales
- **Target**: Zero downtime durante migración
- **Target**: <5% aumento en uso de memoria/CPU
- **Target**: Admin panel usable sin training (UX testing)

### 🎯 Métricas de Negocio
- **Target**: 70% reducción tiempo crear nuevo business type
- **Target**: Capacidad de optimizar templates basado en analytics
- **Target**: Foundation para features marketplace avanzadas

---

## 🔗 REFERENCIAS Y LINKS

### 📚 Documentación Existente
- [Quickstart Module Documentation](../services/saas-mt-pim-service/documentation/quickstart-module.md)
- [Current YAML Structure](../services/saas-mt-pim-service/src/quickstart/data/)
- [PROJECT_TRACKING.md](./PROJECT_TRACKING.md)
- [MARKETPLACE_MULTI_TENANT_ROADMAP.md](./MARKETPLACE_MULTI_TENANT_ROADMAP.md)

### 🛠️ Código Existente
- [QuickstartService](../services/saas-mt-pim-service/src/quickstart/domain/service/quickstart_service.go)
- [Current APIs](../services/saas-mt-pim-service/src/quickstart/infrastructure/controller/)
- [YAML Data](../services/saas-mt-pim-service/src/quickstart/data/)

### 📋 Tracking
- **Epic**: FASE 2.0 + FASE 4.0 en PROJECT_TRACKING.md
- **Total Tasks**: +11 tareas agregadas al roadmap
- **Priority**: High (foundation para marketplace)

---

*Última actualización: 2024-12-08*  
*Documento mantenido por: AI Agent + Human Review*  
*Status: ✅ Ready for Implementation* 