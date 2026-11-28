# HUNAR — REST API Specification (`api.md`)

| | |
|---|---|
| **Project** | Hunar (Capstone Project, 2 Semesters) |
| **Phase Covered** | **Semester 1** — Thin End-to-End Slice |
| **Aligned With** | `srs.md` (v1.1), `project_scope.md`, `design.md`, `database.md` |
| **Base URL** | `/api/v1` (Local: `http://localhost:5000/api/v1`) |
| **Format** | JSON (`Content-Type: application/json`), `multipart/form-data` for media uploads |
| **Validation** | Zod schemas on every body, query, and URL param |

---

## 1. Global API Conventions

### 1.1 Authentication & RBAC (`[FR-AUTH, NFR-SEC-01, 02]`)
* **Access Token:** Short-lived JWT (15 min) sent via `Authorization: Bearer <token>`.
* **Refresh Token:** Stored in an `httpOnly`, `Secure`, `SameSite=Strict` cookie (`/api/v1/auth/refresh`).
* **JWT Payload (Multi-Role Support):**
  ```json
  {
    "sub": "uuid-of-user",
    "roles": ["artisan", "buyer"],
    "iat": 1764350000,
    "exp": 1764350900
  }
  ```
* **Route Guards:** Express middleware `requireAuth` and `requireRole('artisan' | 'buyer' | 'coordinator' | 'admin' | 'verifier')`.

### 1.2 Standard Response Envelopes
**Success:**
```json
{
  "success": true,
  "data": { ... },
  "meta": { }
}
```

**Error:**
```json
{
  "success": false,
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Price must be greater than 0",
    "details": [{ "field": "price_inr", "issue": "too_small" }]
  }
}
```
*Common HTTP Status Codes:* `200 OK`, `201 Created`, `202 Accepted` (async jobs), `400 Bad Request`, `401 Unauthorized`, `403 Forbidden`, `404 Not Found`, `409 Conflict`, `429 Too Many Requests`, `500 Internal Server Error`.

### 1.3 Pagination Contract (`[FR-DIS-04, FR-DIS-09]`)
All list endpoints use **offset pagination** with a default batch size of **12** and a stable sort order (`created_at DESC, id DESC`).
* **Query Params:** `?limit=12&offset=0` (max `limit` capped at `24`).
* **Response `meta`:**
  ```json
  "meta": {
    "limit": 12,
    "offset": 0,
    "returned": 12,
    "has_more": true,
    "next_offset": 12
  }
  ```

### 1.4 Privacy & Safety Rules Enforced at the API Layer
1. **No Exact GPS (`[FR-PRO-09, FR-MAP-04, NFR-PRI-01]`):** Never serialize exact coordinates for any artisan. Return only `region_name`, `district`, `state`, and `village_or_town`.
2. **Draft-Before-Publish (`[FR-LST-07 to 10]`):** AI-generated listings are always created with `status: "draft"` and cannot be published until an explicit `POST /products/:id/approve` call.
3. **Private Originals (`[FR-LST-11, FR-VER-11]`):** Public API responses never return `original_storage_key`. Private verification evidence is served only via short-lived (5-minute) presigned URLs to `admin`/`verifier` roles and logged in `audit_log`.

---

## 2. Mock External APIs First (Adapter Pattern)

External services (OTP SMS, Vision LLM, S3 Storage, Payment Gateway) cost money, require DLT/KYC approvals, or slow down local dev. **Every external integration MUST sit behind a thin adapter interface** controlled by environment variables.

### 2.1 Environment Toggle (`.env.example`)
```env
# Set to "mock" during local dev & CI; switch per provider when ready
OTP_PROVIDER=mock              # mock | firebase | twilio | msg91
VISION_LLM_PROVIDER=mock       # mock | gemini | openai | claude
STORAGE_PROVIDER=minio         # mock_local | minio | r2 | s3
PAYMENT_GATEWAY=mock           # mock | razorpay_test | stripe_test
```

### 2.2 Mock Behaviour by Service

| Service | Adapter Method | Mock Mode Behaviour (Local Dev) | Real Provider (Demo / Prod) |
|---|---|---|---|
| **OTP SMS** | `otpProvider.send(phone, code)` | Does not send SMS. Logs `[MOCK OTP] +919876543210 -> 123456` in terminal and accepts `123456` in dev. | Firebase Auth / Twilio Verify / MSG91 |
| **Vision LLM** | `listingAI.generateDraft(images, craftList)` | Waits 800ms (simulating network) and returns a deterministic, Zod-valid JSON draft (`title`, `description`, `craft_id`, `tags`). Supports `?mock_fail=true` to test manual fallback `[UC-01]`. | Gemini / GPT-4o / Claude (structured JSON output) |
| **Object Storage** | `storage.uploadPrivate(key, buf)` `storage.uploadPublic(key, buf)` | Uses local **MinIO** in Docker (S3-compatible) or saves to `./.local_storage/{private,public}` if Docker isn't running. | Cloudflare R2 / AWS S3 |
| **Payment Gateway** | `payments.createOrder(order)` `payments.verifyWebhook(req)` | Returns a fake `gateway_order_id` (`mock_ord_xxx`). Exposes a dev-only route `POST /payments/mock-complete` to simulate `paid` or `failed` webhooks. | Razorpay / Stripe (**Test Mode Only**) |

### 2.3 Minimal Adapter Structure (Node.js ESM)
```
src/
  adapters/
    otp/
      index.js          # Reads process.env.OTP_PROVIDER -> exports active adapter
      mockOtp.js        # Always logs OTP to console
      twilioOtp.js      # Real implementation plugged in later
    llm/
      index.js
      mockListingAI.js  # Returns valid draft JSON after 800ms
      geminiListingAI.js
    payments/
      index.js
      mockGateway.js
      razorpayTest.js
```
> **Rule for the team:** Business logic in `controllers/` and `services/` must **never** import an external SDK directly—only the adapter from `src/adapters/*`.

---

## 3. Semester 1 Endpoints

### 3.1 Authentication & Sessions (`[FR-AUTH]`)

| Method | Endpoint | Access | Description |
|---|---|---|---|
| `POST` | `/auth/otp/request` | Public (Rate-limited) | Sends a 6-digit OTP to an artisan/buyer phone number |
| `POST` | `/auth/otp/verify` | Public (Rate-limited) | Verifies OTP, creates user if new, issues JWT + refresh cookie |
| `POST` | `/auth/login` | Public (Rate-limited) | Email/phone + Argon2 password login (buyers, coordinators, admins) |
| `POST` | `/auth/refresh` | Cookie | Rotates and returns a fresh short-lived access token |
| `POST` | `/auth/logout` | Auth | Clears refresh cookie |
| `GET` | `/auth/me` | Auth | Returns current user, assigned `roles[]`, and `artisan_profile` summary (if any) |

**Example: `POST /auth/otp/verify`**
```json
// Request
{
  "phone": "+919876543210",
  "otp": "123456",
  "full_name": "Meera Devi",
  "requested_role": "artisan"
}

// Response (200 OK)
{
  "success": true,
  "data": {
    "access_token": "eyJhbGciOiJIUzI1NiIs...",
    "user": {
      "id": "b1a7...",
      "full_name": "Meera Devi",
      "phone": "+919876543210",
      "roles": ["artisan"]
    }
  }
}
```

---

### 3.2 Artisan Storefronts (`[FR-PRO, FR-MKT-03]`)

| Method | Endpoint | Access | Description |
|---|---|---|---|
| `POST` | `/artisans/profile` | `artisan`, `coordinator` | Create storefront profile (auto-generates shareable `slug`; starts `unverified`) |
| `PATCH` | `/artisans/profile/:id` | Owner or `coordinator` | Update name, bio, village/region, primary craft, optional profile photo |
| `GET` | `/artisans/featured` | Public | Returns featured artisans for the home page `[FR-MKT-03]` |
| `GET` | `/artisans/:slug` | Public | Public storefront page (NO exact GPS; includes `verification_status` and shareable link) |
| `GET` | `/artisans/:slug/products` | Public | Published products for this artisan (`?limit=12&offset=0`) |

---

### 3.3 Media Upload & AI-Assisted Listing (`[FR-LST, FR-VER-11, 12]`)

**Strict Upload Order (`design.md §3.3`):**
1. Validate magic bytes (`file-type`) & size limit.
2. Compute `sha256_hash` on untouched original bytes.
3. Save original to **private** bucket (`original_storage_key`).
4. Generate compressed WebP/AVIF + thumbnail + BlurHash (with EXIF & GPS stripped) → save to **public** bucket.

| Method | Endpoint | Access | Description |
|---|---|---|---|
| `POST` | `/products/drafts` | `artisan`, `coordinator` | Upload 1–4 product photos (`multipart/form-data`) + optional `use_ai=true`. Returns `product_id` (status: `draft`) and `job_id` if AI is queued (`202 Accepted`) |
| `GET` | `/jobs/:jobId` | Auth (Owner) | Poll async AI draft job status (`queued` \| `processing` \| `completed` \| `failed`) |
| `PATCH` | `/products/:id` | Owner, `coordinator` | Edit draft/product fields (`title`, `description`, `craft_id`, `tags`, `price_inr`, `stock_quantity`) |
| `POST` | `/products/:id/approve` | Owner, `coordinator` | Explicit human approval `[FR-LST-09]`. Moves `draft → published` (allowed even if artisan is `unverified` `[FR-VER-01]`) |
| `DELETE` | `/products/:id` | Owner, `admin` | Archives (`status = 'archived'`) a product |

**Example: `POST /products/drafts` → `202 Accepted`**
```json
{
  "success": true,
  "data": {
    "product_id": "9f2c...",
    "status": "draft",
    "job_id": "job_ai_8812",
    "images": [
      {
        "id": "img_1",
        "public_url": "https://cdn.hunar.local/pub/img_1.webp",
        "thumbnail_url": "https://cdn.hunar.local/pub/img_1_thumb.webp",
        "blurhash": "LEHV6nWB2yk8pyo0adR*.7kCMdnj"
      }
    ]
  }
}
```

**Example: `GET /jobs/job_ai_8812` → `200 OK`**
```json
{
  "success": true,
  "data": {
    "job_id": "job_ai_8812",
    "status": "completed",
    "draft_suggestion": {
      "title": "Hand-Painted Madhubani Terracotta Vase",
      "description": "Traditional Mithila motif painted with natural pigments on baked clay.",
      "craft_id": "craft_madhubani_uuid",
      "tags": ["madhubani", "terracotta", "home-decor", "handmade"]
    }
  }
}
```
> If `status === "failed"`, the frontend shows a clear message and lets the artisan fill in the form manually `[UC-01 alternative flow, NFR-REL-01]`.

---

### 3.4 Product Discovery, Search & Reference Data (`[FR-DIS]`)

| Method | Endpoint | Access | Description |
|---|---|---|---|
| `GET` | `/products` | Public | Browse/search published products in batches of 12 (`?limit=12&offset=0&q=&craft=&region=&min_price=&max_price=&verified_only=false`) |
| `GET` | `/products/:slug` | Public | Full product detail page + artisan summary + storefront link `[FR-DIS-10]` |
| `GET` | `/crafts` | Public | List active craft categories for filters and listing dropdowns |
| `GET` | `/regions` | Public | List regions (state, district, region_name) for filters and onboarding |

**Example: `GET /products?limit=12&offset=0&q=terracotta`**
```json
{
  "success": true,
  "data": [
    {
      "id": "9f2c...",
      "title": "Hand-Painted Madhubani Terracotta Vase",
      "slug": "hand-painted-madhubani-terracotta-vase-9f2c",
      "price_inr": 850.00,
      "thumbnail_url": "https://cdn.hunar.local/pub/img_1_thumb.webp",
      "blurhash": "LEHV6nWB2yk8pyo0adR*.7kCMdnj",
      "artisan": {
        "display_name": "Meera Devi",
        "slug": "meera-devi-madhubani",
        "verification_status": "unverified",
        "region_name": "Madhubani",
        "state": "Bihar"
      }
    }
  ],
  "meta": {
    "limit": 12,
    "offset": 0,
    "returned": 1,
    "has_more": false,
    "next_offset": null
  }
}
```

---

### 3.5 Cart, Orders & Test-Mode Payments (`[FR-ORD]`)

| Method | Endpoint | Access | Description |
|---|---|---|---|
| `GET` | `/cart` | `buyer` (any Auth) | Fetch current user's cart with live prices and stock |
| `POST` | `/cart/items` | Auth | Add `{ product_id, quantity }` to cart |
| `PATCH` | `/cart/items/:itemId` | Auth | Update item quantity |
| `DELETE` | `/cart/items/:itemId` | Auth | Remove item from cart |
| `POST` | `/orders` | Auth | Create order from cart inside a PostgreSQL transaction. Requires `Idempotency-Key` header |
| `GET` | `/orders` | Auth | List buyer's orders (or artisan's incoming order items) |
| `GET` | `/orders/:id` | Owner / `admin` | Order details, state machine status, and recorded test payment result |
| `POST` | `/payments/initiate` | Auth | Creates test-mode payment intent (`is_test_mode: true`) via active payment adapter |
| `POST` | `/payments/webhook` | Public (HMAC-verified) | Receives Razorpay/Stripe test webhook, verifies HMAC signature, records `raw_payload`, updates order to `paid` or `failed` |
| `POST` | `/payments/mock-complete` | Dev Only (`PAYMENT_GATEWAY=mock`) | Simulates a `paid` or `failed` callback locally without external gateway keys |

---

### 3.6 Consents & Basic Admin Review (`[FR-ADM, NFR-PRI-02]`)

| Method | Endpoint | Access | Description |
|---|---|---|---|
| `POST` | `/consents` | Auth | Record explicit consent (`voice_recording`, `product_images`, `making_video`, `verification_evidence`) |
| `POST` | `/verification/cases` | `artisan`, `coordinator` | Submit verification evidence (ID/craft proof or coordinator vouching) to private bucket |
| `GET` | `/admin/verification-cases` | `admin`, `verifier` | List `open` / `in_review` cases sorted by priority (`?limit=12&offset=0`) |
| `GET` | `/admin/evidence/:evidenceId/url` | `admin`, `verifier` | Returns a 5-min presigned GET URL for private evidence and writes an `audit_log` entry `[NFR-SEC-03]` |
| `POST` | `/admin/verification-cases/:id/decision` | `admin`, `verifier` | Record human decision (`approved` \| `rejected` \| `needs_more_info` + `rationale`). Updates `artisan_profiles.verification_status` and writes `audit_log` |

---

## 4. Deferred to Semester 2 (Do Not Build in S1)

To protect the Semester 1 thin-slice deadline (`project_scope.md §8`), these endpoints are intentionally left for Semester 2:
* `POST /artisans/:id/voice-story` & STT/Translation jobs (`[FR-STO]`)
* Internal Python FastAPI verification bridge (`/internal/verify-image`, `/internal/verify-video`) (`[FR-VER-13 to 15]`)
* `GET /products/:id/fair-price` recommendation endpoint (`[FR-PRI]`)
* `GET /map/regions` GeoJSON aggregation endpoint (`[FR-MAP]`)
* `POST /concierge/chat` (RAG) & `GET /products/semantic-search` (`[FR-AIB]`)

---

## 5. Team Parallel-Work Workflow

1. **Day 1:** Set up Express + Zod + `mock` adapters (`OTP_PROVIDER=mock`, `VISION_LLM_PROVIDER=mock`, `PAYMENT_GATEWAY=mock`).
2. **Backend Team:** Implement endpoints in order: `Auth` → `Storefronts` → `Media + AI Draft` → `Discovery (limit=12)` → `Cart/Orders` → `Basic Admin`.
3. **Frontend Team:** Build the React PWA against the JSON contracts in Section 3 using TanStack Query (`useInfiniteQuery` for `offset` pagination) without waiting for real SMS, LLM, or Razorpay accounts.
4. **End of S1:** Swap `.env` flags one by one from `mock` to real test-mode providers (e.g., Gemini + Razorpay Test + Firebase Phone Auth) and run the Playwright end-to-end test.