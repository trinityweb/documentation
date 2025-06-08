# 🎯 Objetivos
- [ ] 💰 Punto de venta (POS) 
- [ ] 🏪 Marketplace
  
## 🚀 MVP - Estado Actual

### ✅ Completado
- ✅ 🔐 **Backend: IAM**
  - ✅ 🔑 Login Local
    - ✅ 🎟️ Generar Token
    - ✅ 🔄 Refrescar Token
  - ✅ 🌐 Login Federado
    - ✅ 🔍 Google OAuth
  - ✅ 🏢 CRUD Tenant completo
  - ✅ ⚙️ Tenant Config con features
  - ✅ 💳 CRUD Plans completo
  - ✅ 👥 CRUD Users completo
  - ✅ 🛡️ CRUD Roles completo
- ✅ 📦 **Backend: PIM**
  - ✅ 📑 CRUD Categorías con árbol jerárquico
  - ✅ 🔍 Patrón Criteria implementado
- ✅ 🏗️ **Infraestructura Base**
  - ✅ 🐳 Docker Compose configurado
  - ✅ 🌐 Kong API Gateway
  - ✅ 📊 Stack de Monitoreo (Prometheus + Grafana + Loki)
  - ✅ 🗄️ PostgreSQL multi-tenant

### 🔄 En Desarrollo
- 🔄 🎨 **Frontend: Landing + Onboarding**
  - 🔄 Landing page básica
  - [ ] Flujo de onboarding completo
- 🔄 👨‍💼 **Frontend: Seller Backoffice**
  - ✅ Administración básica funcionando
  - 🔄 Integración completa con todos los servicios
- 🔄 💬 **Backend: Chat Service**
  - 🔄 Migración a patrón Criteria
  - [ ] WebSockets para tiempo real

### 📋 Pendientes Inmediatos
- [ ] 🏷️ **Backend: PIM - Productos**
  - [ ] CRUD Productos y Variantes
  - [ ] 🖼️ Storage Images
  - [ ] ⚙️ Catalog Config
- [ ] 📊 **Backend: Stock**
  - [ ] 📍 Location Management
    - [ ] 🏪 Type: Store o DistribucionCenter
    - [ ] 🏭 Warehouse
    - [ ] 📦 stockLocation
- [ ] 🎯 **Backend: Onboarding Service**
- [ ] 🛒 **Backend: Orders Service**
- [ ] 💰 **Backend: Biller with Integrations**
- [ ] 🔔 **Backend: Notification Service**

### 📋 Infraestructura Pendiente
- [ ] 📁 **Repos Individuales**: Separar microservicios
- [ ] 🌍 **Terraform**: DigitalOcean deployment
  - [ ] 🖥️ K3s cluster
  - [ ] 🗄️ PostgreSQL dedicado
  - [ ] 🌐 Load balancer
- [ ] 🔄 **CI/CD**: GitHub Actions
- [ ] 📨 **Mensajería**: AWS SNS/SQS o alternativa
- [ ] 💾 **Storage**: AWS S3 o DigitalOcean Spaces

## 🏪 MarketPlace - Planificado

### 📋 Frontend Ecommerce
- [ ] 🛍️ **Frontend: Ecommerce**
  - [ ] 🏠 Home Page Geolocalizada
  - [ ] 📑 Category Page
  - [ ] 🔍 Search Page
  - [ ] 🏷️ Product Page
  - [ ] 🛒 Carrito de Compras (cart)
  - [ ] 👤 User Login
  - [ ] 💳 CheckOut
  - [ ] 👤 User Page
    - [ ] 📦 Mis ordenes
    - [ ] ⭐ Favoritos
  - [ ] 🏪 Seller/Store Page

### 📋 Backend Marketplace
- [ ] 💰 **Backend: Payments Service**
- [ ] 📦 **Backend: Shipments Service**
  
## 📈 Evolución del Producto

### 📋 Contenido y Marketing
- [ ] 📝 **Backend: CMS**
  - [ ] 🎠 Sliders
  - [ ] 🎡 Carruceles
  - [ ] 📢 Publicidades

### 📋 Expansión de Servicios
- [ ] 📰 **Avisos Clasificados** (Inmuebles/Vehículos)
- [ ] 🕒 **Servicios con Turnos**
- [ ] 💼 **Ofertas Laborales**

## 🎯 Prioridades Inmediatas (Próximas 2 semanas)

### Alta Prioridad
1. [ ] **Completar Chat Service**: Migrar a patrón Criteria
2. [ ] **PIM - Productos**: Implementar CRUD de productos
3. [ ] **Tests**: Implementar tests unitarios básicos
4. [ ] **Deploy**: Configurar infraestructura en DigitalOcean

### Media Prioridad
1. [ ] **Notification Service**: Diseño e implementación
2. [ ] **Stock Service**: Planificación y diseño
3. [ ] **Onboarding**: Flujo completo de registro

### Baja Prioridad
1. [ ] **Orders Service**: Planificación
2. [ ] **Billing Service**: Investigación de integraciones
3. [ ] **Marketplace Frontend**: Prototipo inicial

---

**Última actualización**: Enero 2025  
**Estado**: 🎯 **60% del MVP completado** - Core services funcionando, enfoque en productos y deploy