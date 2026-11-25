# Hunar
## Software Requirements Specification (SRS)

**Project:** Hunar  
**Project type:** Application + Product  
**Project context:** Capstone project based on the SIH 2026 problem statement referenced by the project team  
**Document status:** Working Draft  
**Version:** 1.1

---

## 1. Introduction

### 1.1 Purpose

This document describes the software requirements for **Hunar**, a mobile-first marketplace designed to help marginalized artisans showcase and sell handmade products throughout the year. It defines the intended users, major system capabilities, functional requirements, non-functional requirements, and project constraints.

The document is intended to guide the project team during design, implementation, and testing, and to provide a shared reference for mentors and evaluators.

### 1.2 Project Overview

Hunar is a marketplace where artisans can create a presence, present their craft and products, and reach buyers beyond occasional physical exhibitions, melas, and local events.

The platform is intended to reduce barriers that prevent artisans from participating in online commerce. It will support assisted product cataloguing, voice-based maker stories, product discovery, regional craft discovery, and fair-price guidance.

Hunar is a marketplace within which artisans have storefront profiles. It is **not** a website builder that asks each artisan to create and independently attract customers to a separate online store. It is also not intended to be a CRM in its first release.

### 1.3 Problem Statement

Many marginalized artisans have limited opportunities to showcase their work and reach buyers outside occasional exhibitions and local markets. Creating and maintaining an independent online store may not solve this problem if artisans do not already have an online customer base. Language barriers, limited digital literacy, unreliable connectivity, and difficulty presenting products online can further restrict access to wider markets.

Hunar aims to provide a shared marketplace where buyers can discover artisans and their products, while accessible tools help artisans present their work.

### 1.4 Intended Audience

This document is intended for:
- Project team members responsible for development and testing.
- Project mentors and academic evaluators.
- Coordinators or facilitators involved in artisan onboarding.
- Stakeholders reviewing the proposed system.

### 1.5 Definitions and Acronyms

| Term | Description |
|---|---|
| Artisan | A person or group that creates handmade craft products. |
| Buyer | A user who browses and purchases products through Hunar. |
| Coordinator | A facilitator who assists artisans with onboarding and recording product or story media. |
| Storefront profile | An artisan's profile page within the Hunar marketplace. |
| PWA | Progressive Web Application. |
| OTP | One-Time Password. |
| AI | Artificial Intelligence. |
| SRS | Software Requirements Specification. |

---

## 2. Overall Description

### 2.1 Product Perspective

Hunar will operate as one shared marketplace. The home page will introduce the platform and provide access to products, featured artisans, and craft-region discovery. Buyers will be able to reach an artisan through marketplace browsing or through a shareable artisan profile link, such as a link shared on WhatsApp.

The intended delivery approach is a mobile-first web application with Progressive Web Application (PWA) support. A native mobile application is not planned for the first release.

### 2.2 Product Goals

The primary goals of Hunar are to:

1. Give artisans an ongoing digital marketplace presence.
2. Help artisans create product listings with less manual writing.
3. reduce language barriers through voice transcription and translation.
4. Help buyers understand the artisan and the product's regional context.
5. Provide product presentation and discovery features suited to handmade goods.
6. Offer fair-price guidance that helps identify possible underpricing without preventing an artisan from setting a price.

### 2.3 User Classes and Roles

#### 2.3.1 Artisan

An artisan may be an individual maker or a member of an artisan group. The system should account for users who may have limited digital literacy, use budget smartphones, have intermittent internet access, or prefer not to appear on camera.

Artisans will be able to:
- Register and access their account.
- Create and maintain a storefront profile.
- Add product photographs and product information.
- Review and edit AI-generated listing suggestions before publication.
- Record or provide a voice introduction and maker story.
- View verification status and relevant feedback.
- Manage products and orders, subject to the features enabled in the release.

#### 2.3.2 Buyer

A buyer is a person who discovers artisans and handmade products through the marketplace. Buyers will be able to:
- Browse and search products.
- Filter and explore products.
- View product details and artisan storefront profiles.
- Access maker stories and product-viewing media.
- Use the order and payment flow provided by the release.

#### 2.3.3 Coordinator

A coordinator is a facilitator, such as an SHG or cooperative representative, who helps artisans use the platform. A coordinator may assist with onboarding and may record product photographs, making-process videos, or voice content on an artisan's behalf.

#### 2.3.4 Admin/Verifier

An admin/verifier is an authorized user responsible for platform administration and human review of verification evidence or listings flagged for review. Verification decisions must not rely solely on an automated AI detector.

### 2.4 Operating Environment

The system is intended primarily for smartphones and should also remain usable on desktop browsers. It must account for slow networks and intermittent connectivity. The final supported browser, operating-system, hosting, and backend environment will be specified in the Design Document.

### 2.5 Design and Implementation Constraints

- The experience must be mobile-first.
- The interface should be understandable to users with limited digital literacy.
- The system must account for low-bandwidth and intermittent-network conditions.
- Artisans must retain control over AI-generated product content before publication.
- The system must not expose an artisan's exact GPS location to buyers.
- The first release will use payment gateway test mode only; it will not perform real settlement or payouts.
- CRM capabilities are outside the scope of the first release.
- The first release will be delivered as a mobile-first web application with PWA support.

### 2.6 Assumptions and Dependencies

- Artisans or coordinators will be able to provide product photographs and relevant product information.
- AI cataloguing, speech-to-text, and translation services may depend on external providers, subject to provider selection.
- Payment gateway test-mode access will be available for development and demonstration.
- Human verifiers or designated project team members will be available to review verification evidence in the demonstration.
- Reference data for fair-price recommendations will need to be identified before that feature can be validated.

---

## 3. System Features and Functional Requirements

The requirements below describe the intended behaviour of the first release. Items whose implementation depends on an unresolved project decision are marked **[TBD]**.

### 3.1 Marketplace and Home Page

**Description:** The home page will act as the entry point to the shared marketplace rather than directing artisans to create independent stores.

| ID | Requirement |
|---|---|
| FR-MKT-01 | The system shall display a brief introduction to Hunar on the home page. |
| FR-MKT-02 | The system shall provide access to product discovery from the home page. |
| FR-MKT-03 | The system shall display featured artisan profiles on the home page. |
| FR-MKT-04 | The system shall provide access to craft-region discovery through an interactive map. |
| FR-MKT-05 | The system shall provide a separate About page. |
| FR-MKT-06 | The system may display a short, skippable introduction to first-time users. |

### 3.2 Artisan Storefront Profile

**Description:** Each artisan or artisan group will have a storefront profile inside the Hunar marketplace.

| ID | Requirement |
|---|---|
| FR-PRO-01 | The system shall provide each artisan or artisan group with a storefront profile. |
| FR-PRO-02 | The profile shall display the artisan's or group's name and profile photograph, where provided. |
| FR-PRO-03 | The profile shall display the artisan's craft and village/region information. |
| FR-PRO-04 | The profile shall provide access to the artisan's voice introduction and its translated transcript, when available. |
| FR-PRO-05 | The profile shall display the artisan's published products. |
| FR-PRO-06 | The profile shall display the artisan's verification status. |
| FR-PRO-07 | The system shall provide a shareable link for an artisan's storefront profile. |
| FR-PRO-08 | The system shall allow buyers to reach an artisan's profile through marketplace browsing or a shared profile link. |
| FR-PRO-09 | The system shall show origin at a village/region level and shall not expose the artisan's exact GPS coordinates to buyers. |

### 3.3 User Registration and Authentication

**Description:** Authentication requirements differ by user role. The artisan sign-in flow should be suitable for mobile use.

| ID | Requirement |
|---|---|
| FR-AUTH-01 | The system shall support phone-number OTP authentication for artisans. |
| FR-AUTH-02 | The system shall support account access for buyers. |
| FR-AUTH-03 | The system shall support authorized admin/verifier account access. |
| FR-AUTH-04 | The system shall provide an account recovery or password-reset flow where password-based authentication is enabled. |
| FR-AUTH-05 | The system shall restrict administrative and verification functions to authorized users. |

### 3.4 Artisan Verification and Product Genuineness

**Description:** Hunar will distinguish between an artisan who can use the marketplace and an artisan who has completed the platform's verification process. An artisan may publish as **Unverified**. The Verified badge means that the artisan has been checked through Hunar's review process; it is not a guarantee that every product is handmade.

The verification process considers three separate questions:
1. Whether submitted media appears to be an authentic photograph rather than AI-generated or copied media.
2. Whether there is evidence that the artisan makes the product.
3. Whether the product is handmade. This cannot be conclusively proven by software alone and must not be represented as a certainty.

| ID | Requirement |
|---|---|
| FR-VER-01 | The system shall allow an artisan to use the marketplace and publish eligible listings with an **Unverified** status. |
| FR-VER-02 | The system shall not require an artisan to obtain a Verified badge before publishing products. |
| FR-VER-03 | The system shall support submission of evidence for artisan verification. |
| FR-VER-04 | Verification evidence may include identity or craft-related proof, a making-process video, or coordinator vouching. |
| FR-VER-05 | The system shall support human review of submitted verification evidence. |
| FR-VER-06 | The system shall display a Verified badge only after the required evidence and human review process have been completed successfully. |
| FR-VER-07 | The system shall not describe the Verified badge as an absolute guarantee that a product is handmade. |
| FR-VER-08 | A making-process video shall not require the artisan's face to be visible. |
| FR-VER-09 | The system shall allow a coordinator to record a making-process video on an artisan's behalf. |
| FR-VER-10 | The system shall route AI-flagged listings for human review rather than automatically rejecting them. |
| FR-VER-11 | The system shall retain the original submitted product image privately for verification processing. |
| FR-VER-12 | The system shall calculate and retain a SHA-256 hash of the original submitted image. |
| FR-VER-13 | The verification pipeline should support image-analysis checks through replaceable detection components. |
| FR-VER-14 | The verification pipeline should support checks of available image metadata, provenance/watermark signals, similarity across submitted photographs, duplicate images, and consistency between product photographs and making-process media. |
| FR-VER-15 | The system should combine available verification signals into a risk assessment that can identify submissions for human review. |
| FR-VER-16 | The system shall not automatically reject an artisan or listing solely because an AI detector flags it. |

**Verification implementation note:** AI-image detectors and image metadata are imperfect signals. Their results must be treated as indicators for review, not proof. The project should evaluate the verification approach using a small test set containing genuine photographs, AI-generated images, screen recaptures, and duplicate images. Precision and recall should be reported. Exact thresholds and scoring weights remain **[TBD]**.

### 3.5 AI-Assisted Product Listing and Cataloguing

**Description:** Artisans will provide photographs of a product. AI will generate a draft listing that the artisan can review and edit.

| ID | Requirement |
|---|---|
| FR-LST-01 | The system shall allow an artisan or authorized coordinator to submit product photographs. |
| FR-LST-02 | The listing flow should support three to four product photographs from different angles. |
| FR-LST-03 | The system shall use AI to suggest a product title. |
| FR-LST-04 | The system shall use AI to suggest a product description. |
| FR-LST-05 | The system shall use AI to suggest a product category. |
| FR-LST-06 | The system shall use AI to suggest relevant product tags. |
| FR-LST-07 | The system shall present AI-generated listing content as a draft. |
| FR-LST-08 | The artisan shall be able to review and edit the generated title, description, category, and tags. |
| FR-LST-09 | The system shall require artisan or authorized user approval before a listing is published. |
| FR-LST-10 | The system shall not automatically publish an AI-generated listing without review and approval. |
| FR-LST-11 | The system shall retain the submitted original image privately for verification purposes before any image compression or transformation. |

### 3.6 Maker's Story and Language Support

**Description:** Artisans should be able to introduce themselves and explain their craft in their own voice. The system will provide transcription and translation to help buyers understand the story.

| ID | Requirement |
|---|---|
| FR-STO-01 | The system shall allow an artisan or coordinator to record or submit a voice introduction. |
| FR-STO-02 | The system shall support speech-to-text transcription of the submitted voice recording. |
| FR-STO-03 | The system shall support translation of the transcript into the selected supported language(s). |
| FR-STO-04 | The system shall associate the voice recording and translated transcript with the artisan's storefront profile. |
| FR-STO-05 | The system should allow buyers to access the original voice recording and its transcript. |
| FR-STO-06 | The system shall identify the languages supported by the application. The initial language list is **[TBD]**. |

### 3.7 Product Discovery, Search, and Browsing

**Description:** Buyers will discover products through the shared marketplace and will be able to explore artisan profiles from product listings.

| ID | Requirement |
|---|---|
| FR-DIS-01 | The system shall display published products in the marketplace. |
| FR-DIS-02 | The system shall provide product search. |
| FR-DIS-03 | The system shall provide relevant product filters. The final filter list is **[TBD]**. |
| FR-DIS-04 | The system shall support browsing products in batches of approximately 12 items. |
| FR-DIS-05 | The system shall load the next batch when the buyer reaches the end of the current list using the supported pull-up/load-more interaction. |
| FR-DIS-06 | The interface shall provide a visible hint that more products can be loaded. |
| FR-DIS-07 | The system shall provide a Load more fallback for desktop and keyboard users. |
| FR-DIS-08 | The system shall preserve the buyer's browsing position and applied filters when the buyer returns from a product detail page. |
| FR-DIS-09 | The product-list API shall support pagination using page size and offset. |
| FR-DIS-10 | The system shall allow buyers to open an artisan storefront from a product listing or product detail page. |

### 3.13 AI-Powered Buyer Assistance

**Description:** AI features will help buyers discover, understand, and compare handmade products using the Hunar catalogue and trusted craft information.

| ID | Requirement |
|---|---|
| FR-AIB-01 | The system shall provide an AI Craft Concierge that accepts natural-language product discovery requests. |
| FR-AIB-02 | The Craft Concierge shall support discovery using budget, occasion, recipient, style, and buyer preferences when provided. |
| FR-AIB-03 | AI recommendations shall be drawn from products available in the Hunar catalogue and shall link to the relevant product pages. |
| FR-AIB-04 | The system shall support semantic search using intent, context, style, or descriptive phrases in addition to keyword search. |
| FR-AIB-05 | The Craft Concierge may assist with product comparison and gift discovery using available product attributes. |
| FR-AIB-06 | The system shall provide a craft-information assistant grounded in artisan profiles and trusted craft information available to Hunar. |
| FR-AIB-07 | The AI shall not invent product listings, availability, product attributes, or cultural facts when information is unavailable. |

### 3.8 Product Viewing and Craft-Region Map

**Description:** Hunar will help buyers inspect products and discover crafts by region.

| ID | Requirement |
|---|---|
| FR-VIEW-01 | The system shall display the available photographs of a product. |
| FR-VIEW-02 | The system shall support a photo-based product spin using multiple product photographs to simulate a 360-degree viewing experience. |
| FR-VIEW-03 | Full 3D product reconstruction is out of scope for the first release. |
| FR-MAP-01 | The system shall provide an interactive map of India for craft-region discovery. |
| FR-MAP-02 | The map shall show relevant craft or product locations at a region level. |
| FR-MAP-03 | Selecting a map location shall allow buyers to explore associated crafts or products. |
| FR-MAP-04 | The map shall not reveal an artisan's exact GPS coordinates. |

### 3.9 Fair-Price Recommendation

**Description:** The fair-price feature will provide guidance to artisans and identify possible underpricing. It must not prevent an artisan from choosing or publishing a price.

| ID | Requirement |
|---|---|
| FR-PRI-01 | The system shall provide a suggested fair-price range or recommendation for an eligible product. |
| FR-PRI-02 | The system should consider available reference-price information when generating a recommendation. |
| FR-PRI-03 | The system shall warn the artisan when the entered price appears significantly below the system's recommendation. |
| FR-PRI-04 | The system shall allow the artisan to continue with their chosen price after the warning. |
| FR-PRI-05 | The system shall not block product publication solely because the artisan's price is below the recommended amount. |
| FR-PRI-06 | The source of reference-price data and the recommendation method shall be documented before implementation. **[TBD]** |

### 3.11 Cart, Orders, and Payments

**Description:** The first release will demonstrate the order and payment flow using a payment gateway in test mode only.

| ID | Requirement |
|---|---|
| FR-ORD-01 | The system shall allow buyers to add eligible products to a cart. |
| FR-ORD-02 | The system shall allow buyers to review the products in their cart before proceeding. |
| FR-ORD-03 | The system shall support creation and tracking of orders within the capabilities of the first release. |
| FR-ORD-04 | The system shall integrate with a payment gateway in test mode. |
| FR-ORD-05 | The system shall not perform real-money settlement or artisan payouts in release 1.0. |
| FR-ORD-06 | The system shall record the payment-test result associated with an order. |
| FR-ORD-07 | The final payment gateway, cash-on-delivery support, shipping responsibility, and return policy are **[TBD]**. |

### 3.12 Admin and Verification Management

| ID | Requirement |
|---|---|
| FR-ADM-01 | The system shall provide authorized administrators with access to relevant platform-management functions. |
| FR-ADM-02 | The system shall provide authorized verifiers with access to submitted verification evidence. |
| FR-ADM-03 | The system shall allow a verifier to review evidence and record a verification decision. |
| FR-ADM-04 | The system shall allow flagged listings to be reviewed by an authorized human reviewer. |
| FR-ADM-05 | The system should record the status and outcome of verification reviews. |
| FR-ADM-06 | The exact administrative dashboard capabilities and reviewer workflow are **[TBD]**. |

---

## 4. External Interface Requirements

### 4.1 User Interface

- The application shall use a mobile-first interface.
- The interface should use clear language, recognizable actions, and simple navigation.
- The artisan listing flow should minimize typing and support AI-assisted content creation.
- The artisan should be able to review AI-generated content before publication.
- The interface should provide clear verification-status labels, including Unverified and Verified.
- Product browsing should communicate when more results are available and provide a keyboard-accessible fallback.
- The first-time introduction should be skippable.

### 4.2 Software Interfaces

The following external services or integrations are anticipated, but provider selection is pending:

- OTP delivery service.
- AI service for product cataloguing.
- Speech-to-text service.
- Translation service.
- Image-analysis or AI-image-detection service.
- Payment gateway test environment.
- Map service or map data source.

The selected providers, API contracts, usage limits, and costs will be documented in the Design Document.

### 4.3 Hardware Interfaces

The application is expected to use a smartphone camera and microphone for product photographs, making-process media, and voice introductions. A coordinator may capture media on behalf of an artisan.

---

## 5. Non-Functional Requirements

### 5.1 Performance and Network Conditions

| ID | Requirement |
|---|---|
| NFR-PER-01 | The application should remain usable on slow and intermittent mobile networks. |
| NFR-PER-02 | The application should load product images in a way that is appropriate for limited bandwidth. |
| NFR-PER-03 | The system should provide clear feedback when an operation is waiting, fails, or needs to be retried. |
| NFR-PER-04 | Product browsing should retrieve results in paginated batches rather than requiring the entire catalogue to load at once. |

### 5.2 Security

| ID | Requirement |
|---|---|
| NFR-SEC-01 | The system shall restrict protected functions according to the user's authorized role. |
| NFR-SEC-02 | The system shall protect authentication credentials and authentication tokens. |
| NFR-SEC-03 | The system shall protect private verification media from public access. |
| NFR-SEC-04 | The system shall prevent unauthorized users from accessing private conversations and account information. |
| NFR-SEC-05 | The system shall use secure communication for data transmitted between the client and server. |

### 5.3 Privacy and Consent

| ID | Requirement |
|---|---|
| NFR-PRI-01 | The system shall not publicly expose exact GPS coordinates of artisans. |
| NFR-PRI-02 | The system shall obtain appropriate consent before collecting or publishing voice, image, or video content. |
| NFR-PRI-03 | The system shall explain the purpose of media collected for verification. |
| NFR-PRI-04 | The system shall restrict access to private verification evidence. |
| NFR-PRI-05 | The system shall define data-retention and deletion rules before production deployment. **[TBD]** |

### 5.4 Accessibility and Usability

| ID | Requirement |
|---|---|
| NFR-USE-01 | The interface should be understandable to users with limited digital literacy. |
| NFR-USE-02 | The artisan experience should minimize unnecessary typing. |
| NFR-USE-03 | Important actions and status information should be clearly identifiable. |
| NFR-USE-04 | The application should support keyboard navigation for relevant browsing interactions. |
| NFR-USE-05 | The application should provide language support appropriate to its intended artisan and buyer communities. The initial language list is **[TBD]**. |

### 5.5 Reliability and AI Limitations

| ID | Requirement |
|---|---|
| NFR-REL-01 | Failure of an AI service should not silently publish incomplete or unreviewed listing content. |
| NFR-REL-02 | The system should communicate when AI-generated content or translations are unavailable. |
| NFR-REL-03 | Automated image checks shall be treated as risk indicators and not conclusive proof of authenticity. |
| NFR-REL-04 | Human review shall remain available for flagged verification cases. |
| NFR-REL-05 | The project shall evaluate image-verification performance using a test set and report precision and recall. |

---

## 6. Data and Information Requirements

The system is expected to manage the following broad categories of information:

- User account and role information.
- Artisan and artisan-group storefront profiles.
- Craft, village, and regional information.
- Product details, categories, tags, prices, and publication status.
- Product photographs and associated verification metadata.
- Voice recordings, transcripts, and translated maker stories.
- Verification evidence, review status, and decisions.
- Cart and order information.
- Payment gateway test-mode transaction results.
- Reference data used by the fair-price recommendation feature.

Exact database entities, relationships, retention periods, and storage choices will be specified in the Design Document.

---

## 7. Use Cases

### UC-01: Artisan Creates a Product Listing

**Primary actor:** Artisan  
**Supporting actor:** Coordinator (optional)

**Preconditions:**
- The artisan is able to access their account.
- Product photographs are available.

**Main flow:**
1. The artisan opens the product-listing flow.
2. The artisan or coordinator submits product photographs.
3. The system generates draft title, description, category, and tags using AI.
4. The system displays the generated content to the artisan.
5. The artisan reviews and edits the content.
6. The artisan approves the listing for publication.
7. The system publishes the listing with the applicable verification status.

**Alternative flow:**
- If AI generation is unavailable, the system should communicate the failure and provide an appropriate manual or retry path.
- If a listing is flagged for review, it is routed for human review rather than automatically rejected.

### UC-02: Buyer Discovers an Artisan

**Primary actor:** Buyer

**Main flow:**
1. The buyer opens the Hunar marketplace.
2. The buyer browses, searches, or filters products.
3. The buyer opens a product.
4. The buyer opens the associated artisan storefront profile.
5. The buyer views the artisan's craft, region, story, products, and verification status.
6. The buyer may continue to explore the artisan's published products.

### UC-03: Artisan Submits Verification Evidence

**Primary actor:** Artisan  
**Supporting actors:** Coordinator, Admin/Verifier

**Main flow:**
1. The artisan or coordinator opens the verification flow.
2. The artisan or coordinator submits the available evidence.
3. The system stores the evidence privately.
4. The system performs the configured automated checks.
5. If the case requires review, it is made available to an authorized verifier.
6. The verifier reviews the evidence and records a decision.
7. The system updates the artisan's verification status.

**Alternative flow:**
- An artisan who has not completed verification may continue using the marketplace with an Unverified status, subject to the platform's listing rules.

### UC-04: Buyer Browses More Products

**Primary actor:** Buyer

**Main flow:**
1. The buyer opens a product listing page.
2. The system displays an initial batch of approximately 12 products.
3. The buyer reaches the end of the current batch.
4. The system provides the next batch through the supported load-more interaction.
5. The buyer can continue browsing without losing applied filters when opening and returning from a product detail page.

### UC-05: Artisan Records a Maker's Story

**Primary actor:** Artisan  
**Supporting actor:** Coordinator (optional)

**Main flow:**
1. The artisan or coordinator records a voice introduction.
2. The system processes the recording into text.
3. The system translates the transcript into supported language(s).
4. The recording and transcript are associated with the artisan's storefront profile.
5. Buyers can access the story from the profile.

---

## 8. Verification and Acceptance Considerations

The following points should be considered during system testing and project evaluation:

1. An artisan must be able to publish as Unverified without first obtaining a Verified badge.
2. AI-generated product content must remain a draft until approved by the artisan or an authorized user.
3. A making-process video must be possible without showing the artisan's face.
4. Exact GPS coordinates must not appear on public artisan or product pages.
5. The product-browsing interface must support paginated loading and a desktop/keyboard fallback.
6. The fair-price feature must warn about possible underpricing without blocking the artisan's chosen price.
7. Payment flows in release 1.0 must use test mode and must not perform real settlement or payouts.
8. Image-verification testing should include real photographs, AI-generated images, screen recaptures, and duplicates, with precision and recall reported.

---

## 9. Scope and Limitations

### 9.1 In Scope for the First Release

- Shared marketplace and artisan storefront profiles.
- Artisan and buyer user flows.
- Artisan phone OTP authentication.
- AI-assisted product listing drafts with human approval.
- Voice-based maker stories with transcription and translation.
- Product discovery, search, and browsing.
- AI-powered buyer assistance through Craft Concierge, semantic search, and craft-information support.
- Product photo-spin as the baseline 360-degree experience.
- Craft-region map.
- Fair-price guidance and underpricing warning.
- Artisan verification workflow with human review.
- A basic cart/order/payment demonstration.
- Payment gateway test mode only.

### 9.2 Out of Scope or Deferred

- Independent store-builder functionality similar to Dukaan or Shopify.
- CRM functionality in the first release.
- NFTs or blockchain-based product authenticity claims.
- Real-money settlement and artisan payouts in release 1.0.
- Full 3D product reconstruction; photo-based product spin is the first-release baseline.
- A claim that software can conclusively prove a product is handmade.

---

## 10. Open Questions and Pending Decisions

The following decisions must be resolved during planning and design:

1. Which languages will be supported in the first release?
2. Which AI providers will be used for cataloguing, image analysis, speech-to-text, and translation?
3. What reference-price data will support fair-price recommendations?
4. Who will perform human verification in the demonstration?
5. Which identity or craft documents will be accepted for verification?
6. What legal and privacy limits apply to collecting identity documents, including Aadhaar-related information?
7. Which payment gateway will be used in test mode?
8. Will cash on delivery be supported?
9. What commission, if any, will the platform apply?
10. Who will be responsible for shipping and fulfilment?
11. What return policy will apply to handmade products?
12. What data-retention and deletion periods will apply?
13. What hosting platform and technology stack will be used?
14. What AI quality thresholds and review triggers will be used?
15. What are the team roles, semester milestones, and delivery responsibilities?

---

## 11. Future Design Documentation

The Design Document will define the implementation details that are intentionally not fixed in this SRS, including:

- System architecture and technology stack.
- Database schema and data relationships.
- API design and pagination contracts.
- AI cataloguing, speech, translation, and verification pipelines.
- Image processing and storage strategy.
- UI/UX flows and responsive behaviour.
- Authentication and authorization design.
- Security controls, privacy controls, and data retention.
- External packages, services, and deployment approach.

---

## 12. Revision History

| Version | Date | Description |
|---|---|---|
| 1.0 | 18 November 2026 | Initial structured Markdown draft based on the current Hunar project outline and decisions. |
| 1.1 | 19 November 2026 | Reviewed against project requirements; added buyer-side AI assistance, clarified PWA and photo-spin scope, and removed unnecessary first-release messaging, reviews, and optional authentication paths. |
