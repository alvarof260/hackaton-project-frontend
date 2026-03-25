# PRD: CommiFlow - Versión Hackathon (MVP)

## deadline: 7 días | equipo: 3 personas

> **NOTA**: Este documento es un subset del PRD principal. Mantiene la visión del producto pero con scope reducido para una hackathon de 7 días.

---

## 1. Scope Reducido — Lo que SÍ entra

| Módulo | Feature | Prioridad |
|--------|---------|-----------|
| **Auth** | Registro/Login con JWT | P0 |
| **Muro Público** | Perfil del artista (avatar, bio, banner) | P0 |
| **Muro Público** | Galería de imágenes | P0 |
| **Muro Público** | Catálogo de Tiers | P0 |
| **Muro Público** | Calculadora de extras (frontend) | P0 |
| **Muro Público** | Cola de trabajo pública (solo lectura) | P0 |
| **Compra** | Cliente envía requerimiento con referencias | P0 |
| **Compra** | Backend valida precio (recalculo) | P0 |
| **Tablero Artista** | Bandeja de pedidos entrantes | P0 |
| **Tablero Artista** | Aceptar / Rechazar pedido | P0 |
| **Tablero Artista** | Cambiar estado del pedido | P0 |
| **Pagos** | UNA pasarela (MercadoPago O Stripe) | P1 |
| **Pagos** | Webhook de confirmación | P1 |

---

## 2. Lo que QUEDA FUERA (Postergado)

| Feature | Original PRD | Razón |
|---------|--------------|-------|
| 3 pasarelas de pago | RF-15, RF-16 | Tiempo de integración x3 |
| Contratos auto-generados | RF-11 | Complejidad legal + PDF |
| Marca de agua en imágenes | RF-12 | Processing dinámico innecesario |
| AWS S3 / R2 con Presigned URLs | RNF-04 | Config infra, no código |
| Locking transaccional de slots | RNF-05 | Boolean simple alcanza |
| Múltiples monedas por artista | RF-05 | Una moneda por cuenta |

---

## 3. Arquitectura Simplificada

### Stack
- **Frontend**: React 19 (SPA) + Zustand
- **Backend**: .NET 10 (Minimal APIs)
- **DB**: PostgreSQL
- **Auth**: JWT (sin IdentityServer complejo)

### Endpoints Core

```
PÚBLICO (sin auth)
GET  /api/v1/artists/{username}           → Perfil + disponibilidad
GET  /api/v1/artists/{username}/tiers    → Catálogo + addons
GET  /api/v1/artists/{username}/gallery   → Imágenes del portfolio
GET  /api/v1/artists/{username}/queue    → Cola de pedidos activos

CLIENTE (loguedo)
POST /api/v1/commissions                  → Crear pedido

ARTISTA (loguedo + rol)
GET  /api/v1/dashboard/commissions       → Bandeja de entrada
PATCH /api/v1/dashboard/commissions/{id}/accept
PATCH /api/v1/dashboard/commissions/{id}/reject
PATCH /api/v1/dashboard/commissions/{id}/status   → Cambiar estado

WEBHOOK
POST /api/v1/webhooks/{provider}        → Confirmación de pago
```

---

## 4. Modelo de Datos (Simplificado)

```
Users
├── Id (UUID)
├── Username (unique)
├── Email
├── PasswordHash
└── Role (Artist | Client)

ArtistProfiles
├── Id (UUID)
├── UserId (FK)
├── DisplayName
├── Bio
├── BannerUrl
├── AvatarUrl
├── IsQueueOpen (boolean)
└── PaymentProvider (MercadoPago | Stripe)

Tiers
├── Id (UUID)
├── ArtistId (FK)
├── Name
├── Description
├── BasePrice
├── EstimatedDays
└── IsActive

Addons
├── Id (UUID)
├── TierId (FK)
├── Name
├── Type (Fixed | Percentage)
└── Value

Commissions
├── Id (UUID)
├── ClientId (FK)
├── ArtistId (FK)
├── TierId (FK)
├── Status (PendingApproval | AwaitingPayment | InQueue | WIP | Completed | Cancelled)
├── ClientBrief
├── ReferenceImages (JSON array)
├── FinalPrice
└── SnapshotData (JSONB)

Payments
├── Id (UUID)
├── CommissionId (FK)
├── Provider
├── ProviderTransactionId
├── Amount
└── Status (Pending | Approved | Refunded)
```

---

## 5. Flujo Happy Path (Hackathon)

```
1. Artista se registra y configura su perfil
2. Artista crea Tiers (ej: "Boceto $50", "Full Color $150")
3. Artista crea Addons (ej: "+1 Personaje +$30", "Comercial x2")
4. Cliente visita /{username}
5. Cliente selecciona Tier + Extras → Calculadora muestra precio
6. Cliente completa brief + sube referencias → POST /commissions
7. Commission queda en "PendingApproval"
8. Artista ve el pedido en su dashboard
9. Artista hace clic en "Aceptar" → Se genera link de pago
10. Cliente paga → Webhook actualiza estado a "InQueue"
11. Cliente ve su pedido en la cola pública
12. Artista trabaja → cambia estado: WIP → Completed
13. Cliente recibe notificación (email o simple alert)
```

---

## 6. Notas de Implementación

### Imágenes (Simplificado)
- Para la hackathon: guardar URLs en la DB o serveo estático básico
- NO configurar S3/R2/Cloudinary
- Si necesitan storage: usar servicio gratuito como imgur o similar para testing

### Pagos (UNA sola pasarela)
- **Recomendado**: MercadoPago (si es audience argentino) o Stripe (internacional)
- Webhook minimal: solo處理 "payment_approved" o "checkout.session.completed"
- NO hacer refunds ni disputas por ahora

### Slots (Simplificado)
- Boolean `IsQueueOpen` en ArtistProfile
- Si está cerrado, el artista no acepta nuevos pedidos
- NO implementar race condition handling

### Testing
- 2 días: Auth + CRUD básico de perfiles y tiers
- 2 días: Calculadora + flujo de compra
- 2 días: Dashboard + estados + cola pública
- 1 día: Integración de pago + testing end-to-end

---

## 7. Definition of Done (Hackathon)

- [ ] Usuario puede registrarse como Artist o Client
- [ ] Artista puede crear/editar Tiers y Addons
- [ ] Cliente ve catálogo con calculadora funcional
- [ ] Cliente puede crear un pedido con referencias
- [ ] Artista ve bandeja de pedidos
- [ ] Artista puede aceptar/rechazar pedidos
- [ ] Artista puede cambiar estado del pedido
- [ ] Cola pública muestra pedidos activos
- [ ] Una pasarela de pago funciona end-to-end

---

## 8. Distribución de Tareas Sugerida

| Persona | Backend | Frontend |
|---------|---------|----------|
| Dev 1 | Auth + Models + Repositories | Components básicos + Router |
| Dev 2 | Tiers/Addons CRUD + Calculator logic | Muro Público + Calculadora |
| Dev 3 | Commissions + Dashboard + Webhooks | Tablero Artista + Cola Pública |

**Dev 1 y Dev 2 ayudan con testing de la pasarela al final.**

---

*Documento creado para hackathon de 7 días. Scope realista, no ambicioso.*
