# 🚀 Onboarding Simplificado - Resumen Ejecutivo

> **Flujo optimizado de 5 pasos para configurar una tienda en menos de 10 minutos**

## 🎯 Objetivo

Crear un proceso de onboarding **dividido en dos fases**:

1. **🚀 Onboarding Inicial** (3-5 min): Registro mínimo para generar engagement
2. **⚙️ Configuración Completa** (Backoffice): Setup detallado paso a paso

**Filosofía**: Conseguir el registro rápido, luego guiar la configuración completa con gamificación.

## 📊 Métricas Objetivo

- ⏱️ **Tiempo total**: < 10 minutos
- 📈 **Tasa de completación**: > 85%
- 🎯 **Abandono por paso**: < 15%
- ✅ **Time-to-first-product**: < 30 minutos

## 🔄 FASE 1: Onboarding Inicial (Landing → Registro)

> **Objetivo**: Conseguir que se registre en 3-5 minutos máximo

### Paso 1: 🏠 Bienvenida
**Pantalla**: Landing de bienvenida
```
┌─────────────────────────────────────┐
│           🎉 Bienvenido a tu        │
│             nueva tienda            │
│                                     │
│  Comienza a configurar tu tienda    │
│     en unos simples pasos.          │
│                                     │
│         [ Comenzar ]                │
└─────────────────────────────────────┘
```

**Objetivo**: Generar expectativa positiva y dar contexto del proceso.

---

### Paso 2: 👤 Datos de Usuario
**Pantalla**: Registro básico del administrador

```
┌─────────────────────────────────────┐
│    Paso 2 de 5    ●●○○○             │
│                                     │
│         Nombre                      │
│    [________________]               │
│                                     │
│      Correo electrónico             │
│    [________________]               │
│                                     │
│    Debe contener al menos 8 caracteres│
│                                     │
│     Confirmar contraseña            │
│    [••••••••••••••]                │
│                                     │
│     Confirmar contraseña            │
│    [••••••••••••••]                │
│                                     │
│         [ Siguiente ]               │
└─────────────────────────────────────┘
```

**Campos**:
- Nombre completo
- Correo electrónico
- Contraseña (con validación visual)
- Confirmar contraseña

**Validaciones**:
- Email válido
- Contraseña mínimo 8 caracteres
- Confirmación coincidente

---

### Paso 3: ✉️ Verificación de Email
**Pantalla**: Verificación por código

```
┌─────────────────────────────────────┐
│    Paso 3 de 5    ●●●○○             │
│                                     │
│         📧 Verify Your Email        │
│                                     │
│   Please enter the 6-digit          │
│   verification code that was        │
│   sent to:                          │
│                                     │
│     example@example.com             │
│                                     │
│     Verification Code               │
│    [____][____][____]               │
│    [____][____][____]               │
│                                     │
│         [ Continue ]                │
│                                     │
│      Don't receive the code?        │
└─────────────────────────────────────┘
```

**Funcionalidad**:
- Código de 6 dígitos enviado por email
- Opción "reenviar código" 
- Validación automática al completar

---

### Paso 4: 🏪 Configuración Inicial del Catálogo
**Pantalla**: Configuración básica del negocio y selección de categorías

```
┌─────────────────────────────────────┐
│    Paso 4 de 5    ●●●●○             │
│                                     │
│      Configuración de tu Tienda     │
│                                     │
│     Nombre de la tienda             │
│    [________________]               │
│                                     │
│     Seleccionar rubro               │
│    [ Hogar y Construcción  ▼ ]      │
│                                     │
│           Tamaño                    │
│   🏪        🏢        🏬           │
│  Micro      PYME     Varias         │
│ emprendimiento     Sucursales       │
│                                     │
│    Categorías principales (min 1)   │
│   ☑ Pinturas      ☑ Herramientas   │
│   ☐ Cerrajería    ☑ Electricidad   │
│   ☐ Plomería      ☐ Jardinería     │
│                                     │
│         [ Siguiente ]               │
└─────────────────────────────────────┘
```

**Campos**:
- **Nombre de la tienda**: Texto libre
- **Rubro**: Dropdown sincronizado con PIM Service
- **Tamaño**: Selector visual con 3 opciones
- **Categorías principales**: Checkbox múltiple (mínimo 1, dinámico según rubro)

**Rubros disponibles** (sincronizados con PIM Service):

| Onboarding UI | PIM business_type | Nombre PIM |
|---------------|------------------|------------|
| Pinturería | `home-construction` | Hogar y Construcción |
| Ferretería | `home-construction` | Hogar y Construcción |
| Ropa y accesorios | `fashion` | Moda y Vestimenta |
| Electrónicos | `electronics` | Electrónicos y Tecnología |
| Repuestos automotriz | `automotive` | Automotriz y Repuestos |
| Libros y papelería | `books-media` | Libros y Medios |
| Alimentos y bebidas | `food-beverage` | Alimentos y Bebidas |
| Salud y farmacia | `health-pharmacy` | Salud y Farmacia |
| Deportes | `sports-fitness` | Deportes y Fitness |
| Belleza | `beauty-cosmetics` | Belleza y Cosméticos |
| Juguetes | `toys-games` | Juguetes y Juegos |
| Mascotas | `pet-supplies` | Mascotas y Suministros |
| Oficina | `office-supplies` | Oficina y Papelería |
| Joyería | `jewelry-accessories` | Joyería y Accesorios |
| Polirubro | `polirubro` | Polirubro |

> **Nota**: Los business types se obtienen dinámicamente desde PIM Service para garantizar sincronización.

---

### Paso 5: 🎉 Bienvenida al Backoffice
**Pantalla**: Redirigir al dashboard con onboarding pendiente

```
┌─────────────────────────────────────┐
│         🎉 ¡Bienvenido!             │
│                                     │
│    ¡Tu cuenta está creada!          │
│     Ahora configuremos tu tienda    │
│                                     │
│     [ Ir al Panel de Control ]     │
│                                     │
│   💡 Tip: Configura tu tienda en    │
│      menos de 10 minutos            │
└─────────────────────────────────────┘
```

**Acción**: Redirigir a `/backoffice/onboarding-setup`

---

## 🚀 Próximos Pasos

Una vez completado el onboarding inicial de 5 pasos, el usuario será redirigido al **backoffice** donde podrá completar la configuración avanzada de su tienda con funcionalidades como:

- 📝 Información de la empresa
- 🏢 Perfil operativo  
- 📍 Configuración de ubicaciones
- 🎨 Personalización de marca
- 📦 Catálogo inicial

> **Nota**: La configuración completa del backoffice se diseñará e implementará en una fase posterior.


## 🗄️ Base de Datos

### Tabla: `onboarding_step_definitions` (Master data - Definición de pasos)
```sql
CREATE TABLE onboarding_step_definitions (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    step_number INT NOT NULL UNIQUE, -- 1, 2, 3, 4, 5
    step_name VARCHAR(255) NOT NULL, -- 'bienvenida', 'registro', 'verificacion', etc.
    step_title VARCHAR(255) NOT NULL, -- 'Bienvenida', 'Datos de Usuario', etc.
    description TEXT,
    is_required BOOLEAN DEFAULT TRUE,
    
    -- Configuración del paso
    has_ui BOOLEAN DEFAULT TRUE, -- false para pasos automáticos
    requires_user_input BOOLEAN DEFAULT TRUE,
    can_be_skipped BOOLEAN DEFAULT FALSE,
    
    -- Orden y agrupación
    display_order INT NOT NULL,
    is_active BOOLEAN DEFAULT TRUE,
    
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW()
);
```

### Tabla: `onboarding_processes`
```sql
CREATE TABLE onboarding_processes (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID NOT NULL,
    user_id UUID NOT NULL,
    
    -- Estado del proceso
    current_step_number INT DEFAULT 1,
    is_completed BOOLEAN DEFAULT FALSE,
    
    -- Datos capturados durante el onboarding
    company_name VARCHAR(255),
    business_type VARCHAR(100), -- 'pintureria', 'ferreteria', etc.
    store_size VARCHAR(50), -- 'micro', 'pyme', 'multiple'
    
    -- Control de progreso
    steps_completed JSONB DEFAULT '[]', -- [1, 2, 3]
    steps_pending JSONB DEFAULT '[1,2,3,4,5]', -- [4, 5]
    steps_skipped JSONB DEFAULT '[]', -- []
    
    -- Timing
    started_at TIMESTAMP DEFAULT NOW(),
    completed_at TIMESTAMP NULL,
    
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW()
);
```

### Tabla: `onboarding_step_progress` (Progreso individual de cada paso)
```sql
CREATE TABLE onboarding_step_progress (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    process_id UUID NOT NULL REFERENCES onboarding_processes(id),
    step_definition_id UUID NOT NULL REFERENCES onboarding_step_definitions(id),
    
    -- Estado del paso
    status VARCHAR(50) DEFAULT 'pending', -- 'pending', 'in_progress', 'completed', 'skipped', 'failed'
    
    -- Datos específicos del paso
    step_data JSONB, -- datos variables según el paso
    
    -- Timing individual
    started_at TIMESTAMP NULL,
    completed_at TIMESTAMP NULL,
    time_spent_seconds INT NULL,
    
    -- Errores/intentos
    attempts_count INT DEFAULT 0,
    last_error TEXT NULL,
    
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW(),
    
    UNIQUE(process_id, step_definition_id)
);
```

### 📋 Datos Iniciales (Seed Data)

#### Steps Definition:
```sql
INSERT INTO onboarding_step_definitions (step_number, step_name, step_title, description, has_ui, requires_user_input, display_order) VALUES
(1, 'bienvenida', 'Bienvenida', 'Pantalla de bienvenida al onboarding', true, false, 1),
(2, 'registro', 'Datos de Usuario', 'Captura nombre, email y contraseña', true, true, 2),
(3, 'verificacion', 'Verificar Email', 'Validación del código de verificación', true, true, 3),
(4, 'configuracion_tienda', 'Detalles de la Tienda', 'Configuración básica del negocio', true, true, 4),
(5, 'completar', 'Finalizar', 'Completar onboarding y redirección', true, false, 5);
```

### 🔗 Business Types desde PIM Service

> **Nota**: Los `business_types` NO se almacenan localmente. Se obtienen dinámicamente desde el **PIM Service** que ya tiene toda la configuración.

#### Endpoint PIM Service:
```bash
GET /api/v1/quickstart/business-types
```

#### Respuesta esperada:
```json
{
  "business_types": [
    {
      "id": "retail",
      "name": "Comercio Minorista", 
      "description": "Tiendas de venta al por menor",
      "icon": "store"
    },
    {
      "id": "pintureria",
      "name": "Pinturería",
      "description": "Venta de pinturas y accesorios", 
      "icon": "🎨"
    },
    {
      "id": "ferreteria", 
      "name": "Ferretería",
      "description": "Herramientas y materiales de construcción",
      "icon": "🔧"
    }
  ]
}
```

#### En el paso 4 del onboarding:
```go
// SetupStoreUseCase obtiene los business types del PIM Service y aplica configuración
func (uc *SetupStoreUseCase) Execute(req SetupStoreRequest) (*SetupStoreResponse, error) {
    // 1. Obtener tipos de negocio disponibles
    businessTypes, err := uc.pimClient.GetBusinessTypes()
    if err != nil {
        return nil, err
    }
    
    // 2. Actualizar datos en el proceso de onboarding
    process := uc.onboardingRepo.GetByID(req.ProcessID)
    process.CompanyName = req.StoreName
    process.BusinessType = req.BusinessType
    process.StoreSize = req.StoreSize
    
    // 3. Recomendar plan basado en store_size
    recommendedPlan := uc.determinePlanBySize(req.StoreSize)
    
    // 4. Guardar configuración para usar después en el backoffice
    uc.onboardingRepo.Save(process)
    
    return &SetupStoreResponse{
        StoreConfigured: true,
        BusinessTypeApplied: req.BusinessType,
        StoreSizeApplied: req.StoreSize,
        PlanRecommended: recommendedPlan,
        NextStep: 5,
    }, nil
}

func (uc *SetupStoreUseCase) determinePlanBySize(storeSize string) string {
    switch storeSize {
    case "micro":
        return "basic"
    case "pyme": 
        return "professional"
    case "multiple":
        return "enterprise"
    default:
        return "basic"
    }
}
```

#### En el backoffice (FUTURO):
```go
// Cuando el usuario llegue al backoffice, se aplicará la configuración completa
func (uc *ApplyOnboardingConfigUseCase) Execute(tenantID string) error {
    // 1. Obtener datos del onboarding
    process := uc.onboardingRepo.GetByTenantID(tenantID)
    
    // 2. Asignar plan recomendado al tenant (IAM Service)
    err := uc.iamClient.SetTenantPlan(tenantID, process.RecommendedPlan)
    
    // 3. Aplicar quickstart template (PIM Service)
    templateConfig := QuickstartConfig{
        BusinessType: process.BusinessType,
        StoreSize: process.StoreSize, // Determina límites y funcionalidades
    }
    err = uc.pimClient.ApplyQuickstartTemplate(tenantID, templateConfig)
    
    return nil
}
```

---

### 🔄 Implementación de los Flujos Mejorados

#### 1. 👤 Flujo de Registro de Usuario (Paso 2)
```go
// RegisterUserUseCase - Flujo completo con creación de tenant y usuario TENANT_ADMIN
func (uc *RegisterUserUseCase) Execute(req RegisterUserRequest) (*RegisterUserResponse, error) {
    // 1. 🏢 Crear tenant temporal
    tenantData := &CreateTenantRequest{
        Name:        fmt.Sprintf("Tienda de %s", req.Name),
        Slug:        generateSlugFromName(req.Name),
        Description: fmt.Sprintf("Tienda administrada por %s", req.Email),
        Type:        "STARTUP", // Default temporal, se actualiza en paso 4
        OwnerID:     "temp-uuid", // Se actualiza después
    }
    
    tenant, err := uc.iamClient.CreateTenant(tenantData)
    if err != nil {
        return nil, fmt.Errorf("error creating tenant: %w", err)
    }
    
    // 2. 👤 Obtener rol TENANT_ADMIN
    tenantAdminRole, err := uc.iamClient.GetRoleByType("TENANT_ADMIN")
    if err != nil {
        return nil, fmt.Errorf("error getting tenant admin role: %w", err)
    }
    
    // 3. 🧑‍💼 Crear usuario con rol de administrador
    userData := &CreateUserRequest{
        Email:    req.Email,
        Password: req.Password,
        TenantID: tenant.ID,
        RoleID:   tenantAdminRole.ID,
        Provider: "LOCAL",
    }
    
    user, err := uc.iamClient.CreateUser(userData)
    if err != nil {
        // Rollback: eliminar tenant si falla creación de usuario
        uc.iamClient.DeleteTenant(tenant.ID)
        return nil, fmt.Errorf("error creating user: %w", err)
    }
    
    // 4. 🔄 Actualizar owner del tenant
    err = uc.iamClient.UpdateTenantOwner(tenant.ID, user.ID)
    if err != nil {
        return nil, fmt.Errorf("error updating tenant owner: %w", err)
    }
    
    // 5. 📧 Enviar código de verificación
    err = uc.notificationClient.SendVerificationCode(user.Email)
    if err != nil {
        return nil, fmt.Errorf("error sending verification code: %w", err)
    }
    
    // 6. 💾 Actualizar proceso de onboarding
    process, err := uc.onboardingRepo.GetByID(req.ProcessID)
    if err != nil {
        return nil, err
    }
    
    process.TenantID = tenant.ID
    process.UserID = user.ID
    process.CurrentStepNumber = 3
    process.StepsCompleted = append(process.StepsCompleted, 1, 2)
    process.StepsPending = []int{3, 4, 5}
    
    err = uc.onboardingRepo.Save(process)
    if err != nil {
        return nil, err
    }
    
    return &RegisterUserResponse{
        ProcessID:         req.ProcessID,
        UserID:           user.ID,
        TenantID:         tenant.ID,
        TenantSlug:       tenant.Slug,
        RoleAssigned:     "TENANT_ADMIN",
        TenantType:       tenant.Type,
        CanAccessBackoffice: true,
        VerificationCodeSent: true,
        CurrentStep:      3,
        StepsCompleted:   []int{1, 2},
        StepsPending:     []int{3, 4, 5},
        NextStepURL:      "/onboarding/verificar-email",
    }, nil
}

// Función auxiliar para generar slug único
func generateSlugFromName(name string) string {
    // Convertir a lowercase y reemplazar espacios por guiones
    slug := strings.ToLower(strings.ReplaceAll(name, " ", "-"))
    // Agregar timestamp para unicidad
    timestamp := time.Now().Format("20060102")
    return fmt.Sprintf("%s-%s", slug, timestamp)
}
```

#### 2. 🏪 Flujo de Configuración de Tienda (Paso 4)
```go
// SetupStoreUseCase - Actualización completa del tenant con datos finales
func (uc *SetupStoreUseCase) Execute(req SetupStoreRequest) (*SetupStoreResponse, error) {
    // 1. 📋 Obtener proceso actual
    process, err := uc.onboardingRepo.GetByID(req.ProcessID)
    if err != nil {
        return nil, err
    }
    
    // 2. 🏢 Actualizar tenant con datos finales
    tenantUpdateData := &UpdateTenantRequest{
        Name:        req.StoreName,
        Slug:        generateSlugFromStoreName(req.StoreName),
        Description: fmt.Sprintf("Tienda de %s especializada en %s", req.StoreName, req.BusinessType),
        Type:        mapStoreTypeToTenantType(req.StoreSize),
    }
    
    tenant, err := uc.iamClient.UpdateTenant(process.TenantID, tenantUpdateData)
    if err != nil {
        return nil, fmt.Errorf("error updating tenant: %w", err)
    }
    
    // 3. 🎯 Aplicar configuración PIM con quickstart template
    pimSetupData := &PIMSetupRequest{
        BusinessType:       req.BusinessType,
        SelectedCategories: req.SelectedCategories,
        SelectedAttributes: []string{}, // Usar por defecto del business_type
        SelectedVariants:   []string{}, // Usar por defecto del business_type
        SelectedProducts:   []string{}, // Incluir productos ejemplo según req.IncludeSampleProducts
    }
    
    if req.IncludeSampleProducts {
        // Obtener productos ejemplo por defecto del business_type
        sampleProducts, err := uc.pimClient.GetSampleProductsByBusinessType(req.BusinessType)
        if err == nil {
            pimSetupData.SelectedProducts = sampleProducts
        }
    }
    
    pimHistory, err := uc.pimClient.SetupTenant(process.TenantID, pimSetupData)
    if err != nil {
        return nil, fmt.Errorf("error setting up PIM configuration: %w", err)
    }
    
    // 4. 📊 Determinar plan recomendado
    recommendedPlan := uc.determinePlanBySize(req.StoreSize)
    
    // 5. 💾 Actualizar proceso con datos de tienda y configuración PIM
    process.CompanyName = req.StoreName
    process.BusinessType = req.BusinessType
    process.StoreSize = req.StoreSize
    process.PIMConfigured = true
    process.CatalogSetupCompleted = true
    process.CurrentStepNumber = 5
    process.StepsCompleted = append(process.StepsCompleted, 4)
    process.StepsPending = []int{5}
    
    err = uc.onboardingRepo.Save(process)
    if err != nil {
        return nil, err
    }
    
    return &SetupStoreResponse{
        ProcessID:              req.ProcessID,
        StoreConfigured:        true,
        BusinessTypeApplied:    req.BusinessType,
        StoreSizeApplied:       req.StoreSize,
        PlanRecommended:        recommendedPlan,
        TenantUpdated:          true,
        TenantName:             tenant.Name,
        TenantSlug:             tenant.Slug,
        TenantType:             tenant.Type,
        CatalogConfigured:      true,
        CategoriesCreated:      req.SelectedCategories,
        AttributesCreated:      len(pimHistory.AttributesCreated),
        SampleProductsCreated:  len(pimHistory.ProductsCreated),
        PIMSetupCompleted:      true,
        CurrentStep:            5,
        StepsCompleted:         process.StepsCompleted,
        StepsPending:           []int{5},
        NextStepURL:            "/onboarding/completar",
    }, nil
}

// Mapeo de store_size a tenant_type
func mapStoreTypeToTenantType(storeSize string) string {
    switch storeSize {
    case "micro":
        return "STARTUP"
    case "pyme":
        return "BUSINESS"
    case "multiple":
        return "ENTERPRISE"
    default:
        return "STARTUP"
    }
}

// Mapeo de store_size a plan recomendado
func (uc *SetupStoreUseCase) determinePlanBySize(storeSize string) string {
    switch storeSize {
    case "micro":
        return "basic"
    case "pyme":
        return "professional"
    case "multiple":
        return "enterprise"
    default:
        return "basic"
    }
}
```

#### 3. 🎉 Flujo de Completación (Paso 5)
```go
// CompleteOnboardingUseCase - Validación final y preparación para backoffice
func (uc *CompleteOnboardingUseCase) Execute(req CompleteOnboardingRequest) (*CompleteOnboardingResponse, error) {
    // 1. 📋 Obtener y validar proceso
    process, err := uc.onboardingRepo.GetByID(req.ProcessID)
    if err != nil {
        return nil, err
    }
    
    if len(process.StepsCompleted) < 4 {
        return nil, fmt.Errorf("onboarding incompleto: solo %d pasos completados", len(process.StepsCompleted))
    }
    
    // 2. ✅ Validar que el usuario tenga permisos correctos
    user, err := uc.iamClient.GetUserByID(process.UserID)
    if err != nil {
        return nil, fmt.Errorf("error validating user: %w", err)
    }
    
    // Verificar que tiene rol TENANT_ADMIN
    role, err := uc.iamClient.GetRoleByID(user.RoleID)
    if err != nil || role.Type != "TENANT_ADMIN" {
        return nil, fmt.Errorf("user does not have TENANT_ADMIN role")
    }
    
    // 3. 🔐 Generar token de acceso para backoffice
    loginData := &LoginRequest{
        Email:    user.Email,
        TenantID: process.TenantID,
    }
    
    authResponse, err := uc.iamClient.GenerateAccessToken(loginData)
    if err != nil {
        return nil, fmt.Errorf("error generating access token: %w", err)
    }
    
    // 4. 🏢 Obtener resumen del tenant
    tenant, err := uc.iamClient.GetTenantByID(process.TenantID)
    if err != nil {
        return nil, fmt.Errorf("error getting tenant summary: %w", err)
    }
    
    // 5. 💾 Marcar proceso como completado
    process.IsCompleted = true
    process.CompletedAt = time.Now()
    process.CurrentStepNumber = 5
    process.StepsCompleted = []int{1, 2, 3, 4, 5}
    process.StepsPending = []int{}
    
    err = uc.onboardingRepo.Save(process)
    if err != nil {
        return nil, err
    }
    
    // 6. 📧 Notificar configuración lista (opcional)
    uc.notificationClient.SendOnboardingCompleted(user.Email, tenant.Name)
    
    return &CompleteOnboardingResponse{
        ProcessID:               req.ProcessID,
        OnboardingCompleted:     true,
        UserReadyForBackoffice:  true,
        TenantAdminConfirmed:    true,
        AccessToken:            authResponse.AccessToken,
        BackofficeURL:          fmt.Sprintf("/backoffice/dashboard?first_time=true&process_id=%s", req.ProcessID),
        RedirectURL:            fmt.Sprintf("https://app.example.com/backoffice/dashboard?first_time=true&process_id=%s", req.ProcessID),
        TenantSummary: TenantSummary{
            TenantID:         tenant.ID,
            TenantName:       tenant.Name,
            TenantSlug:       tenant.Slug,
            BusinessType:     process.BusinessType,
            StoreSize:        process.StoreSize,
            PlanRecommended:  uc.determinePlanBySize(process.StoreSize),
            OwnerConfirmed:   true,
        },
        StepsCompleted:   []int{1, 2, 3, 4, 5},
        StepsPending:     []int{},
        TotalTimeMinutes: float64(time.Since(process.StartedAt).Seconds()) / 60.0,
    }, nil
}
```

#### 4. 📊 Iniciar un nuevo proceso de onboarding:
```go
// Al crear un nuevo proceso
process := &OnboardingProcess{
    TenantID: "", // Se asigna en paso 2
    UserID: "", // Se asigna en paso 2
    CurrentStepNumber: 1,
    StepsCompleted: []int{}, // vacío
    StepsPending: []int{1, 2, 3, 4, 5}, // todos pendientes
    StepsSkipped: []int{}, // vacío
}

// También crear registros de progreso para cada paso
for _, stepDef := range stepDefinitions {
    stepProgress := &OnboardingStepProgress{
        ProcessID: process.ID,
        StepDefinitionID: stepDef.ID,
        Status: "pending",
    }
}
```

#### 2. Completar un paso:
```go
// Cuando el usuario completa el paso 2 (registro)
func CompleteStep(processID string, stepNumber int, stepData map[string]interface{}) error {
    // 1. Actualizar el progreso individual del paso
    stepProgress.Status = "completed"
    stepProgress.CompletedAt = time.Now()
    stepProgress.StepData = stepData
    
    // 2. Actualizar el proceso general
    process.StepsCompleted = append(process.StepsCompleted, stepNumber)
    process.StepsPending = removeFromSlice(process.StepsPending, stepNumber)
    process.CurrentStepNumber = getNextPendingStep(process.StepsPending)
    
    return nil
}
```

#### 3. Obtener estado actual:
```go
type OnboardingStatus struct {
    ProcessID       string                    `json:"process_id"`
    CurrentStep     int                      `json:"current_step"`
    StepsCompleted  []int                    `json:"steps_completed"`
    StepsPending    []int                    `json:"steps_pending"`
    StepsSkipped    []int                    `json:"steps_skipped"`
    ProgressPercent float64                  `json:"progress_percent"`
    StepDetails     []OnboardingStepDetail   `json:"step_details"`
}

type OnboardingStepDetail struct {
    StepNumber      int       `json:"step_number"`
    StepName        string    `json:"step_name"`
    StepTitle       string    `json:"step_title"`
    Status          string    `json:"status"`
    IsRequired      bool      `json:"is_required"`
    CanBeSkipped    bool      `json:"can_be_skipped"`
    CompletedAt     *time.Time `json:"completed_at,omitempty"`
    TimeSpent       *int      `json:"time_spent_seconds,omitempty"`
}
```

#### 4. Ejemplo de respuesta del endpoint `/status`:
```json
{
  "process_id": "uuid-process-123",
  "current_step": 3,
  "steps_completed": [1, 2],
  "steps_pending": [3, 4, 5],
  "steps_skipped": [],
  "progress_percent": 40.0,
  "step_details": [
    {
      "step_number": 1,
      "step_name": "bienvenida",
      "step_title": "Bienvenida",
      "status": "completed",
      "is_required": true,
      "can_be_skipped": false,
      "completed_at": "2024-01-15T10:30:00Z",
      "time_spent_seconds": 45
    },
    {
      "step_number": 2,
      "step_name": "registro",
      "step_title": "Datos de Usuario",
      "status": "completed",
      "is_required": true,
      "can_be_skipped": false,
      "completed_at": "2024-01-15T10:32:30Z",
      "time_spent_seconds": 120
    },
    {
      "step_number": 3,
      "step_name": "verificacion",
      "step_title": "Verificar Email",
      "status": "in_progress",
      "is_required": true,
      "can_be_skipped": false
    },
    {
      "step_number": 4,
      "step_name": "configuracion_tienda",
      "step_title": "Detalles de la Tienda",
      "status": "pending",
      "is_required": true,
      "can_be_skipped": false
    },
    {
      "step_number": 5,
      "step_name": "completar",
      "step_title": "Finalizar",
      "status": "pending",
      "is_required": true,
      "can_be_skipped": false
    }
  ]
}
```

---

## ✅ Resumen del Servicio

El **Onboarding Service** maneja únicamente los **5 pasos iniciales** del proceso de registro:

1. **🏠 Bienvenida** - Landing page estática
2. **👤 Registro** - Crear tenant/usuario (IAM + Notification)
3. **✉️ Verificación** - Validar email (IAM)
4. **🏪 Configuración básica** - Tipo de negocio y tamaño (PIM)
5. **🎉 Completar** - Redirección al backoffice

### 🎯 Ventajas de la Nueva Estructura:
- ✅ **Steps configurables** via base de datos
- ✅ **Control granular** de estado por paso
- ✅ **Flexibilidad** para agregar/quitar pasos sin código
- ✅ **Tracking detallado** de tiempo y errores
- ✅ **Estados claros**: completed, pending, skipped, failed
- ✅ **Datos por paso** almacenados en JSONB
- ✅ **Business types desde PIM** - No duplicamos datos

### 🔗 Dependencias de Servicios

| **Servicio** | **Puerto** | **Responsabilidad** | **Endpoints Usados** |
|-------------|------------|-------------------|---------------------|
| **IAM Service** | 8080 | Tenants, usuarios, roles, verificación | `POST /tenants`, `POST /users`, `GET /roles`, `PUT /tenants/{id}` |
| **PIM Service** | 8090 | Business types, categorías y configuración completa | `GET /quickstart/business-types`, `GET /quickstart/categories/{businessType}`, `POST /quickstart/setup` |
| **Notification Service** | TBD | Emails de verificación | `POST /notifications/send-email` |
| **Backoffice** | 3000 | Redirección post-onboarding | N/A (solo redirect) |

---

## 🚀 Endpoints del Onboarding Service

### **Base URL**: `http://localhost:8110/api/v1/onboarding`

#### 📋 **1. Obtener configuración de steps**
```http
GET /api/v1/onboarding/steps
```
**Descripción**: Obtiene la lista de pasos configurados en el onboarding  
**Response**:
```json
{
  "steps": [
    {
      "step_number": 1,
      "step_name": "bienvenida", 
      "step_title": "Bienvenida",
      "description": "Pantalla de bienvenida al onboarding",
      "is_required": true,
      "has_ui": true,
      "requires_user_input": false,
      "can_be_skipped": false
    },
    {
      "step_number": 2,
      "step_name": "registro",
      "step_title": "Datos de Usuario", 
      "description": "Captura nombre, email y contraseña",
      "is_required": true,
      "has_ui": true,
      "requires_user_input": true,
      "can_be_skipped": false
    }
  ]
}
```

#### 🚀 **2. Iniciar proceso de onboarding**
```http
POST /api/v1/onboarding/start
```
**Request**:
```json
{
  "source": "landing_page",
  "utm_campaign": "google_ads_q1_2024",
  "referrer": "https://google.com"
}
```
**Response**:
```json
{
  "process_id": "uuid-process-123",
  "current_step": 1,
  "steps_completed": [],
  "steps_pending": [1, 2, 3, 4, 5],
  "next_step_url": "/onboarding/bienvenida"
}
```

#### 👤 **3. Registro de usuario (Paso 2)**
```http
POST /api/v1/onboarding/register-user
```
**Request**:
```json
{
  "process_id": "uuid-process-123",
  "name": "Juan Pérez",
  "email": "juan@example.com",
  "password": "mi-password-123",
  "password_confirmation": "mi-password-123"
}
```

**⚙️ Flujo interno completo:**
1. **🏢 Crear Tenant automáticamente** con datos temporales
2. **👤 Obtener rol TENANT_ADMIN** desde IAM Service
3. **🧑‍💼 Crear Usuario** con rol de administrador
4. **🔄 Actualizar owner** del tenant con el usuario creado
5. **📧 Enviar código** de verificación

**Response**:
```json
{
  "process_id": "uuid-process-123",
  "user_id": "uuid-user-456",
  "tenant_id": "uuid-tenant-789",
  "tenant_slug": "juan-perez-store",
  "role_assigned": "TENANT_ADMIN",
  "tenant_type": "STARTUP",
  "verification_code_sent": true,
  "can_access_backoffice": true,
  "current_step": 3,
  "steps_completed": [1, 2],
  "steps_pending": [3, 4, 5],
  "next_step_url": "/onboarding/verificar-email"
}
```

#### ✉️ **4. Verificar email (Paso 3)**
```http
POST /api/v1/onboarding/verify-email
```
**Request**:
```json
{
  "process_id": "uuid-process-123",
  "verification_code": "123456"
}
```
**Response**:
```json
{
  "process_id": "uuid-process-123",
  "verified": true,
  "current_step": 4,
  "steps_completed": [1, 2, 3],
  "steps_pending": [4, 5],
  "next_step_url": "/onboarding/configurar-tienda"
}
```

#### 🏪 **5. Configurar tienda (Paso 4)**
```http
POST /api/v1/onboarding/setup-store
```
**Request**:
```json
{
  "process_id": "uuid-process-123",
  "store_name": "Pinturería San Martín",
  "business_type": "home-construction",
  "store_size": "micro",
  "selected_categories": ["paint", "brushes", "tools"],
  "include_sample_products": true,
  "include_default_attributes": true
}
```

**⚙️ Flujo interno:**
1. **🏢 Actualizar Tenant** con datos finales de la tienda
2. **📊 Determinar plan** recomendado según store_size
3. **🎯 Aplicar configuración PIM** con categorías y atributos seleccionados
4. **📦 Configurar catálogo inicial** con productos ejemplo (opcional)
5. **✅ Validar** que el usuario tenga permisos correctos

**Response**:
```json
{
  "process_id": "uuid-process-123",
  "store_configured": true,
  "business_type_applied": "home-construction",
  "store_size_applied": "micro",
  "plan_recommended": "basic",
  "tenant_updated": true,
  "tenant_name": "Pinturería San Martín",
  "tenant_slug": "pintureria-san-martin",
  "tenant_type": "STARTUP",
  "catalog_configured": true,
  "categories_created": ["paint", "brushes", "tools"],
  "attributes_created": 15,
  "sample_products_created": 8,
  "pim_setup_completed": true,
  "current_step": 5,
  "steps_completed": [1, 2, 3, 4],
  "steps_pending": [5],
  "next_step_url": "/onboarding/completar"
}
```

**Mapeo completo de `store_size`**:

| store_size | Tenant Type (IAM) | Plan Recomendado | Configuración PIM | Límites |
|------------|------------------|------------------|-------------------|---------|
| `micro` | `STARTUP` | `basic` | Template básico | 100 productos, 5 categorías |
| `pyme` | `BUSINESS` | `professional` | Template estándar | 1000 productos, 20 categorías |
| `multiple` | `ENTERPRISE` | `enterprise` | Template completo | Ilimitado, funciones avanzadas |

**Integración con otros servicios**:
- **IAM Service**: Actualiza tenant final + asigna plan recomendado
- **PIM Service**: Aplica configuración completa de quickstart template
- **Notification Service**: Notifica configuración completada

**Datos enviados al PIM Service**:
```json
{
  "businessType": "home-construction",
  "selectedCategories": ["paint", "brushes", "tools"],
  "selectedAttributes": [], // Se incluyen atributos por defecto del business_type
  "selectedVariants": [],   // Se incluyen variantes por defecto del business_type
  "selectedProducts": []    // Se incluyen productos ejemplo si include_sample_products=true
}
```

#### 🎉 **6. Completar onboarding (Paso 5)**
```http
POST /api/v1/onboarding/complete
```
**Request**:
```json
{
  "process_id": "uuid-process-123"
}
```

**⚙️ Flujo interno final:**
1. **✅ Validar completación** de todos los pasos críticos
2. **🔐 Generar tokens** de acceso para backoffice
3. **🎯 Preparar datos** de configuración para backoffice
4. **🏢 Confirmar permisos** TENANT_ADMIN activos
5. **📧 Notificar** configuración lista

**Response**:
```json
{
  "process_id": "uuid-process-123",
  "onboarding_completed": true,
  "completion_time_seconds": 245,
  "user_ready_for_backoffice": true,
  "tenant_admin_confirmed": true,
  "access_token": "jwt-token-for-backoffice",
  "backoffice_url": "/backoffice/dashboard?first_time=true&process_id=uuid-process-123",
  "redirect_url": "https://app.example.com/backoffice/dashboard?first_time=true&process_id=uuid-process-123",
  "tenant_summary": {
    "tenant_id": "uuid-tenant-789",
    "tenant_name": "Pinturería San Martín",
    "tenant_slug": "pintureria-san-martin",
    "business_type": "pintureria",
    "store_size": "micro",
    "plan_recommended": "basic",
    "owner_confirmed": true
  },
  "steps_completed": [1, 2, 3, 4, 5],
  "steps_pending": [],
  "total_time_minutes": 4.08
}
```

#### 📊 **7. Obtener estado del proceso**
```http
GET /api/v1/onboarding/status/{process_id}
```
**Response**:
```json
{
  "process_id": "uuid-process-123",
  "tenant_id": "uuid-tenant-789", 
  "user_id": "uuid-user-456",
  "current_step": 3,
  "steps_completed": [1, 2],
  "steps_pending": [3, 4, 5],
  "steps_skipped": [],
  "progress_percent": 40.0,
  "is_completed": false,
  "started_at": "2024-01-15T10:30:00Z",
  "estimated_remaining_time_minutes": 3,
  "step_details": [
    {
      "step_number": 1,
      "step_name": "bienvenida",
      "step_title": "Bienvenida",
      "status": "completed",
      "completed_at": "2024-01-15T10:30:30Z",
      "time_spent_seconds": 30
    },
    {
      "step_number": 2,
      "step_name": "registro", 
      "step_title": "Datos de Usuario",
      "status": "completed",
      "completed_at": "2024-01-15T10:32:45Z",
      "time_spent_seconds": 135
    },
    {
      "step_number": 3,
      "step_name": "verificacion",
      "step_title": "Verificar Email",
      "status": "in_progress",
      "started_at": "2024-01-15T10:32:45Z"
    }
  ]
}
```

#### 🔄 **8. Reenviar código de verificación**
```http
POST /api/v1/onboarding/resend-verification
```
**Request**:
```json
{
  "process_id": "uuid-process-123"
}
```
**Response**:
```json
{
  "verification_code_sent": true,
  "can_resend_again_in_seconds": 60
}
```

#### 🏷️ **9. Obtener business types disponibles**
```http
GET /api/v1/onboarding/business-types
```
**Descripción**: Proxy al PIM Service para obtener tipos de negocio actualizados
**Response**:
```json
{
  "business_types": [
    {
      "id": "pintureria",
      "name": "Pinturería", 
      "description": "Venta de pinturas y accesorios",
      "icon": "🎨"
    },
    {
      "id": "ferreteria",
      "name": "Ferretería",
      "description": "Herramientas y materiales de construcción", 
      "icon": "🔧"
    }
  ]
}
```

#### ❌ **10. Abandonar proceso**
```http
POST /api/v1/onboarding/abandon
```
**Request**:
```json
{
  "process_id": "uuid-process-123",
  "reason": "too_complicated",
  "feedback": "Demasiados pasos, prefiero hacerlo después"
}
```
**Response**:
```json
{
  "process_abandoned": true,
  "can_resume": true,
  "resume_url": "/onboarding/resume/uuid-process-123"
}
```

#### 🔄 **11. Reanudar proceso abandonado**
```http
POST /api/v1/onboarding/resume/{process_id}
```
**Response**:
```json
{
  "process_resumed": true,
  "current_step": 3,
  "next_step_url": "/onboarding/verificar-email"
}
```

### 🔍 **Códigos de Error Comunes**

| **Código** | **Mensaje** | **Descripción** |
|------------|-------------|-----------------|
| `400` | `INVALID_PROCESS_ID` | Process ID inválido o no encontrado |
| `400` | `STEP_ALREADY_COMPLETED` | Intento de completar un paso ya terminado |
| `400` | `INVALID_VERIFICATION_CODE` | Código de verificación incorrecto |
| `400` | `EMAIL_ALREADY_EXISTS` | Email ya registrado en el sistema |
| `429` | `VERIFICATION_CODE_RATE_LIMIT` | Demasiados intentos de reenvío |
| `500` | `IAM_SERVICE_ERROR` | Error en comunicación con IAM Service |
| `500` | `PIM_SERVICE_ERROR` | Error en comunicación con PIM Service |
| `500` | `TENANT_CREATION_FAILED` | Error al crear tenant en IAM Service |
| `500` | `USER_CREATION_FAILED` | Error al crear usuario con rol TENANT_ADMIN |
| `400` | `INVALID_ROLE_ASSIGNMENT` | Error al asignar rol TENANT_ADMIN |
| `500` | `TENANT_UPDATE_FAILED` | Error al actualizar datos del tenant |

## 🔗 **Integración Mejorada con PIM Service**

### **📋 Nuevos Endpoints Requeridos en Onboarding:**

#### **1. Obtener Business Types dinámicamente**
```http
GET /api/v1/onboarding/business-types
```
**Descripción**: Proxy al PIM Service para obtener tipos de negocio actualizados
**Integración**: `GET /api/v1/quickstart/business-types` → PIM Service

#### **2. Obtener Categorías por Business Type**
```http
GET /api/v1/onboarding/categories?business_type=home-construction
```
**Descripción**: Proxy al PIM Service para obtener categorías disponibles
**Integración**: `GET /api/v1/quickstart/categories/{businessType}` → PIM Service

#### **3. Setup Store actualizado con configuración PIM**
```http
POST /api/v1/onboarding/setup-store
```
**Nuevo Request**:
```json
{
  "process_id": "uuid-process-123",
  "store_name": "Pinturería San Martín",
  "business_type": "home-construction",
  "store_size": "micro",
  "selected_categories": ["paint", "brushes", "tools"],
  "include_sample_products": true,
  "include_default_attributes": true
}
```

### **🎯 Flujo de Integración PIM Completo:**

```go
// SetupStoreUseCase - Integración completa con PIM Service
func (uc *SetupStoreUseCase) Execute(req SetupStoreRequest) (*SetupStoreResponse, error) {
    // 1. Validar business_type con PIM Service
    businessTypes, err := uc.pimClient.GetBusinessTypes()
    if err != nil {
        return nil, fmt.Errorf("error validating business type: %w", err)
    }
    
    if !isValidBusinessType(req.BusinessType, businessTypes) {
        return nil, fmt.Errorf("invalid business type: %s", req.BusinessType)
    }
    
    // 2. Validar categorías seleccionadas
    availableCategories, err := uc.pimClient.GetCategoriesByBusinessType(req.BusinessType)
    if err != nil {
        return nil, fmt.Errorf("error getting categories: %w", err)
    }
    
    if !areValidCategories(req.SelectedCategories, availableCategories) {
        return nil, fmt.Errorf("invalid categories selected")
    }
    
    // 3. Actualizar tenant en IAM Service
    tenant, err := uc.updateTenant(process.TenantID, req)
    if err != nil {
        return nil, fmt.Errorf("error updating tenant: %w", err)
    }
    
    // 4. Aplicar configuración completa en PIM Service
    pimSetupData := &PIMSetupRequest{
        BusinessType:       req.BusinessType,
        SelectedCategories: req.SelectedCategories,
        SelectedAttributes: []string{}, // Usar defaults del business_type
        SelectedVariants:   []string{}, // Usar defaults del business_type
        SelectedProducts:   []string{}, // Opcional según req.IncludeSampleProducts
    }
    
    // 4.1. Incluir productos ejemplo si se solicita
    if req.IncludeSampleProducts {
        sampleProducts, err := uc.pimClient.GetSampleProductsByBusinessType(req.BusinessType)
        if err == nil && len(sampleProducts) > 0 {
            // Limitar a máximo 5 productos ejemplo para onboarding rápido
            if len(sampleProducts) > 5 {
                pimSetupData.SelectedProducts = sampleProducts[:5]
            } else {
                pimSetupData.SelectedProducts = sampleProducts
            }
        }
    }
    
    // 4.2. Aplicar configuración PIM
    pimHistory, err := uc.pimClient.SetupTenant(process.TenantID, pimSetupData)
    if err != nil {
        return nil, fmt.Errorf("error configuring PIM: %w", err)
    }
    
    // 5. Actualizar proceso con configuración completada
    process.PIMConfigured = true
    process.CatalogSetupCompleted = true
    process.CategoriesConfigured = len(req.SelectedCategories)
    process.AttributesConfigured = len(pimHistory.AttributesCreated)
    process.ProductsConfigured = len(pimHistory.ProductsCreated)
    
    return &SetupStoreResponse{
        // ... campos existentes ...
        CatalogConfigured:      true,
        CategoriesCreated:      req.SelectedCategories,
        AttributesCreated:      len(pimHistory.AttributesCreated),
        SampleProductsCreated:  len(pimHistory.ProductsCreated),
        PIMSetupCompleted:      true,
    }, nil
}
```

### **📊 Mapeo de Business Types:**

```go
// Mapeo entre UI amigable y business_type del PIM
var BusinessTypeMapping = map[string]string{
    "pintureria":      "home-construction",
    "ferreteria":      "home-construction",
    "ropa":            "fashion",
    "electronica":     "electronics",
    "automotriz":      "automotive",
    "libros":          "books-media",
    "alimentos":       "food-beverage",
    "farmacia":        "health-pharmacy",
    "deportes":        "sports-fitness",
    "belleza":         "beauty-cosmetics",
    "juguetes":        "toys-games",
    "mascotas":        "pet-supplies",
    "oficina":         "office-supplies",
    "joyeria":         "jewelry-accessories",
    "polirubro":       "polirubro",
}

func mapToBusinessType(userFriendlyType string) string {
    if pimType, exists := BusinessTypeMapping[userFriendlyType]; exists {
        return pimType
    }
    return userFriendlyType // fallback
}
```

### **🔄 Endpoints PIM Service utilizados:**

| Endpoint PIM | Propósito | Cuándo se usa |
|-------------|-----------|---------------|
| `GET /quickstart/business-types` | Obtener lista de tipos de negocio | Al cargar paso 4 (dropdown) |
| `GET /quickstart/categories/{businessType}` | Obtener categorías disponibles | Al seleccionar business_type |
| `POST /quickstart/setup` | Aplicar configuración completa | Al confirmar setup en paso 4 |

### **✅ Validaciones agregadas:**

1. **Business Type válido**: Verificar contra lista del PIM Service
2. **Categorías válidas**: Verificar que las categorías seleccionadas existen para el business_type
3. **Mínimo una categoría**: El usuario debe seleccionar al menos 1 categoría
4. **Productos ejemplo limitados**: Máximo 5 productos ejemplo para onboarding rápido

### **🎯 Resultado Final Garantizado:**

Después del paso 4, el tenant tendrá:
- ✅ **Configuración IAM**: Tenant actualizado con datos finales
- ✅ **Configuración PIM**: Catálogo básico con categorías y atributos
- ✅ **Productos ejemplo**: Hasta 5 productos ejemplo (opcional)
- ✅ **Estructura base**: Lista para comenzar a usar en backoffice

## 🔒 **Validaciones y Requisitos para Backoffice**

### **✅ Requisitos obligatorios para acceso al backoffice:**

#### **1. Usuario TENANT_ADMIN activo**
```go
// Validaciones que debe pasar el usuario:
type BackofficeAccessValidation struct {
    UserID     string `validate:"required,uuid"`
    TenantID   string `validate:"required,uuid"`
    RoleType   string `validate:"required,eq=TENANT_ADMIN"`
    UserStatus string `validate:"required,eq=ACTIVE"`
    EmailVerified bool `validate:"required,eq=true"`
}
```

#### **2. Tenant configurado correctamente**
```go
// Validaciones del tenant:
type TenantValidation struct {
    TenantID    string `validate:"required,uuid"`
    TenantName  string `validate:"required,min=2,max=100"`
    TenantSlug  string `validate:"required,min=2,max=50"`
    TenantType  string `validate:"required,oneof=STARTUP BUSINESS ENTERPRISE"`
    OwnerID     string `validate:"required,uuid"`
    TenantStatus string `validate:"required,eq=ACTIVE"`
}
```

#### **3. Proceso de onboarding completado**
```go
// Validaciones del proceso:
type OnboardingValidation struct {
    ProcessID      string `validate:"required,uuid"`
    IsCompleted    bool   `validate:"required,eq=true"`
    StepsCompleted []int  `validate:"required,len=5"`
    EmailVerified  bool   `validate:"required,eq=true"`
}
```

### **🔐 Permisos mínimos requeridos:**

El usuario debe tener rol **TENANT_ADMIN** con los siguientes permisos:
- `tenant:*` - Gestión completa del tenant
- `user:*` - Gestión de usuarios del tenant  
- `role:*` - Gestión de roles del tenant

### **🚨 Validaciones de seguridad:**

#### **1. Verificación de ownership**
```go
func (uc *ValidateBackofficeAccessUseCase) ValidateOwnership(userID, tenantID string) error {
    tenant, err := uc.iamClient.GetTenantByID(tenantID)
    if err != nil {
        return fmt.Errorf("tenant not found: %w", err)
    }
    
    if tenant.OwnerID != userID {
        return fmt.Errorf("user is not the owner of the tenant")
    }
    
    return nil
}
```

#### **2. Verificación de rol activo**
```go
func (uc *ValidateBackofficeAccessUseCase) ValidateRole(userID string) error {
    user, err := uc.iamClient.GetUserByID(userID)
    if err != nil {
        return fmt.Errorf("user not found: %w", err)
    }
    
    role, err := uc.iamClient.GetRoleByID(user.RoleID)
    if err != nil {
        return fmt.Errorf("role not found: %w", err)
    }
    
    if role.Type != "TENANT_ADMIN" {
        return fmt.Errorf("user does not have TENANT_ADMIN role")
    }
    
    if !role.IsActive {
        return fmt.Errorf("role is not active")
    }
    
    return nil
}
```

### **📊 Flujo de validación completo antes del backoffice:**

```go
func (uc *CompleteOnboardingUseCase) ValidateBackofficeReadiness(processID string) (*BackofficeReadinessResponse, error) {
    // 1. Validar proceso completo
    process, err := uc.onboardingRepo.GetByID(processID)
    if err != nil {
        return nil, fmt.Errorf("process not found: %w", err)
    }
    
    if !process.IsCompleted || len(process.StepsCompleted) < 5 {
        return nil, fmt.Errorf("onboarding process not completed")
    }
    
    // 2. Validar usuario y rol
    user, err := uc.iamClient.GetUserByID(process.UserID)
    if err != nil {
        return nil, fmt.Errorf("user validation failed: %w", err)
    }
    
    // Verificar que tiene rol TENANT_ADMIN
    role, err := uc.iamClient.GetRoleByID(user.RoleID)
    if err != nil || role.Type != "TENANT_ADMIN" {
        return nil, fmt.Errorf("user does not have required TENANT_ADMIN role")
    }
    
    // 3. Validar tenant
    tenant, err := uc.iamClient.GetTenantByID(process.TenantID)
    if err != nil {
        return nil, fmt.Errorf("tenant validation failed: %w", err)
    }
    
    if tenant.Status != "ACTIVE" || tenant.OwnerID != user.ID {
        return nil, fmt.Errorf("tenant validation failed: inactive or ownership mismatch")
    }
    
    // 4. Validar email verificado
    if !user.EmailVerified {
        return nil, fmt.Errorf("email not verified")
    }
    
    return &BackofficeReadinessResponse{
        Ready: true,
        UserID: user.ID,
        TenantID: tenant.ID,
        RoleType: role.Type,
        Permissions: role.Permissions,
        CanAccessBackoffice: true,
        ValidationsPassed: []string{
            "onboarding_completed",
            "user_active",
            "role_tenant_admin",
            "tenant_active",
            "ownership_confirmed",
            "email_verified",
        },
    }, nil
}
```

**Puerto**: 8110 | **Base de datos**: onboarding_db | **Arquitectura**: Hexagonal 