# Hunar: Technologies, Algorithms and Tools

| | |
|---|---|
| **Project** | Hunar (capstone, 2 semesters) |
| **Based on** | `srs.md` v1.1 and `project_scope.md` |
| **Fixed by the team** | JavaScript, Node.js + Express (backend), React (frontend), PostgreSQL (database) |
| **Status** | Proposal for team and mentor review; feeds the Design Document |

> **How to read this file.** The SRS leaves providers and the stack to the Design Document (open decisions 2, 10 and 13). Everything below beyond Node, Express, React and PostgreSQL is a **recommendation with alternatives**, not a decision. Requirement IDs are in brackets so every choice traces back to the SRS. Phase tags: **S1** = Semester 1, **S2** = Semester 2.

---

## 1. Architecture at a glance

```
 React PWA (mobile-first)
   │  HTTPS / REST (JSON)
   ▼
 Express API (Node.js) ──► PostgreSQL (+ pgvector, pg_trgm, full-text)
   │            │
   │            ├──► Redis ──► BullMQ workers (async AI + verification jobs)
   │            │                 │
   │            │                 ├──► LLM / vision API  (listing, Concierge)
   │            │                 ├──► Speech-to-text + translation APIs
   │            │                 ├──► Verification service (Python, FastAPI)
   │            │                 └──► ffmpeg (audio/video processing)
   │            │
   │            └──► Object storage (S3-compatible)
   │                    ├── public bucket  (compressed product images, audio)
   │                    └── PRIVATE bucket (originals, ID proof, videos)
   │
   └──► External: OTP provider, payment gateway (test mode), map tiles
```

**Why async jobs matter:** AI listing, transcription, translation and verification are slow. Users on weak networks must not wait on one long request. The API accepts the upload, returns a job ID immediately, and the client polls (or uses SSE) for the result `[NFR-PER-01, NFR-PER-03]`.

**Why a small Python service:** Node is fine for the product. Computer-vision and ML evaluation libraries (OpenCV, scikit-learn, CLIP) are far more mature in Python. Keep it to **one** small FastAPI service for verification, so the rest of the team stays in JavaScript. `[FR-VER-13: replaceable detection components]`

---

## 2. Frontend (React)

| Need | Technology | Notes |
|---|---|---|
| Build tool | **Vite** + React 18 | Fast builds, simple PWA plugin |
| Language | **JavaScript (ES2022+, JSX)** | Fixed. Use **JSDoc** comments for editor hints, and **Zod** schemas for runtime safety |
| Routing | **React Router** | Needed for scroll and filter restoration `[FR-DIS-08]` |
| Server state | **TanStack Query** | `useInfiniteQuery` for batches of 12, caching, retries, loading and error states `[FR-DIS-04, NFR-PER-03]` |
| Client state | **Zustand** (or Context) | Cart, UI state, language choice |
| Styling | **Tailwind CSS** | Mobile-first utilities; large touch targets |
| Forms | **React Hook Form + Zod** | Same Zod schemas can validate on the server |
| PWA | **vite-plugin-pwa (Workbox)** | Installable app, caching, offline shell `[2.1 PWA]` |
| Offline drafts | **IndexedDB** via `idb` | Hold unsent listings/recordings on a bad network |
| Infinite scroll | **IntersectionObserver** (`react-intersection-observer`) | Plus a visible hint and a **Load more** button `[FR-DIS-05 to 07]` |
| Voice recording | **MediaRecorder API** | Browser-native; outputs WebM/Opus `[FR-STO-01]` |
| Camera capture | `<input type="file" capture>` + **getUserMedia** | Photos and making-process video `[FR-LST-01, FR-VER-09]` |
| Client image prep | **browser-image-compression** or Canvas | For the *display copy only*. The **original must be uploaded untouched first** `[FR-LST-11]` |
| Image loading | Native `loading="lazy"`, `srcset`, WebP/AVIF, **BlurHash** placeholders | Low-bandwidth friendly `[NFR-PER-02]` |
| Craft map | **react-leaflet** (or MapLibre GL) + India **GeoJSON** | Region-level polygons/markers only `[FR-MAP-01 to 04]` |
| 360° photo spin | Custom component: image sequence + pointer-drag/swipe | No library required; ~100 lines `[FR-VIEW-02]` |
| Internationalisation | **react-i18next** | UI strings in the supported languages `[NFR-USE-05]` |
| Accessibility | **eslint-plugin-jsx-a11y**, semantic HTML, focus management | Keyboard fallback `[NFR-USE-04]` |
| Icons / UI primitives | Lucide, **Radix UI** or Headless UI | Accessible dialogs, menus |

**Algorithms and techniques on the client**
- **Infinite scroll with sentinel element:** an invisible element at the list end triggers the next fetch when it enters the viewport.
- **Scroll and filter restoration:** store filters in the URL query string and cache pages in TanStack Query, so Back returns to the same place `[FR-DIS-08]`.
- **Debounced search input** (about 300 ms) to avoid a request on every keystroke.
- **Optimistic UI and retry with exponential backoff** for flaky networks.
- **Image sequence spin:** map horizontal drag distance to a frame index, `frame = floor(dx / step) mod N`.

---

## 3. Backend (Node.js + Express)

### 3.1 Core
| Need | Technology | Notes |
|---|---|---|
| Runtime / framework | **Node.js LTS + Express** | Fixed |
| Language | **JavaScript (ES modules)** | Fixed. Set `"type": "module"` in `package.json`; use `node --watch` (or nodemon) in development |
| Validation | **Zod** (or Joi) | Validate every request body, query and param. In plain JavaScript this is your main protection against bad data, so do not skip it |
| DB access | **Prisma** (easiest) or **Drizzle** / **Knex** | Prisma does not natively model `pgvector`; use raw SQL for vector queries |
| Migrations | Prisma Migrate / Knex migrations / `node-pg-migrate` | Versioned schema |
| API docs | **OpenAPI / Swagger** (`swagger-ui-express`) | Also documents the pagination contract `[FR-DIS-09]` |
| Logging | **pino** (+ `pino-http`) | Structured logs |
| Config | **dotenv** + a validated config module | Never commit keys |
| Error handling | Central Express error middleware, typed errors | |

### 3.2 Authentication and authorisation `[FR-AUTH, NFR-SEC]`
| Need | Technology |
|---|---|
| Phone OTP for artisans | **MSG91**, **Twilio Verify** or **Firebase Auth (phone)**. India note: SMS DLT registration is required for local providers, so Firebase or Twilio Verify can be faster for a demo |
| Sessions | **JWT access token (short-lived) + refresh token** in `httpOnly`, `Secure`, `SameSite` cookies |
| Admin/verifier passwords | **argon2** (or bcrypt) |
| Role-based access control | Custom Express middleware (`requireRole('admin')`), roles: artisan, buyer, coordinator, admin/verifier |
| Brute-force protection | **express-rate-limit** (+ Redis store), OTP attempt limits and cooldowns |
| Hardening | **helmet**, **cors** (strict origins), CSRF protection if using cookies, HTTPS everywhere |
| Audit trail | `audit_log` table for verification decisions and admin actions `[FR-ADM-05]` |

**Algorithms:** OTP = random 6-digit code, stored **hashed** with a short expiry (about 5 min) and attempt counter; JWT signing (HS256/RS256); rate limiting by token bucket or sliding window.

### 3.3 Files and media `[FR-LST-11, FR-VER-11, NFR-SEC-03]`
| Need | Technology |
|---|---|
| Upload handling | **multer** or **busboy** (streaming); better: **presigned upload URLs** straight to storage |
| Object storage | **AWS S3**, **Cloudflare R2**, or **MinIO** for local dev (all S3-compatible, use `@aws-sdk/client-s3`) |
| Private evidence | **Private bucket** + short-lived **presigned GET URLs**, accessed only through role-checked endpoints |
| Image processing | **sharp** (resize, WebP/AVIF, strip EXIF *from public copies*) |
| EXIF/metadata read | **exifr** or `exiftool-vendored` (read from the **original**) |
| Hashing | Node built-in `crypto.createHash('sha256')`, streamed |
| Audio/video | **ffmpeg** via `fluent-ffmpeg`: transcode audio to 16 kHz mono WAV, extract video keyframes |
| File-type safety | Check **magic bytes** (`file-type` package), not just extension; enforce size limits |
| Optional malware scan | **ClamAV** |
| CDN | Cloudflare / CloudFront in front of the public bucket |

**Upload rule from the SRS, in order:**
1. receive the original → 2. compute SHA-256 → 3. save the original to the **private** bucket → 4. only then create compressed public derivatives.

### 3.4 Background jobs
| Need | Technology |
|---|---|
| Queue | **BullMQ** + **Redis** |
| Jobs | `generate-listing-draft`, `transcribe-voice`, `translate-transcript`, `run-verification`, `compute-embeddings`, `generate-image-derivatives` |
| Behaviour | Retries with backoff, dead-letter handling, job status exposed to the client |

Simpler alternative if Redis feels heavy: **pg-boss** (queue inside PostgreSQL).

### 3.5 Orders and payments `[FR-ORD]`
| Need | Technology |
|---|---|
| Gateway (test mode) | **Razorpay** (India-friendly) or **Stripe** test mode |
| Webhooks | Verify the **HMAC signature** on every callback |
| Order consistency | **PostgreSQL transactions**, an **order state machine** (`created → payment_pending → paid/failed → shipped → delivered / cancelled`) |
| Safe retries | **Idempotency keys** on order and payment endpoints |
| Record the test result | `payments` table storing gateway ID, status, raw payload `[FR-ORD-06]` |

---

## 4. Database (PostgreSQL)

PostgreSQL can carry most of the "search + AI" load with extensions, so you avoid adding Elasticsearch or a separate vector database.

| Extension / feature | Used for |
|---|---|
| **Full-text search** (`tsvector`, `GIN` index) | Keyword search on title, description, tags `[FR-DIS-02]` |
| **pg_trgm** | Typo-tolerant, fuzzy matching (important for transliterated craft names) |
| **pgvector** (HNSW or IVFFlat index) | Semantic search, Concierge retrieval, similar-image and similar-product lookup `[FR-AIB-04]` |
| **JSONB** | Raw verification signals, AI outputs, gateway payloads |
| **PostGIS** (optional) | Only if you need spatial queries. For region-level origin, a plain `regions` table with a centroid is enough `[FR-PRO-09]` |
| **Partial / composite indexes** | Published-only products, category + price filters |
| **Transactions and constraints** | Orders, cart, verification state |

**Suggested core entities** (to finalise in the Design Document): `users`, `roles`, `artisan_profiles`, `artisan_groups`, `regions`, `crafts`, `products`, `product_images`, `product_embeddings`, `tags`, `voice_stories`, `transcripts`, `translations`, `verification_cases`, `verification_evidence`, `verification_signals`, `review_decisions`, `price_references`, `carts`, `orders`, `order_items`, `payments`, `consents`, `audit_log`.

**Pagination `[FR-DIS-09]`:** the SRS asks for page size + offset (`LIMIT 12 OFFSET n`). Add a stable `ORDER BY` (for example `created_at DESC, id DESC`) so pages do not overlap. Note: *keyset* pagination (`WHERE (created_at, id) < (...)`) is faster and more stable at scale, so you could offer it as an optional improvement later.

---

## 5. AI components

All providers are still open decisions `[SRS §10, Q2]`. Wrap each one behind an **interface/adapter** in code (for example `SpeechToText.transcribe()`), so a provider can be swapped without touching business logic. This also supports `[FR-VER-13]`.

### 5.1 AI-assisted listing `[FR-LST]`
| Need | Options |
|---|---|
| Vision-language model | **Google Gemini**, **OpenAI GPT-4o**, **Anthropic Claude**, or an open model via an API host |
| Output | Request **structured JSON output** (schema-constrained) for `title`, `description`, `category`, `tags` |
| Safety | Result is always saved as `status = draft`; publishing needs an explicit approve action `[FR-LST-07 to 10, NFR-REL-01]` |

**Technique:** send 3 to 4 photos plus a fixed prompt and a fixed category list, so the model chooses from your taxonomy. Validate the JSON with Zod; on failure retry once, then fall back to a manual form `[UC-01 alternative flow]`.

### 5.2 Speech and language `[FR-STO]`
| Need | Options |
|---|---|
| Speech-to-text | **Sarvam AI**, **Bhashini (Govt. of India)**, **Google Cloud Speech-to-Text**, **OpenAI Whisper** (self-host or API) |
| Translation | **Bhashini**, **Google Cloud Translation**, **AI4Bharat IndicTrans2** (open-source) |
| Optional text-to-speech | Google TTS / Bhashini, to read translated stories aloud |

Quality varies a lot across Indian languages and dialects. This is why the scope fixes a **supported-language list** and shows a clear message when output is unavailable `[NFR-REL-02]`. Test each shortlisted provider on real recordings in your target languages before choosing.

**Pipeline:** browser audio (WebM/Opus) → ffmpeg → 16 kHz mono WAV → STT → transcript → translate to each supported language → store original audio + transcript + translations against the artisan profile.

### 5.3 Semantic search and Craft Concierge `[FR-AIB]`
This is **RAG (Retrieval-Augmented Generation)** over your own catalogue.

| Step | Technology / algorithm |
|---|---|
| Embed products | A text embedding model (multilingual if you support Indic queries: e.g. **bge-m3**, **multilingual-e5**, or a provider API). Store in `pgvector` |
| Query understanding | LLM extracts filters from natural language: budget, occasion, recipient, style `[FR-AIB-02]` |
| Retrieve | **Hybrid search:** vector similarity + full-text, merged with **Reciprocal Rank Fusion** |
| Generate | LLM writes the recommendation **using only retrieved products** |
| Ground and verify | Server checks every product ID in the answer exists and is published, and attaches the real product link `[FR-AIB-03, FR-AIB-07]` |
| Craft-info assistant | Same RAG pattern over artisan profiles and a **curated craft knowledge base** (source still to decide, Q4) `[FR-AIB-06]` |

**Guardrails:** instruct the model to answer "I don't have that information" when retrieval is empty; use **tool/function calling** so the model can only *query* the catalogue rather than make products up; log conversations for evaluation.

**Frameworks (optional):** you can do all this with plain code and the provider SDK. **LangChain.js** or **LlamaIndex.TS** are available but add complexity; for a student project, plain code is easier to debug and explain.

---

## 6. Verification pipeline `[FR-VER, NFR-REL-03 to 05]`

Software cannot prove something is handmade. The pipeline produces **risk signals**; a **human decides**. Nothing is auto-rejected `[FR-VER-10, 16]`.

| Signal | Technique | Tool |
|---|---|---|
| Exact duplicate | **SHA-256** of the original, compared with stored hashes `[FR-VER-12]` | Node `crypto` |
| Near-duplicate / reused image | **Perceptual hashing** (pHash, dHash) compared by **Hamming distance** | `imagehash` (Python), or implement dHash with `sharp` |
| Semantic similarity across a seller's photos and across sellers | **CLIP image embeddings** + **cosine similarity** | `open_clip` / `transformers` (Python), stored in pgvector |
| AI-generated image detection | Third-party detector API, or an open-source classifier | e.g. Hive, Sightengine, or a Hugging Face model; keep behind an interface |
| Metadata check | Read EXIF: camera make/model, timestamps, GPS present/absent, editing software tags | `exifr`, `exiftool` |
| Provenance / watermark | **C2PA Content Credentials** manifest check; provider watermark signals where available | `c2pa-node` / `c2patool` |
| Screen recapture | Moiré detection (frequency analysis with FFT), screen-bezel and glare cues | OpenCV + NumPy |
| Editing/compression artefacts | **Error Level Analysis (ELA)** (weak signal, use cautiously) | Pillow / OpenCV |
| Photo-to-video consistency | Extract video **keyframes**, then compare with product photos using **ORB/SIFT feature matching** (OpenCV) and/or CLIP similarity | ffmpeg + OpenCV |
| Coordinator vouching, documents | Stored as evidence and shown to the reviewer; no algorithm needed | UI + private storage |

### 6.1 Combining signals into a risk assessment `[FR-VER-15]`
Start simple and explainable:

```
risk = w1·ai_detector + w2·duplicate + w3·metadata_anomaly
     + w4·recapture + w5·video_mismatch + w6·similarity_anomaly      (each signal scaled 0..1)

risk < T_low            → low    → normal flow
T_low ≤ risk < T_high   → medium → human review queue
risk ≥ T_high           → high   → priority human review     (never auto-reject)
```

- **Version 1:** hand-set weights and rules (transparent, easy to defend).
- **Version 2 (if you have enough labelled data):** **logistic regression** or a small **gradient-boosted tree** (`scikit-learn`) trained on your test set, still only used to *prioritise* review.
- Thresholds and weights are open decisions `[SRS §10, Q14]`; tune them on the evaluation set.

### 6.2 Evaluation (a required deliverable) `[NFR-REL-05]`
Build a labelled test set of **real photos, AI-generated images, screen recaptures and duplicates**, then report:

- **Confusion matrix**, **Precision = TP / (TP + FP)**, **Recall = TP / (TP + FN)**, **F1**
- **ROC curve / AUC** and **Precision–Recall curve** for threshold selection
- Per-class breakdown (which attack type slips through)

Tools: `scikit-learn`, `pandas`, `matplotlib`, Jupyter notebook. Keep the dataset small but honest, and describe its limits in the report.

---

## 7. Fair-price recommendation `[FR-PRI]`

Reference data is the bigger problem than the algorithm `[SRS §10, Q3]`. Candidate data sources: your own catalogue (grows over time), government craft/handicraft price lists, comparable listings from public marketplaces (check terms of use), and expert or cooperative input.

| Method | Idea | Use |
|---|---|---|
| **Percentile band** | Find comparable products (same category/material/size) and use the **25th to 75th percentile** (or median ± IQR) as the fair range | Main method for V1; robust to outliers |
| **Similar-product retrieval** | Find comparables by **embedding similarity + category filter** (kNN) | Choosing the comparable set |
| **Cost-plus formula** | `materials + (labour hours × fair hourly wage) + overhead + margin` | Fallback when comparables are few; also educates the artisan |
| **Regression (optional)** | Predict price from category, material, size, craft, region | Only if you collect enough data |

**Warning rule:** `if entered_price < k × lower_bound (for example k = 0.7) → show underpricing warning`, then let the artisan **continue anyway** `[FR-PRI-03 to 05]`. Never block publishing. Handle the **cold-start** case (no comparables) by showing the cost-plus guide or "not enough data" instead of a number.

---

## 8. Search and discovery `[FR-DIS, FR-AIB-04]`

| Piece | Technique |
|---|---|
| Keyword search | PostgreSQL `tsvector` with `ts_rank` |
| Fuzzy matching | `pg_trgm` similarity |
| Semantic search | pgvector **cosine distance** on product embeddings |
| Hybrid ranking | **Reciprocal Rank Fusion:** `score(d) = Σ 1 / (k + rank_i(d))`, with `k ≈ 60` |
| Filters | Category, price range, craft, region, verification status, etc. (final list TBD) |
| Featured artisans | Simple rules first (recent, complete profile, verified), not a recommender system |

Hybrid ranking beats either method alone because keyword search is precise and semantic search handles intent ("a gift for my mother who likes blue").

---

## 9. Security and privacy checklist `[NFR-SEC, NFR-PRI]`

- HTTPS only, HSTS, `helmet` headers
- Role-based access on every protected route, tested explicitly
- Private bucket for originals, ID proof and videos; access via short-lived signed URLs; log who viewed what
- **Never return exact GPS** in any API response, only region/village names. Strip GPS from EXIF on public images `[FR-PRO-09, FR-MAP-04, NFR-PRI-01]`
- A `consents` table: what was consented to (voice, image, video), when, and by whom (artisan or coordinator) `[NFR-PRI-02]`
- Input validation (Zod), parameterised queries (ORM) against SQL injection, output escaping against XSS
- Upload checks: magic bytes, size limits, optional ClamAV
- Rate limits on OTP, login, AI endpoints (these cost money)
- Secrets in environment variables or a secrets manager, never in Git
- Data retention and deletion rules still to be defined `[NFR-PRI-05]`. Be careful with Aadhaar-related data; consider **not collecting** it at all `[SRS §10, Q6]`

---

## 10. Performance on slow networks `[NFR-PER]`

- Paginated API (12 items), compressed responses (`compression` middleware), HTTP caching headers
- Responsive images (`srcset`), WebP/AVIF, lazy loading, BlurHash placeholders, CDN
- Service worker caches the app shell and recently viewed products
- Async jobs with a visible progress state and retry button
- Chunked or resumable uploads for large videos (for example **tus** protocol, or multipart upload to S3)
- Test with Chrome DevTools "Slow 3G" throttling and **Lighthouse**, and record the results as evidence for the success criterion

---

## 11. Testing, quality and DevOps

| Area | Tools |
|---|---|
| Unit / integration (backend) | **Vitest** or **Jest** + **Supertest** |
| Component tests (frontend) | **React Testing Library** |
| End-to-end | **Playwright** (artisan lists → buyer orders in test mode → result recorded) |
| API testing | Postman / Bruno, OpenAPI contract |
| Load / network | **k6**, Lighthouse, DevTools throttling |
| Accessibility | axe-core / Lighthouse a11y, manual keyboard pass |
| AI evaluation | Python notebooks with `scikit-learn`; a fixed prompt set for Concierge (check no invented products) |
| Code quality | **ESLint** (with `eslint-plugin-react-hooks`, `jsx-a11y`), **Prettier**, **Husky** + lint-staged; optional `// @ts-check` + JSDoc for type checking without TypeScript |
| Version control / CI | **Git + GitHub**, **GitHub Actions** (lint, test, build) |
| Containers | **Docker** + **docker-compose** (Postgres, Redis, MinIO, API, verification service) |
| Hosting (demo) | Render, Railway, Fly.io, or AWS; Vercel/Netlify for the frontend; managed Postgres (Neon, Supabase, RDS) with pgvector |
| Monitoring | **Sentry** (errors), pino logs |
| Design and docs | **Figma** (UI), dbdiagram.io / draw.io (ERD, architecture), Markdown docs |

---

## 12. Algorithm and concept glossary (things to learn)

| Concept | One-line explanation | Used in |
|---|---|---|
| **SHA-256** | Fixed-size fingerprint; any change in the file changes the hash | Original image integrity, exact duplicates |
| **Perceptual hash (pHash/dHash)** | Fingerprint that stays similar for visually similar images | Near-duplicate detection |
| **Hamming distance** | Number of differing bits between two hashes | Comparing perceptual hashes (small = similar) |
| **Embeddings** | Vectors that capture meaning of text or images | Semantic search, Concierge, similarity |
| **Cosine similarity** | Angle-based closeness of two vectors | Ranking by meaning |
| **HNSW / IVFFlat** | Approximate nearest-neighbour indexes | Fast pgvector search |
| **RAG** | Retrieve real data first, then let the LLM answer from it | Concierge, craft-info assistant |
| **Hybrid search + RRF** | Merge keyword and vector rankings | Product search |
| **Function / tool calling, structured output** | Constrain LLM actions and format | Listing draft, Concierge |
| **TF-IDF / BM25 / `ts_rank`** | Keyword relevance scoring | Full-text search |
| **Trigram similarity** | Fuzzy string matching | Typo-tolerant search |
| **ORB / SIFT feature matching** | Detect and match image keypoints | Photo-to-video consistency |
| **FFT / moiré analysis** | Frequency-domain patterns from re-photographed screens | Screen recapture detection |
| **ELA** | Highlights compression differences in an edited image | Weak tampering hint |
| **C2PA** | Signed metadata recording how media was created or edited | Provenance checks |
| **Logistic regression / gradient boosting** | Learned classifiers | Risk scoring V2 |
| **Precision, recall, F1, ROC/AUC** | Classifier quality metrics | Verification evaluation |
| **Percentiles / IQR / median** | Robust statistics | Fair-price range |
| **kNN** | Use nearest neighbours as a reference set | Comparable products |
| **JWT, OAuth-style refresh tokens** | Stateless session tokens | Authentication |
| **HMAC** | Keyed hash to prove a message is authentic | Payment webhook verification |
| **Token bucket / sliding window** | Rate-limiting algorithms | OTP and AI endpoint protection |
| **State machine** | Defined states and allowed transitions | Orders, verification cases, listing status |
| **Idempotency** | Repeating a request has the same effect as doing it once | Payments, retries |
| **Exponential backoff** | Increase wait time between retries | Flaky network, provider errors |
| **Offset vs keyset pagination** | Two ways to page through a list | Product list API |
| **IntersectionObserver** | Browser API to detect element visibility | Pull-up / infinite loading |
| **Service worker caching** | Browser-side cache and offline logic | PWA |

---

## 13. Suggested build order by phase

**Semester 1: thin end-to-end slice**
Node + Express (JavaScript, ES modules) setup, PostgreSQL schema and migrations, OTP login + JWT + RBAC, storage with private/public buckets, image upload with SHA-256 + private original, AI listing draft with approval flow (LLM structured output), product list with full-text search and offset pagination, React PWA shell with infinite loading + Load more + scroll restoration, product page, artisan storefront, cart, Razorpay/Stripe test checkout and order records, basic admin review screen, Docker + CI.

**Semester 2: depth features**
Voice recording → STT → translation, Python verification service with the signals in section 6, risk score and reviewer queue, evaluation report (precision/recall), 360° spin, craft map, fair-price engine, pgvector embeddings + semantic search, Craft Concierge (RAG) and craft-info assistant (**cut first if time runs short**), performance and accessibility testing, polish and demo data.

---

## 14. Decisions this document still needs from the team

| # | Decision | Impacts |
|---|---|---|
| 1 | Supported languages | STT and translation provider, i18n, embeddings model |
| 2 | LLM / vision provider and budget | Listing, Concierge, cost controls |
| 3 | STT + translation provider (test on real recordings) | Maker's story quality |
| 4 | AI-image detector (API vs open-source) | Verification accuracy and cost |
| 5 | OTP provider (DLT registration lead time) | Login for artisans |
| 6 | Payment gateway (Razorpay vs Stripe test) | Orders |
| 7 | Hosting + object storage + managed Postgres with pgvector | Deployment |
| 8 | Fair-price reference data source | Pricing feature validity |
| 9 | Craft knowledge base source | Craft-info assistant |
| 10 | ORM choice (Prisma vs Drizzle/Knex) | Backend structure, vector queries |
| 11 | Redis + BullMQ vs pg-boss | Job infrastructure |
| 12 | India map data source and boundary depiction | Craft map (use an official or approved boundary dataset) |

**Cost note:** LLM, STT, translation, detector and SMS calls are paid. Add per-user rate limits, cache results (for example translations and embeddings), and use free tiers or open-source models for development.
