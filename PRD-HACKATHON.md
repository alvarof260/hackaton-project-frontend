# PRD: CommiFlow — Versión Hackathon (MVP)

## Deadline: 7 días | Equipo: 3 personas | Deploy: CubePath

> **NOTA**: Este documento es un subset del PRD principal ([PRD.md](./PRD.md)). Mantiene la visión del producto pero con scope reducido para una hackathon de 7 días. El documento completo de referencia es el [SRS.md](./SRS.md).

> **RFC 2119**: Las palabras clave "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY" y "OPTIONAL" se interpretan según [RFC 2119](https://www.rfc-editor.org/rfc/rfc2119).

---

## 1. Problem Statement

Vender arte por encargo en 2026 sigue siendo un caos organizativo para artistas independientes. El flujo de trabajo actual está roto:

- **Fricción en la cotización**: Mensajes interminables por Instagram/X para presupuestar. El artista calcula mentalmente, explica precios, y reza para que el cliente no desaparezca.
- **Ansiedad del cliente**: Sin sistema de seguimiento transparente, el cliente interrumpe constantemente pidiendo actualizaciones.
- **Gestión de pagos manual**: Links sueltos de Mercado Pago, comprobantes revisados a mano, miedo constante al chargeback.
- **Desorganización de slots**: "Comisiones Abiertas (3/5)" en un tweet que a las 2 horas queda sepultado.

La mayoría de los artistas resuelve esto con un frankenstein de herramientas: Linktree, Google Docs para precios, Trello público para la cola, y DMs para negociar.

**CommiFlow elimina este frankenstein.** Unificamos catálogo, calculadora de precios dinámica, cola de trabajo pública y pasarela de pagos en una sola herramienta diseñada para este nicho.

---

## 2. Objetivos del MVP (Hackathon)

### 2.1. Objetivo Principal

Entregar un producto mínimo viable funcional que demuestre el flujo core end-to-end: artista configura su tienda → cliente cotiza → cliente paga → artista gestiona pedidos.

### 2.2. Criterios de Evaluación

| Criterio | Peso estimado | Enfoque |
|----------|---------------|---------|
| **Originalidad** | Alto | Nicho específico, no es otro e-commerce genérico |
| **Calidad de código** | Alto | Arquitectura limpia, tipado fuerte, separación de concerns |
| **Uso de CubePath** | Alto | Deploy real en CubePath VPS, aprovechar API/CLI para infra |

### 2.3. Definición de Éxito (MUSTs del MVP)

El MVP MUST ser capaz de demostrar el siguiente flujo completo ante el jurado:

1. Un artista se registra y configura su perfil público (avatar, bio, galería, tiers, addons)
2. Un cliente visita `/{username}` y ve el muro público del artista
3. El cliente selecciona un tier, agrega extras, y la calculadora muestra el precio en tiempo real
4. El cliente completa el brief con descripción y referencias visuales, y envía el pedido
5. El artista ve el pedido en su dashboard, lo acepta
6. Se genera un link de pago de MercadoPago
7. (Demo) El pago se simula/confirma → el pedido pasa a la cola pública
8. El artista cambia el estado del pedido (WIP → Completed)

---

## 3. Scope — Lo que SÍ entra

| Módulo | Feature | Prioridad | RF Referencia |
|--------|---------|-----------|---------------|
| **Auth** | Registro/Login con JWT | MUST | RF-01, RF-02 |
| **Auth** | Perfil base (username slug, display name) | MUST | RF-03 |
| **Muro Público** | Perfil del artista (avatar, bio, banner) | MUST | RF-03 |
| **Muro Público** | Galería de imágenes (upload real) | MUST | RF-04 |
| **Muro Público** | Catálogo de Tiers (CRUD artista) | MUST | RF-05 |
| **Muro Público** | Addons por Tier (CRUD artista) | MUST | RF-06 |
| **Muro Público** | Calculadora de extras (frontend React) | MUST | RF-07 |
| **Muro Público** | Cola de trabajo pública (solo lectura) | MUST | RF-13 |
| **Compra** | Cliente envía requerimiento con referencias (upload real) | MUST | RF-08, RF-09 |
| **Compra** | Backend valida precio (recalculo server-side) | MUST | RNF-02 |
| **Tablero Artista** | Bandeja de pedidos entrantes | MUST | RF-10 |
| **Tablero Artista** | Aceptar pedido → genera link de pago | MUST | RF-11, RF-16 |
| **Tablero Artista** | Rechazar pedido (con motivo opcional) | SHOULD | RF-11 |
| **Tablero Artista** | Cambiar estado del pedido | MUST | RF-12 |
| **Pagos** | MercadoPago: generación de preferencia | MUST | RF-16 |
| **Pagos** | MercadoPago: webhook de confirmación | MUST | RF-17 |
| **Deploy** | Aplicación corriendo en CubePath VPS | MUST | — |
| **Deploy** | Demostrar uso de CubePath API/CLI en el proceso | SHOULD | — |

---

## 4. Lo que QUEDA FUERA (Postergado)

| Feature | Razón | RF Referencia |
|---------|-------|---------------|
| Stripe / PayPal | Tiempo de integración x3. UNA pasarela alcanza para demo. | RF-15, RF-16 |
| Contratos auto-generados | Complejidad legal + PDF generation. Scope creep. | RF-11 (SRS) |
| Marca de agua en previews | Processing dinámico innecesario para MVP. | RF-12 (SRS) |
| Múltiples monedas por artista | Una moneda (ARS) por cuenta para MVP. | RF-05 (SRS) |
| Locking transaccional de slots | Boolean `IsQueueOpen` alcanza. Race conditions no críticas en demo. | RNF-05 |
| Cliente anónimo en cola | Nice-to-have, no afecta el demo. | RF-14 (SRS) |
| Notificaciones (email/push) | Fuera de scope. El estado se ve en el dashboard y la cola. | — |
| Sistema de reviews/ratings | Post-MVP. | — |
| Admin backoffice | No necesario para demo de 7 días. | — |
| SEO / SSR (Next.js) | SPA con React alcanza. El muro público es cacheable. | — |
| AWS S3/R2 con Presigned URLs | Upload directo a storage local o servicio simple en CubePath. | RNF-04 |

---

## 5. Decisiones Técnicas (Tech Stack)

### 5.1. Frontend: React 19 (SPA) + Zustand

- **MUST** usar React como SPA desacoplada del backend.
- **SHOULD** usar Zustand para estado global de la calculadora y flujo de compra.
- **MUST** usar React Router v6 para navegación entre muro público y dashboard.
- La calculadora **MUST** recalcular el precio en el cliente sin round-trips al servidor (UX inmediata).
- **Trade-off aceptado**: Sin SSR, sacrificamos SEO dinámico. Ganamos simplicidad de deploy.

### 5.2. Backend: .NET 10 (Minimal APIs)

- **MUST** usar .NET 10 con Minimal APIs.
- **MUST** implementar arquitectura en capas: Endpoints → Services → Repositories.
- **MUST** usar Entity Framework Core como ORM con PostgreSQL.
- **MUST** usar Inyección de Dependencias nativa de .NET.

### 5.3. Base de Datos: PostgreSQL

- **MUST** usar PostgreSQL como base de datos relacional.
- **SHOULD** correr en el mismo CubePath VPS que el backend para simplificar el deploy.

### 5.4. Autenticación: JWT

- **MUST** implementar autenticación basada en JWT (access tokens).
- **SHOULD** incluir refresh tokens para UX aceptable en la demo.
- **MUST** hashear contraseñas con BCrypt o Argon2.

### 5.5. Deploy: CubePath

- **MUST** deployar la aplicación en un CubePath Cloud VPS.
- **SHOULD** usar CubePath API o CubeCLI para provisionar infraestructura como parte del flujo de hackathon (criterio de evaluación).
- **RECOMMENDED**: VPS con Docker Compose (app + PostgreSQL) para simplicidad.
- CubePath ofrece: deploy en 30s, billing por hora ($0.007/h), DDoS protection incluido, API REST pública, CLI open-source.

---

## 6. Modelo de Datos

```
Users
├── Id (UUID, PK)
├── Username (String, Unique) — slug para URL pública: /{username}
├── Email (String, Unique)
├── PasswordHash (String)
├── Role (Enum: Artist | Client)
└── CreatedAt (DateTime UTC)

ArtistProfiles
├── Id (UUID, PK)
├── UserId (UUID, FK → Users, Unique)
├── DisplayName (String)
├── Bio (Text)
├── BannerUrl (String, nullable)
├── AvatarUrl (String, nullable)
├── IsQueueOpen (Boolean — control maestro para pausar comisiones)
└── PaymentProvider (Enum: MercadoPago)
    └── MercadoPagoAccessToken (String, encriptado)

Tiers
├── Id (UUID, PK)
├── ArtistId (UUID, FK → ArtistProfiles)
├── Name (String — ej: "Boceto", "Full Color")
├── Description (Text)
├── BasePrice (Decimal 18,2)
├── EstimatedDays (Int)
└── IsActive (Boolean)

Addons
├── Id (UUID, PK)
├── TierId (UUID, FK → Tiers)
├── Name (String — ej: "+1 Personaje", "Uso Comercial")
├── Type (Enum: Fixed | Percentage)
└── Value (Decimal 18,2)

Commissions
├── Id (UUID, PK)
├── ClientId (UUID, FK → Users)
├── ArtistId (UUID, FK → ArtistProfiles)
├── TierId (UUID, FK → Tiers)
├── Status (Enum — ver §6.1)
├── ClientBrief (Text)
├── ReferenceImages (JSON array de URLs)
├── FinalPrice (Decimal 18,2 — precio validado por backend)
└── SnapshotData (JSONB — foto del precio al momento de la compra)

Payments
├── Id (UUID, PK)
├── CommissionId (UUID, FK → Commissions)
├── Provider (Enum: MercadoPago)
├── ProviderTransactionId (String — ID que devuelve la pasarela)
├── Amount (Decimal 18,2)
└── Status (Enum: Pending | Approved | Refunded)
```

### 6.1. Estados de Comisión (State Machine)

```
                    ┌─────────────────┐
                    │ PendingApproval │ ← Estado inicial al crear pedido
                    └────────┬────────┘
                             │
              ┌──────────────┼──────────────┐
              │              │              │
              ▼              │              ▼
     ┌────────────────┐      │     ┌───────────┐
     │ AwaitingPayment│      │     │ Cancelled │
     └───────┬────────┘      │     └───────────┘
             │               │     (artista rechaza)
             │               │
     (cliente paga)    (artista rechaza)
             │
             ▼
     ┌─────────┐
     │ InQueue │ ← Visible en cola pública
     └────┬────┘
          │
          ▼
     ┌─────────┐
     │   WIP   │ ← Artista está trabajando
     └────┬────┘
          │
          ▼
     ┌────────────┐
     │ Completed  │ ← Entregado
     └────────────┘
```

| Estado | Descripción | Quién transiciona |
|--------|-------------|-------------------|
| `PendingApproval` | Pedido creado, esperando revisión del artista | Sistema (al crear) |
| `AwaitingPayment` | Artista aceptó, esperando que el cliente pague | Artista (al aceptar) |
| `InQueue` | Pago confirmado, pedido en la cola pública | Sistema (webhook de pago) |
| `WIP` | Artista está trabajando en el pedido | Artista |
| `Completed` | Pedido entregado | Artista |
| `Cancelled` | Pedido rechazado o cancelado | Artista (rechazar) o Sistema (timeout futuro) |

**Transiciones válidas** (MUST validarse en backend):
- `PendingApproval` → `AwaitingPayment` (artista acepta)
- `PendingApproval` → `Cancelled` (artista rechaza)
- `AwaitingPayment` → `InQueue` (webhook confirma pago)
- `InQueue` → `WIP` (artista inicia trabajo)
- `WIP` → `Completed` (artista entrega)

Cualquier otra transición **MUST** ser rechazada por el backend con HTTP 409 Conflict.

---

## 7. API Endpoints

### 7.1. Público (Sin autenticación)

```
GET  /api/v1/artists/{username}
     → Perfil del artista + estado de la cola (IsQueueOpen)
     → Response: ArtistProfile DTO

GET  /api/v1/artists/{username}/tiers
     → Catálogo de servicios con sus addons
     → Response: List<TierWithAddonsDTO>

GET  /api/v1/artists/{username}/gallery
     → Imágenes del portfolio
     → Response: List<GalleryImageDTO>

GET  /api/v1/artists/{username}/queue
     → Cola de pedidos activos (solo tier name + estado, sin datos sensibles)
     → Response: List<QueueItemDTO>
     → SHOULD paginar con query params: ?page=1&pageSize=10
```

### 7.2. Cliente (Autenticado)

```
POST /api/v1/commissions
     → Crear pedido
     → Body: { tierId, addonIds[], clientBrief, referenceImageUrls[] }
     → Backend MUST recalcular el precio (NO confiar en el frontend)
     → Backend MUST guardar SnapshotData con la composición del precio
     → Response: CommissionDTO (con status: PendingApproval)
```

### 7.3. Artista (Autenticado + Rol Artist)

```
POST /api/v1/catalog/tiers
     → Crear tier
     → Body: { name, description, basePrice, estimatedDays }

PUT  /api/v1/catalog/tiers/{id}
     → Editar tier

DELETE /api/v1/catalog/tiers/{id}
     → Eliminar (soft delete → IsActive = false) o hard delete

POST /api/v1/catalog/tiers/{tierId}/addons
     → Crear addon para un tier

GET  /api/v1/dashboard/commissions
     → Bandeja de entrada (pedidos filtrados por estado)
     → SHOULD soportar filtro: ?status=PendingApproval

PATCH /api/v1/dashboard/commissions/{id}/accept
     → Aceptar pedido
     → Backend genera link de pago con SDK de MercadoPago
     → Response: { commissionId, paymentUrl, newStatus: "AwaitingPayment" }

PATCH /api/v1/dashboard/commissions/{id}/reject
     → Rechazar pedido
     → Body (OPTIONAL): { reason: "..." }
     → newStatus: "Cancelled"

PATCH /api/v1/dashboard/commissions/{id}/status
     → Cambiar estado del pedido
     → Body: { newStatus: "WIP" | "Completed" }
     → Backend MUST validar que la transición es válida (ver §6.1)
     → MUST retornar 409 si la transición no es válida
```

### 7.4. Webhooks (Sin autenticación por token, verificado por firma)

```
POST /api/v1/webhooks/mercadopago
     → MercadoPago envía notificación de pago
     → Backend MUST verificar firma HMAC del webhook
     → Al recibir "payment_approved":
       1. Buscar Commission por external_reference
       2. Validar que está en AwaitingPayment
       3. Crear registro en Payments
       4. Cambiar Commission status → InQueue
       5. Retornar HTTP 200
     → MUST retornar 200 rápido (< 5s) para que MercadoPago no reintente
```

---

## 8. Reglas de Negocio (Business Rules)

### BR-01: Recalculo de precio (CRITICAL)
El frontend calcula el precio para mostrar al usuario. El backend **MUST** recalcular el precio de forma independiente al recibir `POST /commissions`. Si el precio del frontend no coincide con el del backend (tolerancia: $0.01), el backend **MUST** usar SIEMPRE su propio cálculo y guardarlo en `FinalPrice` y `SnapshotData`.

### BR-02: Snapshot inmutable
Cuando se crea una Commission, `SnapshotData` **MUST** contener una copia exacta del nombre del tier, precio base, y cada addon con su nombre y precio al momento de la compra. Si el artista modifica los precios después, los pedidos existentes NO cambian.

### BR-03: IsQueueOpen
Si `ArtistProfile.IsQueueOpen = false`, el endpoint `POST /commissions` para ese artista **MUST** retornar HTTP 403 con mensaje "El artista no está aceptando comisiones actualmente".

### BR-04: Solo un pago activo por comisión
Si una Commission ya tiene un Payment en estado `Approved`, el backend **MUST NOT** generar un nuevo link de pago. Retornar HTTP 409.

### BR-05: Validación de transiciones
El backend **MUST** validar que cada cambio de estado de Commission siga la máquina de estados definida en §6.1. Transiciones inválidas **MUST** retornar HTTP 409.

---

## 9. Flujo Happy Path (Paso a paso)

```
1. Artista se registra (rol: Artist)
   └→ JWT token de respuesta

2. Artista configura su perfil
   └→ DisplayName, Bio, Avatar, Banner, IsQueueOpen = true

3. Artista crea Tiers
   └→ Ej: "Boceto" ($50, 3 días), "Full Color" ($150, 7 días)

4. Artista crea Addons por cada Tier
   └→ Ej: "+1 Personaje" (Fixed, $30), "Uso Comercial" (Percentage, x2)

5. Cliente visita /{username}
   └→ Ve perfil, galería, catálogo, cola pública

6. Cliente selecciona Tier + Extras
   └→ Calculadora React muestra precio en tiempo real

7. Cliente completa brief + sube hasta 5 imágenes de referencia
   └→ POST /api/v1/commissions

8. Backend recalcula precio, guarda SnapshotData, crea Commission
   └→ Status: PendingApproval

9. Artista ve el pedido en su dashboard
   └→ GET /api/v1/dashboard/commissions?status=PendingApproval

10. Artista acepta el pedido
    └→ PATCH /dashboard/commissions/{id}/accept
    └→ Backend genera preferencia en MercadoPago
    └→ Response: { paymentUrl }
    └→ Status: AwaitingPayment

11. Cliente abre el link de pago y paga
    └→ MercadoPago procesa el pago

12. MercadoPago envía webhook a /api/v1/webhooks/mercadopago
    └→ Backend verifica firma, crea Payment, cambia status → InQueue

13. Cliente y público ven el pedido en la cola pública
    └→ GET /api/v1/artists/{username}/queue

14. Artista inicia trabajo
    └→ PATCH /dashboard/commissions/{id}/status { newStatus: "WIP" }

15. Artista entrega
    └→ PATCH /dashboard/commissions/{id}/status { newStatus: "Completed" }
```

---

## 10. Imágenes (Storage Simplificado)

Para la hackathon:
- Las imágenes (galería del artista + referencias del cliente) **SHOULD** almacenarse en el servidor del CubePath VPS o en un servicio externo gratuito.
- **RECOMMENDED**: Servir imágenes estáticamente desde el mismo VPS (carpeta `/uploads` servida por Nginx o por el propio .NET como static files).
- **SHOULD NOT** configurar S3/R2/Cloudinary para el MVP. Es overkill para una demo.
- **MUST** validar tipo MIME y tamaño máximo (5MB por imagen) en el backend.

---

## 11. Pagos (MercadoPago)

- **Pasarela**: MercadoPago (única para el MVP).
- **Flujo**: El artista acepta → backend crea una "Preferencia" de MercadoPago vía SDK → devuelve URL de pago al frontend → cliente paga en la página de MercadoPago → webhook confirma.
- **Webhook minimal**: Solo procesar evento `payment` con status `approved`. Ignorar el resto.
- **MUST NOT** hacer refunds ni manejar disputas en el MVP.
- **MUST** verificar la firma del webhook para evitar falsificaciones.

---

## 12. Deploy en CubePath

### 12.1. Arquitectura de Deploy

```
CubePath Cloud VPS (1 instancia)
├── Docker Compose
│   ├── app (.NET 10 API + React SPA servida como static files)
│   └── postgres (PostgreSQL 16+)
├── Nginx (reverse proxy + TLS)
└── Volumen persistente (PostgreSQL data + uploads)
```

### 12.2. Uso de CubePath (Criterio de Evaluación)

Para destacar en el criterio "Uso de CubePath":
- **SHOULD** usar CubeCLI o CubePath API para provisionar el VPS como parte del proceso de hackathon (script automatizado).
- **SHOULD** documentar el proceso de deploy en CubePath (screenshots, comandos CubeCLI usados).
- **RECOMMENDED**: Crear un script `deploy.sh` que use `cubecli` para:
  1. Crear VPS
  2. Instalar Docker
  3. Clonar repo
  4. Levantar Docker Compose
- Las locations disponibles (Barcelona, Houston, Miami) **SHOULD** elegirse según la latencia objetivo.

---

## 13. Distribución de Tareas

> **Premisa**: 2 devs son más fuertes en backend, 1 es más fuerte en frontend.

| Persona | Responsabilidad Backend | Responsabilidad Frontend | Días 1-2 | Días 3-4 | Días 5-6 | Día 7 |
|---------|------------------------|-------------------------|----------|----------|----------|-------|
| **Dev 1** (Backend) | Auth (JWT), Users, Models, DB setup, Migrations | Router base + Auth pages (Login/Register) | Auth completo | Endpoints de catálogo | Dashboard endpoints | Testing + Bug fixes |
| **Dev 2** (Backend) | Tiers/Addons CRUD, Price Calculator logic, Commission CRUD | Muro Público (perfil, galería, cola) | Models + DB | Calculator backend + Commission creation | Accept/Reject + Status transitions | MercadoPago integration |
| **Dev 3** (Frontend) | Webhook MercadoPago, Deploy scripts | Calculadora React, Commission form, Dashboard UI | React setup + Components básicos | Calculadora + Muro | Dashboard completo | Deploy en CubePath + E2E testing |

**NOTAS**:
- Dev 1 y Dev 2 DEBEN ayudar con el frontend cuando el backend esté estable (día 5+).
- Dev 3 DEBE tener el webhook de MercadoPago como único task de backend (aislado, testeable).
- El día 7 es sagrado para integración, testing end-to-end y deploy en CubePath.

---

## 14. Timeline Sugerido

| Día | Fase | Entregable |
|-----|------|-----------|
| **1** | Setup + Auth | Repo creado, .NET + React bootstrapped, Auth JWT funcional, DB con migraciones |
| **2** | Catálogo + Galería | CRUD de Tiers, Addons, upload de galería. Muro público muestra perfil + catálogo |
| **3** | Calculadora + Compra | Calculadora React funcional (tiempo real). Formulario de commission con upload de referencias |
| **4** | Commission + Backend validation | POST /commissions con recalculo de precio. SnapshotData guardado correctamente |
| **5** | Dashboard Artista | Bandeja de pedidos. Accept/Reject. Cambio de estados. Cola pública visible |
| **6** | MercadoPago + Webhook | Generación de preferencia. Webhook de confirmación. Flujo end-to-end de pago |
| **7** | Deploy + Testing + Polish | Deploy en CubePath. E2E testing. Bug fixes. Preparación de demo |

---

## 15. Definition of Done (Hackathon)

Cada item **MUST** estar verificado antes de la presentación:

- [ ] Usuario puede registrarse como Artist o Client
- [ ] Artista puede configurar su perfil público (avatar, bio, banner, galería)
- [ ] Artista puede crear/editar/eliminar Tiers y Addons
- [ ] Cliente puede visitar /{username} y ver el muro público
- [ ] Calculadora de extras recalcular el precio en tiempo real (frontend)
- [ ] Cliente puede crear un pedido con brief + imágenes de referencia (upload real)
- [ ] Backend recalcula y valida el precio al crear la comisión
- [ ] Artista ve bandeja de pedidos entrantes en su dashboard
- [ ] Artista puede aceptar un pedido (genera link de MercadoPago)
- [ ] Artista puede rechazar un pedido
- [ ] Artista puede cambiar estado del pedido (InQueue → WIP → Completed)
- [ ] Webhook de MercadoPago confirma el pago y actualiza el estado
- [ ] Cola pública muestra pedidos activos con su estado
- [ ] Aplicación deployada y accesible en CubePath VPS
- [ ] Transiciones de estado inválidas retornan 409

---

## 16. Riesgos y Mitigaciones

| Riesgo | Probabilidad | Impacto | Mitigación |
|--------|-------------|---------|------------|
| SDK de MercadoPago tiene bugs/documentación confusa | Alta | Alto | Dedicar día 6 completo. Tener fallback de simular pago manualmente para la demo. |
| Upload de imágenes consume tiempo | Media | Medio | Usar upload local al VPS, no S3. Servir como static files. |
| .NET 10 tiene breaking changes o bugs | Baja | Alto | Usar features estables. Minimal APIs + EF Core son maduras. |
| CubePath deploy falla el día 7 | Media | Alto | Tener Docker Compose funcionando localmente ANTES del deploy. CubePath es solo "subirlo". |
| Scope creep (agregar features no planeadas) | Alta | Alto | Este documento es el contrato. Cualquier feature nueva pasa por revisión de scope. |

---

## 17. Decisiones Asumidas (Assumptions)

1. Moneda única: ARS (Peso Argentino) para todo el MVP.
2. Un solo rol por usuario (Artist o Client, no ambos).
3. No hay sistema de notificaciones. El estado se consulta activamente.
4. No hay paginación obligatoria en MVP (SHOULD, no MUST), pero los endpoints la soportan si hay tiempo.
5. Las imágenes se suben al mismo VPS, no a CDN externo.
6. No hay sistema de recuperación de contraseña para el MVP.
7. El webhook de MercadoPago no tiene retry robusto. Si falla, se procesa manualmente en la demo.

---

*Documento generado para hackathon de 7 días. Scope realista, no ambicioso. RFC 2119 aplicado.*
