# ALLIGO HIRE A PRO ECOSYSTEM: MASTER WORK MAP & ARCHITECTURE SPECIFICATION
**Version:** 2.0.0 — Production-Ready Standard  
**Status:** Canonical Source of Truth  
**Target Modules:** 
- Admin Dashboard: `/admin/dashboard/freelance`, `/admin/dashboard/client-layout.tsx`, `/admin/dashboard/page.tsx`, `/admin/dashboard/layout.tsx`, `/admin/dashboard/promotions`
- Freelancer Dashboard: `/dashboard/freelancer`, `/dashboard/freelancer/layout.tsx`
- Client Dashboard: `/dashboard/client`, `/dashboard/client/layout.tsx`
- Marketplace & Pro Catalog: `/hire-pro`, `/hire-pro/[category]`, `/hire-a-pro`, `/hire-a-pro/[category]`, `/hire-a-pro/[category]/[serviceId]`, `/hire-a-pro/provider/[providerId]`

---

## 1. ECOSYSTEM ARCHITECTURE & WORKFLOW MATRIX

```
┌─────────────────────────────────────────────────────────────────────────────────────────┐
│                                ALLIGO HIRE A PRO ECOSYSTEM                              │
└─────────────────────────────────────────────────────────────────────────────────────────┘
         │                                   │                                  │
         ▼                                   ▼                                  ▼
┌──────────────────┐               ┌───────────────────┐              ┌───────────────────┐
│   CLIENT HUB     │               │    FREELANCER     │              │   ADMIN CONSOLE   │
│  (/hire-a-pro)   │               │(/dashboard/free)  │              │(/admin/dashboard) │
│(/dashboard/client│               │                   │              │                   │
└──────────────────┘               └───────────────────┘              └───────────────────┘
         │                                   │                                  │
         │ Discover Categories / Services   │ Manage Profile / Portfolio       │ Marketplace Radar
         │ Search & Filter Providers         │ Post Gigs / Offer Services       │ Provider Approvals
         │ View Profile & Reviews            │ Find Work / Submit Proposals     │ Service Moderation
         │ Create Project / Brief            │ Accept Contracts / Milestones    │ Dispute Settlement
         │ Book / Hire Service Provider      │ Deliver Tasks / Upload Work      │ Escrow & Payouts
         │ Escrow Payment / M-Pesa / Card   │ Track Earnings & Withdrawals     │ Featured Promotions
         │ Leave Rating & Review             │ Manage Client Reviews            │ Marketplace Metrics
         │                                   │                                  │
         └─────────────────┬─────────────────┴──────────────────────────────────┘
                           │
                           ▼
         ┌───────────────────────────────────┐
         │     CORE STATE ENGINE (FIRESTORE) │
         │ `services`, `proposals`, `jobs`   │
         │ `contracts`, `reviews`, `users`   │
         └───────────────────────────────────┘
```

---

## 2. ROUTES & COMPONENT MAP

### A. Client Experience & Discovery Hub
* **Routes:**
  * `/hire-a-pro`: Main discovery marketplace with mode switch (`jobs` for open briefs / `services` for gig offerings), category grids, subheader search, sidebar multi-criteria filters, and "Post a Brief" modal.
  * `/hire-pro`: Canonically redirects to `/hire-a-pro`.
  * `/hire-a-pro/[category]`: Category/subcategory service catalogue, provider avatars, rating indicators, and pricing cards.
  * `/hire-pro/[category]`: Canonically redirects to `/hire-a-pro/[category]`.
  * `/hire-a-pro/[category]/[serviceId]`: Rich service details with image carousel, tier scope, provider summary, instant order & booking modal with custom requirements and delivery estimates.
  * `/hire-a-pro/provider/[providerId]`: Public provider profile with verification badges, bio, skills, hourly rate, portfolio project showcase, certifications, and active services.
  * `/dashboard/client`: Client control center with active projects, milestone trackers, contract summaries, total spend KPIs, and direct CTA to post new briefs or hire pros.

### B. Freelancer & Provider Workspace
* **Route:** `/dashboard/freelancer`
  * **Availability Switch:** Instant Online/Available toggle synced to provider profile in Firestore.
  * **Profile Management:** Bio, skills, certificates, portfolio showcase, hourly rate, and badge status.
  * **Service/Gig Management:** Create, publish, pause, or update service offerings with tiered pricing.
  * **Find Work & Proposals:** Job board feed with custom proposal submission modal, proposed price, delivery time, and cover letter.
  * **Contracts & Active Milestones:** Live contract status (`active`, `milestone_review`, `completed`), deliverable submission, and client approvals.
  * **Earnings & Wallet:** Real-time earnings balance, net payout ledger, and M-Pesa withdrawal.
  * **Reviews & Ratings:** Public client feedback, 5-star breakdown, and response handling.

### C. Admin Freelance Marketplace Console
* **Route:** `/admin/dashboard/freelance`
  * **Marketplace Overview:** Total Providers, Active Projects, Platform Gross Volume, Net 10-15% Commission Revenue, Average Provider Rating.
  * **Providers Management:** Verification status (`APPROVED`, `PENDING_REVIEW`, `SUSPENDED`), document verification, and skill audits.
  * **Services & Listings Moderation:** Review newly created services, approve/reject listings, feature services.
  * **Projects & Proposals Inspection:** Complete oversight over contracts, milestones, and dispute resolution.
  * **Promotions & Featured Listings (`/admin/dashboard/promotions`):** Featured provider carousel management, ad campaign approvals, and category sponsorship.

---

## 3. FIRESTORE DATA MODELS & SCHEMAS

### Collection: `services/{serviceId}`
```typescript
interface ServiceDocument {
  id: string;
  providerId: string;
  providerName: string;
  title: string;
  description: string;
  categoryId: string;
  categoryName?: string;
  subcategoryId?: string;
  subcategoryName?: string;
  price: number;
  currency: 'KES' | 'USD';
  deliveryDays: number;
  revisions: number;
  images: string[];
  features?: string[];
  status: 'active' | 'pending_review' | 'paused' | 'suspended';
  averageRating: number;
  totalReviews: number;
  createdAt: string | Timestamp;
  updatedAt: string | Timestamp;
}
```

### Collection: `serviceOrders` / `contracts/{contractId}`
```typescript
interface ContractDocument {
  id: string;
  clientId: string;
  clientName: string;
  providerId: string;
  providerName: string;
  serviceId?: string;
  jobId?: string;
  title: string;
  requirements: string;
  amount: number;
  platformFee: number;
  providerEarnings: number;
  currency: 'KES' | 'USD';
  status: 'pending' | 'active' | 'in_review' | 'completed' | 'cancelled' | 'disputed';
  paymentStatus: 'escrowed' | 'released' | 'refunded';
  deliveryDate?: string;
  createdAt: string | Timestamp;
  completedAt?: string | Timestamp;
}
```

### Collection: `serviceJobs` / `projects/{jobId}`
```typescript
interface ProjectJobDocument {
  id: string;
  clientId: string;
  clientName: string;
  title: string;
  description: string;
  category: string;
  budgetType: 'fixed' | 'hourly';
  budget: number;
  status: 'open' | 'in_progress' | 'completed' | 'cancelled';
  proposalsCount: number;
  createdAt: string | Timestamp;
}
```

---

## 4. BUTTONS, HANDLERS & NAVIGATION AUDIT

| Component | Element / Button | ID / Target | Action / Outcome |
|---|---|---|---|
| `/hire-a-pro` | "Post a Brief" | `post-brief-btn` | Opens `ProBriefModal`, submits new job to `serviceJobs`, updates feed. |
| `/hire-a-pro` | "Mode Switcher" | `tab-jobs` / `tab-services` | Switches active view between job listings and freelancer gig catalogue. |
| `/hire-a-pro/[category]/[serviceId]` | "Book / Hire Now" | `book-service-btn` | Opens order modal, triggers `createServiceOrderAction`, creates contract. |
| `/hire-a-pro/provider/[providerId]` | "Contact / Message" | `message-provider-btn` | Initiates real-time conversation or redirects to chat thread. |
| `/dashboard/freelancer` | "Availability Switch" | `freelancer-avail-toggle` | Updates online availability state in Firestore user/provider record. |
| `/dashboard/client` | "Post New Job" | `client-post-job-btn` | Opens brief creator and redirects to active projects. |
| `/admin/dashboard/freelance` | "Approve Provider" | `admin-approve-freelancer` | Sets verification status to `APPROVED`. |
| `/admin/dashboard/freelance` | "Approve / Flag Service"| `admin-mod-service` | Approves or suspends service listings across the marketplace. |

---

## 5. VERIFICATION & QUALITY STANDARDS
1. **0 Broken Links or Redirects:** All `/hire-pro/*` aliases route cleanly to `/hire-a-pro/*`.
2. **Real Database Transactions:** Orders create real escrow records and contracts in Firestore.
3. **TypeScript & Lint Compliance:** 0 syntax warnings or Next.js build errors.
