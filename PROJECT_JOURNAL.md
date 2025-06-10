# 📖 MARKETPLACE MULTI-TENANT - BITÁCORA DEL PROYECTO

## 🎯 Información del Proyecto

**Proyecto**: Marketplace Multi-Tenant SaaS  
**Objetivo**: Implementar capacidades marketplace sobre arquitectura existente  
**Product Owner**: [NOMBRE]  
**Tech Lead**: [NOMBRE]  
**Inicio**: [FECHA_INICIO]  

---

## 📅 ENTRADAS DE BITÁCORA

### 📝 [FECHA] - Kick-off y Análisis Inicial

#### ✅ Lo que se hizo:
- Análisis completo del codebase existente usando herramientas MCP
- Identificación de servicios: IAM, PIM, Stock, Chat, Backoffice, API Gateway
- Análisis de la arquitectura multi-tenant actual con tenant_id en todas las tablas
- Revisión de patrones existentes: Criteria filtering, automated git hooks
- Identificación del tenant demo hardcodeado: '9a4c3eb9-2471-4688-bfc8-973e5b3e4ce8'

#### 🧠 Decisiones tomadas:
- **Arquitectura híbrida**: Mantener base multi-tenant + agregar capacidades marketplace
- **Taxonomía global**: Separar categorías marketplace de categorías tenant
- **Patrón de mapeo**: Tenants mapean categorías globales a sus nombres específicos
- **Motor de búsqueda**: ElasticSearch para búsquedas cross-tenant
- **Onboarding guiado**: Wizard inteligente por tipo de negocio

#### 🎯 Próximos pasos:
- [ ] Configurar entorno de desarrollo
- [ ] Crear documento técnico detallado (MARKETPLACE_MULTI_TENANT_ROADMAP.md)
- [ ] Definir épicas y tracking (PROJECT_TRACKING.md)
- [ ] Iniciar FASE 1: Taxonomía Global

#### 📊 Métricas establecidas:
- **Target onboarding**: <10 minutos
- **Target búsqueda**: <500ms p95
- **Target adopción**: >85% completion rate
- **Timeline**: 17 semanas

#### 💡 Insights importantes:
- La base multi-tenant existente es sólida y bien diseñada
- El patrón Criteria ya implementado se puede extender para marketplace
- Necesidad de balance entre flexibilidad para sellers y consistencia para buyers
- ElasticSearch será clave para performance en búsquedas cross-tenant

---

### 📝 [2024-12-08] - Análisis y Migración del Sistema Quickstart

#### ✅ Lo que se hizo:
- Análisis completo del sistema quickstart existente (archivos YAML)
- Identificación de 15 tipos de negocio con ~800 categorías y ~200 atributos
- Evaluación de limitaciones: hardcoding, redeploy necesario, no dinamismo
- Propuesta de migración a BD con administración desde backoffice

#### 🧠 Decisiones tomadas:
- **Migración BD**: De archivos YAML estáticos → base de datos dinámica
- **Admin Panel**: Sistema completo para gestionar tipos de negocio y templates
- **Compatibilidad**: Mantener APIs existentes durante transición
- **Performance**: Cache en Redis para mantener velocidad de respuesta

#### 🎯 Beneficios esperados:
- ⚡ **Setup dinámico**: De 2 días dev → 30 min product manager
- 🎯 **Personalización**: Templates por región/industria/nicho
- 📊 **Analytics**: Tracking de uso de templates y A/B testing
- 🔄 **Iteración**: Updates sin redeploy, rollback inmediato

#### 📋 Tareas agregadas al roadmap:
- **FASE 2.0**: 6 tareas migración quickstart a BD
- **FASE 4.0**: 5 tareas admin panel quickstart dinámico
- **Total**: +11 tareas (140 → 145 tareas total)

#### ⚠️ Riesgos identificados:
- **Migración automática**: Conversión YAML → BD sin pérdida de datos
- **Performance**: BD queries vs memoria (mitigado con Redis)
- **Change management**: Training para product managers

#### 🔗 Implementación técnica:
```sql
business_types (BD) -> quickstart_templates (BD) -> tenant_configurations (runtime)
-- Estructura:
-- business_types: id, name, description, icon
-- quickstart_templates: business_type_id, categories[], attributes[], products[]  
-- tenant_configurations: tenant_id, business_type_id, selected_items, created_at
```

---

### 📝 [FECHA] - Creación de Documentación Técnica

#### ✅ Lo que se hizo:
- Creación del roadmap técnico completo (MARKETPLACE_MULTI_TENANT_ROADMAP.md)
- Definición de 7 fases con 134+ tareas específicas
- Especificación de 96+ archivos con código de ejemplo
- Definición de 18 migraciones de base de datos
- Justificaciones funcionales con casos de uso reales (María, Marcos)

#### 🧠 Decisiones tomadas:
- **Enfoque progresivo**: 7 fases claramente delimitadas
- **Granularidad alta**: Cada archivo y endpoint especificado
- **Casos de uso reales**: Sellers argentinos como María (Bahía Blanca) y Marcos (Río Cuarto)
- **Balance funcional-técnico**: Justificaciones de negocio + implementación detallada

#### 🎯 Próximos pasos:
- [ ] Crear sistema de tracking de épicas y tareas
- [ ] Establecer proceso de updates regulares
- [ ] Configurar métricas de avance
- [ ] Comenzar implementación FASE 1

#### 📚 Documentos creados:
- `MARKETPLACE_MULTI_TENANT_ROADMAP.md` - Guía técnica completa
- `PROJECT_TRACKING.md` - Seguimiento de épicas y tareas
- `PROJECT_JOURNAL.md` - Esta bitácora

---

### 📝 [2024-12-08] - Validación de Consistencia y Corrección de Diseño

#### ✅ Lo que se hizo:
- Análisis profundo de servicios existentes vs propuesta marketplace
- Identificación de inconsistencia CRÍTICA: `product_marketplace_attributes` vs `variant_marketplace_attributes`
- Corrección completa del diseño en documentación
- Validación de compatibilidad total con código existente
- Creación de documento de validación de consistencia

#### 🧠 Decisiones tomadas:
- **❌ ERROR CORREGIDO**: Los atributos marketplace van en VARIANTES, no productos
- **✅ DISEÑO CORRECTO**: `variant_marketplace_attributes` conecta a `product_variants`
- **Justificación**: Sistema actual maneja atributos (talle, color) en variantes, búsquedas por variantes con stock
- **Compatibilidad**: Zero breaking changes, solo extensiones

#### 🎯 Impacto de la corrección:
- **Base de datos**: Tabla corregida de productos → variantes
- **Búsquedas**: Filtros por variantes específicas (con stock real)
- **Performance**: Mejor indexación por variantes activas
- **UX**: Filtros muestran solo opciones disponibles

#### 📋 Acciones realizadas:
- [x] Actualizado `MARKETPLACE_MULTI_TENANT_ROADMAP.md` con diseño correcto
- [x] Creado `DATABASE_SCHEMA_EXPLAINED.md` con explicación detallada
- [x] Creado `CONSISTENCY_VALIDATION.md` con análisis completo
- [x] Agregada advertencia prominente sobre corrección aplicada

#### ✅ Validación de servicios existentes:
- **PIM Service**: ✅ Compatible (solo extensiones)
- **IAM Service**: ✅ Sin cambios necesarios  
- **Backoffice**: ✅ Extensible sin breaking changes
- **Kong/Gateway**: ✅ Nuevos endpoints agregados
- **Quickstart**: ✅ Compatible y potenciado

#### 💡 Insights importantes:
- El análisis de código existente previno un error arquitectural mayor
- La estructura actual de variantes es más sólida de lo que pensamos inicialmente
- Plan de migración incremental permite adopción gradual sin riesgo
- Documentación técnica debe validarse contra código real siempre

### 📝 [FECHA] - Setup de Tracking y Monitoreo

#### ✅ Lo que se hizo:
- Implementación de sistema de tracking con 145 tareas específicas
- Definición de métricas de avance por épica
- Establecimiento de proceso de updates semanales
- Identificación de riesgos y dependencias críticas

#### 🧠 Decisiones tomadas:
- **Estados de tarea**: ⏳ Planificado | 🟡 En Progreso | ✅ Completado | ❌ Bloqueado | ⏸️ Pausado
- **Frecuencia updates**: Daily para tareas, Semanal para épicas
- **Métricas clave**: Completion rate, time to market, satisfaction scores
- **Riesgos monitoreados**: Performance, UX adoption, scalabilidad

#### 🎯 Próximos pasos:
- [ ] Configurar entorno de desarrollo local
- [ ] Setup ElasticSearch cluster
- [ ] Preparar datos de prueba
- [ ] Iniciar migración marketplace_categories

#### ⚠️ Riesgos identificados:
- **Alto**: Performance ElasticSearch, Adopción <10min onboarding
- **Medio**: Integración datos existentes, UX complejidad
- **Dependencias**: ElasticSearch antes S7, Sellers beta antes S15

---

### 📝 [2024-12-12] - Creación de Repositorios Frontend Base

#### ✅ Lo que se hizo:
- Creación completa de `saas-mt-marketplace-admin` (puerto 3002)
- Creación completa de `saas-mt-marketplace-frontend` (puerto 3003)
- Reutilización de estilos y componentes del backoffice existente
- Implementación de dashboards base para ambos proyectos
- Configuración de dependencias y estructura base

#### 🧠 Decisiones tomadas:
- **Consistencia visual**: Reutilizar exactamente los estilos de TiendaVecina del backoffice
- **Puertos diferenciados**: Admin 3002, Frontend público 3003, Backoffice 3001
- **Estructura modular**: Componentes UI copiados para independencia de desarrollo
- **Design system unificado**: Misma paleta de colores y componentes base

---

### 📝 [2025-06-09] - Implementación FASE 1: Fundación Marketplace 

#### ✅ Lo que se hizo:
- **Migraciones de Base de Datos**: 4 migraciones completas (008-011)
  - `marketplace_categories` con taxonomía jerárquica y triggers
  - `marketplace_attributes` con tipos y validaciones
  - Tablas de mapeo y extensiones tenant-específicas
  - 50 categorías base + 19 atributos + 140+ valores predefinidos
- **Entidades de Dominio**: 4 entidades principales
  - `MarketplaceCategory` con validaciones y navegación jerárquica
  - `MarketplaceAttribute` con tipos de datos y valores predefinidos
  - `TenantCategoryMapping` para personalización por tenant
  - `TenantAttributeExtension` para valores adicionales por tenant

#### 🧠 Decisiones tomadas:
- **Compatibilidad con Quickstart**: Basado en business-types.yaml existente
- **Arquitectura híbrida**: Categorías globales + personalización tenant
- **Validaciones robustas**: Triggers DB + validaciones Go + constraints
- **Flexibilidad controlada**: Extensiones sin romper consistencia marketplace

#### 🎯 Progreso alcanzado:
- **FASE 1**: 10/32 tareas completadas (31%)
- **Proyecto Total**: 10/145 tareas completadas (7%)
- **Base sólida**: Lista para casos de uso y APIs

#### 📋 Próximos pasos identificados:
1. Completar entidades restantes (category_tree, attribute_value)
2. Implementar casos de uso (create, map, extend, validate)
3. Crear handlers REST para administración
4. Desarrollar componentes frontend para gestión
5. Tests unitarios y de integración

---

### 📝 [2024-12-09] - Integración Docker y Despliegue Completo

#### ✅ Lo que se hizo:
- Configuración completa de Docker para ambos servicios marketplace
- Resolución de conflictos de puertos (Grafana en 3002 → Admin en 3004, Frontend en 3005)
- Creación de Dockerfiles optimizados con multi-stage builds
- Configuración de Next.js standalone output para producción
- Integración exitosa en docker-compose.yml
- Documentación completa de puertos y servicios

#### 🧠 Decisiones tomadas:
- **Puertos finales**: Admin 3004, Frontend 3005 (evitando conflicto con Grafana 3002)
- **Build strategy**: Multi-stage Docker builds para optimización
- **Configuración**: PostCSS y Tailwind CSS v3 (no v4) para compatibilidad
- **Dependencias**: Ambos servicios dependen del API Gateway
- **Documentación**: Archivo `DOCKER_SERVICES_PORTS.md` para referencia

#### 🎯 Logros técnicos:
- ✅ **Builds exitosos**: Ambos servicios compilan sin errores
- ✅ **Docker funcional**: Contenedores ejecutándose correctamente
- ✅ **Servicios respondiendo**: HTTP 200 en ambos puertos
- ✅ **Stack completo**: 17 servicios corriendo simultáneamente
- ✅ **Documentación**: Guía completa de puertos y comandos

#### 📋 Archivos creados/modificados:
- `services/saas-mt-marketplace-admin/Dockerfile`
- `services/saas-mt-marketplace-admin/.dockerignore`
- `services/saas-mt-marketplace-admin/postcss.config.js`
- `services/saas-mt-marketplace-frontend/Dockerfile`
- `services/saas-mt-marketplace-frontend/.dockerignore`
- `services/saas-mt-marketplace-frontend/postcss.config.js`
- `docker-compose.yml` (servicios agregados)
- `DOCKER_SERVICES_PORTS.md` (documentación nueva)

#### 🔧 Problemas resueltos:
- **Conflicto puertos**: Grafana ocupaba 3002 → Cambio a 3004/3005
- **Tailwind v4 syntax**: `@import "tailwindcss"` → `@tailwind` directives
- **Package lock sync**: Regeneración de package-lock.json
- **PostCSS missing**: Configuración de postcss.config.js
- **Autoprefixer**: Instalación de dependencia faltante

#### 🌐 URLs de acceso:
- **Backoffice**: http://localhost:3000
- **Marketplace Admin**: http://localhost:3004
- **Marketplace Frontend**: http://localhost:3005
- **Grafana**: http://localhost:3002
- **API Gateway**: http://localhost:8001

#### 📊 Estado del proyecto:
- **Archivos implementados**: 15/96+ (16%)
- **Servicios frontend**: 3/3 funcionando
- **Infraestructura**: Docker stack completo
- **Repositorios**: Migrados a organización trinityweb
- **Próximo**: Conectar con APIs backend y comenzar FASE 1

---

### 📝 [2024-12-09] - Migración de Repositorios a Organización trinityweb

#### ✅ Lo que se hizo:
- Migración completa de repositorios a organización trinityweb
- Creación de repositorios en https://github.com/orgs/trinityweb/repositories
- Actualización de remotes locales para apuntar a trinityweb
- Actualización de scripts multi-repositorio para incluir nuevos proyectos
- Documentación completa de arquitectura de repositorios

#### 🧠 Decisiones tomadas:
- **Organización centralizada**: Todos los repos bajo trinityweb para mejor gestión
- **URLs actualizadas**: Remotes locales apuntan a trinityweb
- **Scripts actualizados**: multi-repo-manager.sh y quick-repo-status.sh incluyen nuevos repos
- **Makefile mejorado**: Comandos para rebuild rápido de servicios marketplace

#### 🎯 Repositorios migrados:
- ✅ **saas-mt-marketplace-admin**: https://github.com/trinityweb/saas-mt-marketplace-admin
- ✅ **saas-mt-marketplace-frontend**: https://github.com/trinityweb/saas-mt-marketplace-frontend  
- ✅ **mcp-go-generator-node**: https://github.com/trinityweb/mcp-go-generator-node

#### 📋 Archivos actualizados:
- `scripts/multi-repo-manager.sh` - Lista de repositorios actualizada
- `scripts/quick-repo-status.sh` - Incluye nuevos repos en status
- `Makefile` - Comandos rebuild para marketplace services
- `REPOSITORIES.md` - Documentación completa de arquitectura

#### 🔧 Comandos agregados:
- `make rebuild-marketplace-admin` - Rebuild rápido admin
- `make rebuild-marketplace-frontend` - Rebuild rápido frontend
- `make repos-status` - Estado actualizado con nuevos repos

#### 📊 Estado final:
- **14 repositorios** bajo organización trinityweb
- **Sistema multi-repo** funcionando correctamente
- **Documentación** completa de arquitectura
- **Gestión centralizada** lista para desarrollo colaborativo

#### 🎯 Funcionalidades implementadas:
- **Marketplace Admin**: Dashboard con métricas, gestión de taxonomía, quickstart dinámico
- **Marketplace Frontend**: Homepage con búsqueda, grid de productos, filtros avanzados
- **UI Components**: Button, Card, Input, Badge y utilidades base
- **Responsive Design**: Mobile-first con breakpoints optimizados

#### 📊 Progreso del proyecto:
- **Archivos implementados**: 8/96+ (8% completado)
- **Estructura base**: 100% lista para desarrollo
- **Design system**: 100% consistente con ecosystem

#### 💡 Insights importantes:
- La reutilización de componentes del backoffice acelera significativamente el desarrollo
- Tener repositorios separados permite desarrollo paralelo sin conflictos
- La paleta de colores TiendaVecina funciona perfecto para el marketplace público
- Estructura base sólida facilita la implementación de las siguientes fases

#### 🔗 Próximos pasos inmediatos:
- [ ] Implementar migraciones de base de datos (FASE 1.1)
- [ ] Crear entidades de dominio para taxonomía marketplace
- [ ] Conectar admin panel con APIs del PIM service
- [ ] Implementar sistema de búsqueda cross-tenant

---

## 🏃‍♂️ SPRINTS Y ÉPICAS

### 🏛️ ÉPICA 1: TAXONOMÍA MARKETPLACE GLOBAL
**Estado**: ⏳ Planificado | **Progreso**: 0/32 (0%)

#### Sprint Planeado:
- **Semana 1**: Migraciones + Entidades base
- **Semana 2**: Casos de uso + Repositorios  
- **Semana 3**: APIs + Handlers HTTP
- **Semana 4**: Frontend + Tests E2E

#### Notas de planificación:
- Priorizar migraciones primero para tener base sólida
- Entidades deben soportar jerarquías de categorías
- APIs deben permitir tanto admin global como tenant-specific
- Frontend debe ser intuitivo para sellers no técnicos

---

## 🤝 STAKEHOLDERS Y COMUNICACIÓN

### 👥 Equipo Core
- **Product Manager**: Definición features + UX
- **Tech Lead**: Arquitectura + decisiones técnicas
- **Backend Developer**: Go services + APIs
- **Frontend Developer**: React components + UX
- **QA Engineer**: Testing + validación
- **DevOps**: Deployment + monitoreo

### 📞 Rituales de Comunicación
- **Daily Standups**: 9:00 AM (15 min)
- **Sprint Planning**: Lunes (2h)
- **Sprint Review**: Viernes (1h)
- **Retrospectiva**: Viernes (30 min)

### 📊 Reportes Regulares
- **Weekly**: Progreso épicas + blockers
- **Bi-weekly**: Métricas + cronograma
- **Monthly**: Demo + feedback stakeholders

---

## 💡 DECISIONES DE ARQUITECTURA

### 🏗️ Decisiones Tomadas

#### 1. Arquitectura Híbrida Multi-tenant + Marketplace
**Fecha**: [FECHA]  
**Contexto**: Necesidad de mantener aislamiento tenant + capacidad cross-tenant  
**Decisión**: Taxonomía global + mapeos tenant-specific  
**Alternativas consideradas**: 
- Taxonomía completamente tenant-specific (rechazada: no permite búsqueda cross-tenant)
- Taxonomía completamente global (rechazada: no permite personalización)

#### 2. ElasticSearch para Motor de Búsqueda
**Fecha**: [FECHA]  
**Contexto**: Búsquedas cross-tenant con performance <500ms  
**Decisión**: ElasticSearch con indexing async  
**Alternativas consideradas**:
- PostgreSQL full-text search (rechazada: performance limitada)
- Solr (rechazada: complejidad operacional)

#### 3. Onboarding Wizard por Tipo de Negocio
**Fecha**: [FECHA]  
**Contexto**: Reducir abandono en configuración inicial  
**Decisión**: Wizard guiado con recommendations engine  
**Justificación**: 68% de sellers abandonan setup complejo

### 🤔 Decisiones Pendientes

#### 1. Pricing Strategy
**Contexto**: ¿Cómo monetizar capacidades marketplace?  
**Opciones**: Por seller, por transacción, por features  
**Timeline**: Antes del lanzamiento beta

#### 2. Multi-idioma Support
**Contexto**: ¿Soportar múltiples idiomas desde v1?  
**Opciones**: Solo español, español+inglés, full i18n  
**Timeline**: Definir en FASE 4

---

## 🚫 BLOCKERS Y RESOLUCIONES

### ❌ Blockers Activos
*Ninguno actualmente*

### ✅ Blockers Resueltos

#### 1. Definición de Scope
**Blocker**: Alcance muy amplio, riesgo de scope creep  
**Resolución**: Roadmap granular con 7 fases bien delimitadas  
**Fecha resolución**: [FECHA]

#### 2. Complejidad Técnica
**Blocker**: Integración cross-tenant compleja  
**Resolución**: Arquitectura híbrida manteniendo aislamiento  
**Fecha resolución**: [FECHA]

---

## 📚 APRENDIZAJES Y INSIGHTS

### 💡 Insights Técnicos
- **Multi-tenancy bien diseñado**: La arquitectura existente es sólida
- **Patrón Criteria potente**: Se puede extender elegantemente
- **Performance crítica**: Búsquedas <500ms son make-or-break
- **ElasticSearch necessary**: PostgreSQL no alcanza para cross-tenant search

### 🎯 Insights de Producto
- **Onboarding es crítico**: 10 minutos max o los sellers abandonan
- **Personalización importante**: Sellers quieren reflejar su marca
- **Simplicidad over features**: Mejor menos features pero bien hechas
- **Analytics son diferenciador**: Competencia no ofrece insights útiles

### 👥 Insights de Equipo
- **Documentación crítica**: Proyecto complejo requiere specs detalladas
- **Comunicación async**: Bitácora permite mantener contexto
- **Granularidad ayuda**: Tareas pequeñas dan sensación de progreso

---

## 🔄 CAMBIOS DE SCOPE

### Cambios Aprobados
*Ninguno aún*

### Cambios Propuestos
*Ninguno aún*

---

## 📊 MÉTRICAS Y KPIs

### 📈 Métricas de Desarrollo
- **Velocity**: [CALCULAR] story points por sprint
- **Quality**: [MEDIR] % tests passing, coverage
- **Deployment**: [TRACKING] deployment frequency, lead time

### 🎯 Métricas de Producto (Post-lanzamiento)
- **Adoption**: % sellers que completan onboarding
- **Engagement**: % sellers activos semanalmente  
- **Satisfaction**: NPS score, support tickets
- **Performance**: Time to first product, search latency

---

## 🔗 ENLACES ÚTILES

### 📁 Documentación
- [Roadmap Técnico](./MARKETPLACE_MULTI_TENANT_ROADMAP.md)
- [Tracking de Épicas](./PROJECT_TRACKING.md)
- [Análisis Arquitectura Actual](../README.md)

### 🛠️ Herramientas
- **Repo**: [URL_REPO]
- **Board**: [URL_JIRA/GITHUB_PROJECTS]
- **Monitoring**: [URL_GRAFANA]
- **Docs**: [URL_CONFLUENCE]

### 🎯 Referencias
- **Competitor Analysis**: MercadoLibre, Amazon Marketplace
- **Tech Stack**: Go, React, PostgreSQL, ElasticSearch
- **Deployment**: Kubernetes, Docker

---

## 📝 TEMPLATES

### Template Update Semanal
```markdown
## Update [FECHA]

### ✅ Completado esta semana
- [ÉPICA] [TAREA]: Descripción breve

### 🟡 En progreso
- [ÉPICA] [TAREA]: Status y % completado

### 🔄 Próxima semana  
- [ÉPICA] [TAREA]: Prioridad y owner

### 🚨 Blockers/Issues
- **Blocker**: Descripción y plan de resolución

### 📊 Métricas clave
- **Progreso general**: X/134 tareas (Y%)
- **Épica actual**: X/N tareas
- **Performance**: [metric] 

### 💭 Notas
- Observaciones importantes
- Decisiones tomadas
- Insights aprendidos
```

### Template Retrospectiva Sprint
```markdown
## Retrospectiva Sprint [NÚMERO] - [FECHA]

### 🚀 ¿Qué funcionó bien?
- Item 1
- Item 2

### 🐛 ¿Qué no funcionó?
- Item 1
- Item 2

### 💡 ¿Qué podemos mejorar?
- Acción 1 (Owner: [NOMBRE])
- Acción 2 (Owner: [NOMBRE])

### 📊 Métricas del Sprint
- **Tareas completadas**: X/Y
- **Velocity**: Z story points
- **Blockers promedio**: N días

### 🎯 Focus próximo sprint
- Prioridad 1
- Prioridad 2
```

---

## 📝 NOTAS RÁPIDAS

### 💭 Ideas para Considerar
- Integración con redes sociales para sellers
- Sistema de reviews/ratings de productos
- Herramientas de marketing para sellers
- Analytics predictivos con ML

### 🔖 Para Investigar
- Performance patterns en ElasticSearch
- UX best practices para onboarding
- Pricing models de competencia
- Frameworks de testing E2E

### 📌 Recordatorios
- Configurar ElasticSearch antes S7
- Contactar sellers beta antes S15
- Preparar datos demo para testing
- Documentar decisiones de arquitectura

---

*Última actualización: [FECHA]*  
*Próxima review: [FECHA]*  
*Mantenido por: [NOMBRE]* 