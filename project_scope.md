# Hunar: Project Scope

| | |
|---|---|
| **Project** | Hunar (capstone project, 2 semesters) |
| **Project type** | Application + Product |
| **Source** | SIH 2026 problem statement SIH26090 (MoSJE, Heritage & Culture) |
| **Aligned with** | `srs.md`, SRS v1.1 (19 November 2026) |
| **Status** | Working draft for team and mentor review |

> This file summarises **what is in and out of the project**. The SRS holds the detailed requirements (requirement IDs are given in brackets), and the Design Document will hold the implementation details.

---

## 1. Summary

Hunar is a **mobile-first marketplace** where marginalised artisans showcase and sell handmade products all year round, instead of depending on occasional melas and exhibitions. Each artisan has a storefront profile inside the shared marketplace. AI helps artisans create listings from photos and speak for themselves in their own language, helps buyers discover products, and gives fair-price guidance.

## 2. Problem

- Artisans have few places to show their work beyond occasional exhibitions and local markets.
- A separate online store does not solve this if they have no online customers.
- Language barriers, limited digital literacy, unreliable connectivity and difficulty presenting products online restrict access to wider markets.
- Buyers cannot easily discover artisans or judge whether a listing is trustworthy.

## 3. What Hunar is and is not

- **It is** one shared marketplace with a storefront profile per artisan, delivered as a mobile-first web app with PWA support.
- **It is not** a Dukaan or Shopify-style store builder.
- **It is not** a CRM in the first release.
- **It does not claim** that software can prove a product is handmade. The Verified badge means the artisan passed Hunar's review process.

## 4. Goals

1. Give artisans an ongoing digital marketplace presence.
2. Help artisans create listings with less manual writing.
3. Reduce language barriers through voice transcription and translation.
4. Help buyers understand the artisan and the product's regional context.
5. Offer product presentation and discovery features suited to handmade goods.
6. Give fair-price guidance that flags possible underpricing without blocking the artisan's choice.

## 5. Users

| User | Role |
|---|---|
| Artisan | Registers, builds a storefront profile, lists products, records a voice introduction |
| Buyer | Browses, searches, views artisans and products, orders |
| Coordinator | SHG or cooperative facilitator who helps artisans onboard and records media on their behalf |
| Admin / Verifier | Reviews verification evidence and flagged listings, administers the platform |

## 6. In scope (release 1.0)

### Marketplace and storefront
- [ ] Home page with a short intro, product discovery, featured artisans and the craft map; separate About page; optional skippable intro `[FR-MKT]`
- [ ] Artisan storefront profile: name, photo, craft, village/region, voice introduction with translated transcript, published products, verification status, shareable link `[FR-PRO]`
- [ ] Origin shown at village/region level only, never exact GPS `[FR-PRO-09, FR-MAP-04]`

### Accounts
- [ ] Phone-number OTP login for artisans, buyer accounts, authorised admin/verifier access, role-restricted functions `[FR-AUTH]`

### AI-assisted listing
- [ ] 3 to 4 photos produce a draft title, description, category and tags `[FR-LST]`
- [ ] Listing stays a draft until the artisan or an authorised user approves it; never auto-published `[FR-LST-07 to 10]`
- [ ] Original photo retained privately, with a SHA-256 hash, before any compression `[FR-LST-11, FR-VER-11, FR-VER-12]`

### Maker's story and language
- [ ] Voice recording, speech-to-text, translation into supported languages, shown on the storefront profile `[FR-STO]`
- [ ] Fixed list of supported languages (list to be decided)

### Verification and trust
- [ ] Any artisan can publish as **Unverified**; the Verified badge needs evidence plus human review `[FR-VER-01 to 07]`
- [ ] Evidence can include ID or craft proof, a making-process video (no face required), or coordinator vouching; a coordinator can record the video `[FR-VER-04, 08, 09]`
- [ ] Automated checks run through replaceable detection components: image analysis, metadata, provenance/watermark signals, similarity across photos, duplicates, and photo-to-video consistency `[FR-VER-13, 14]`
- [ ] Signals combine into a risk assessment; flagged items go to a human reviewer; nothing is rejected automatically `[FR-VER-10, 15, 16]`

### Discovery
- [ ] Search and filters, with browsing in batches of about 12 `[FR-DIS-01 to 04]`
- [ ] Pull-up loading with a visible hint, a Load more fallback for desktop and keyboard users, and preserved position and filters on return `[FR-DIS-05 to 08]`
- [ ] Pagination by page size and offset in the API `[FR-DIS-09]`
- [ ] Open the artisan storefront from any product `[FR-DIS-10]`

### AI buyer assistance
- [ ] **Craft Concierge:** natural-language product discovery by budget, occasion, recipient and style, with recommendations drawn only from the Hunar catalogue `[FR-AIB-01 to 03]`
- [ ] Semantic search in addition to keyword search `[FR-AIB-04]`
- [ ] Product comparison and gift discovery `[FR-AIB-05]`
- [ ] Craft-information assistant grounded in artisan profiles and trusted craft information; it must not invent listings or facts `[FR-AIB-06, 07]`

### Product viewing and map
- [ ] Product photos and a photo-based 360-degree spin `[FR-VIEW-01, 02]`
- [ ] Interactive craft-region map of India at region level `[FR-MAP-01 to 03]`

### Pricing
- [ ] Suggested fair-price range, underpricing warning, artisan can continue with their price `[FR-PRI]`

### Orders
- [ ] Cart, order creation and tracking, payment gateway in **test mode** with the test result recorded `[FR-ORD]`

### Administration
- [ ] Admin and verifier access to evidence, review decisions, flagged-listing review, review status and outcome `[FR-ADM]`

## 7. Out of scope or deferred

**Out of scope:**
- Independent store-builder functionality like Dukaan or Shopify
- CRM functionality
- NFTs or blockchain-based authenticity claims
- Real-money settlement and artisan payouts
- Full 3D product reconstruction (photo spin is the baseline)
- Any claim that software can conclusively prove a product is handmade
- A native mobile app (PWA only)

**Deferred from the first release** (removed in SRS v1.1 to protect the timeline):
- Buyer–artisan messaging
- Ratings and reviews
- Notifications
- Optional email and OAuth login paths

## 8. Proposed phasing (to confirm in the project plan)

| Phase | Focus |
|---|---|
| **Semester 1** | Thin end-to-end slice: OTP login and roles, storefront profile, AI-assisted listing with approval, private original-image retention, browse, search and pagination, product page, cart, test-mode checkout and orders, basic admin review |
| **Semester 2** | Voice introduction with transcription and translation, verification pipeline and review queue, photo spin, craft map, fair-price guidance, Craft Concierge and semantic search, evaluation, testing and polish |
| **Later** | CRM, messaging, ratings and reviews, notifications, native app, payouts, more languages, 3D reconstruction |

The SRS has no Must/Should/Could priorities, so this table is the project's priority order. Craft Concierge is an addition beyond the five proposal outcomes, so it sits last in Semester 2 and is the first thing to cut if time runs short.

## 9. Constraints and assumptions

- Two semesters and a small student team.
- Mobile-first, simple for users with limited digital literacy, usable on slow and intermittent networks.
- Artisans keep control of AI-generated content before publication.
- AI, speech-to-text, translation, detection and OTP rely on external providers, still to be selected.
- Payments in test mode only.
- Verification is human-assisted, and the demo needs reviewers (verifiers or team members).
- Fair-price recommendations need reference data that has not yet been identified.
- Artisans or coordinators supply product photos and information. If real artisans are not available, sample data is used.

## 10. Deliverables

- Working prototype (mobile-first web app with PWA support)
- `srs.md` and the Design Document, kept in sync with the build
- Verification evaluation report with precision and recall on a test set
- Final project report and demo

## 11. Success criteria

| Area | Measure |
|---|---|
| End-to-end flow | An artisan lists a product, a buyer finds it, orders it in test mode, and the result is recorded |
| Trust rules | An artisan can publish as Unverified; AI content stays a draft until approved; no exact GPS shown; nothing auto-rejected on an AI flag |
| Verification quality | Precision and recall reported on a test set of real photos, AI-generated images, screen recaptures and duplicates |
| Fair price | Warns about underpricing without blocking the artisan's price |
| Browsing | Batches of about 12 with pull-up loading, a Load more fallback, and preserved position and filters |
| Network | Usable on a slow, throttled connection |
| AI assistance | Concierge recommendations come only from catalogue products and link to product pages |

Numeric targets, thresholds and scoring weights are still to be agreed with the mentor.

## 12. Main risks

| Risk | Plan |
|---|---|
| Too much scope for two semesters | Follow the phasing above, cut Craft Concierge first |
| AI detection is unreliable | Treat it as a risk indicator only, with human review as the final gate |
| Artisans lacking documents are shut out | Unverified publishing, evidence of several kinds, coordinator support |
| Weak translation or transcription for some languages | Fixed language list, clear message when AI output is unavailable |
| Photo compression harms verification | Keep the original privately and hash it before any compression |
| Privacy of a vulnerable group | Consent before collecting voice, images and video; private storage for evidence; region-level location |
| Concierge gives wrong or invented answers | Ground answers in the catalogue and trusted sources only; say so when information is missing |

## 13. Open decisions

1. Supported languages
2. AI providers for cataloguing, image analysis, speech-to-text, translation and the Concierge
3. Reference data for fair-price recommendations
4. Source of trusted craft information for the craft-information assistant
5. Who performs human verification in the demo
6. Accepted identity or craft documents, and legal limits on collecting them (including Aadhaar-related information)
7. Whether the maker's story is per artisan (as in SRS v1.1), per product (as in the proposal), or both
8. Payment gateway, cash on delivery, commission, shipping responsibility and return policy
9. Data retention and deletion periods
10. Hosting and technology stack
11. AI quality thresholds and review triggers
12. Team roles and semester milestones
