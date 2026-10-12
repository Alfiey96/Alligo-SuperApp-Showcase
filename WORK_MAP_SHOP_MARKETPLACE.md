# ALLIGO SHOP & E-COMMERCE MULTI-VENDOR MARKETPLACE: MASTER WORK MAP & ARCHITECTURE SPECIFICATION
**Version:** 2.0.0 — Production-Ready Standard  
**Status:** Canonical Source of Truth  
**Target Modules:** 
- Marketplace Storefront: `/shop`, `/shop/[category]`, `/shop/[category]/[productId]`, `/shop/checkout`, `/shop/order-success`
- Seller Dashboard: `/dashboard/seller`, `/dashboard/seller/inventory`, `/dashboard/seller/orders`, `/dashboard/seller/sales`, `/dashboard/seller/promotions`, `/dashboard/seller/settings`
- Buyer Orders & Tracking: `/dashboard/buyer`, `/dashboard/buyer/orders`, `/dashboard/buyer/tracking/[id]`
- Admin E-Commerce Console: `/admin/dashboard/ecommerce`, `/admin/dashboard/promotions`, `/admin/dashboard/finance`

---

## 1. ECOSYSTEM ARCHITECTURE & WORKFLOW MATRIX

```
┌─────────────────────────────────────────────────────────────────────────────────────────┐
│                           ALLIGO SHOP / MARKETPLACE ECOSYSTEM                           │
└─────────────────────────────────────────────────────────────────────────────────────────┘
         │                                   │                                  │
         ▼                                   ▼                                  ▼
┌──────────────────┐               ┌───────────────────┐              ┌───────────────────┐
│    BUYER HUB     │               │  SELLER CONSOLE   │              │   ADMIN CONSOLE   │
│     (/shop)      │               │(/dashboard/seller)│              │(/admin/dashboard) │
│(/dashboard/buyer)│               │                   │              │                   │
└──────────────────┘               └───────────────────┘              └───────────────────┘
         │                                   │                                  │
         │ Browse Catalogs & Bento Showcases │ Manage Catalog & Variations      │ Merchant Vetting
         │ Filter by Department / Brand / KES│ Real-time Stock Tracking         │ Product Moderation
         │ Product Variant & Stock Discovery │ Order Fulfillment Pipeline       │ Financial Commission Split
         │ Cart Drawer with Stock Ceilings   │ Sponsored Ad Promotions          │ Logistics & Sorting Hub
         │ Checkout (M-Pesa / Card / Wallet) │ Financial Ledger & Earnings      │ Escrow Release & Disputes
         │ Track 8-Stage Order Delivery      │ Store Profile & Custom Policies  │ Revenue Analytics
         │                                   │                                  │
         └─────────────────┬─────────────────┴──────────────────────────────────┘
                           │
                           ▼
         ┌───────────────────────────────────┐
         │     CORE STATE ENGINE (FIRESTORE) │
         │ `products`, `orders`, `promotions`│
         │ `sellerOrders`, `wallets`, `users`│
         └───────────────────────────────────┘
```

---

## 2. 8-STAGE ORDER FULFILLMENT PIPELINE STATE MACHINE

Alligo Shop enforces an authoritative 8-stage state machine governing order progression:

```
[1. order_placed] 
   Initial order creation with server-side authoritative price verification & stock reservation.
       │
[2. order_confirmed] 
   Seller reviews inward order and confirms stock availability.
       │
[3. order_packaged] 
   Seller packages and seals parcel with tamper-evident security.
       │
[4. sorting_station] 
   Seller hands over parcel to Alligo Logistics courier / hub dispatch.
       │
[5. sorting_station_arrived] 
   Alligo Central Sorting Hub scans and approves receipt.
       │
[6. in_transit] 
   Dispatched from Sorting Hub on delivery route to customer.
       │
[7. collection_point] 
   Staged at local collection locker / station for customer pickup.
       │
[8. delivered] 
   Customer confirms receipt. Authoritative escrow release completes; seller net credited.
```

---

## 3. PRODUCTION HARDENING & SECURITY ARCHITECTURE

### A. Authoritative Price Verification
- In `src/app/actions/order-actions.ts` (`createCheckoutOrdersAction`), incoming item prices from the client are treated as untrusted proposals.
- An atomic Firestore transaction looks up every product directly in the `products` collection, verifies active status, verifies variant prices and stock levels, and computes the authoritative total.

### B. Multi-Order Webhook Iteration & Enum Safety
- In `src/app/api/webhooks/paystack/route.ts` (`case 'ECOMMERCE'`), all orders grouped under a multi-vendor checkout (`metadata.orderIds`) are iteratively transitioned.
- Payment status is set to `paid` while keeping the valid order fulfillment pipeline stage (`order_placed` / `order_confirmed`).

### C. Client Mutation Locking in Security Rules
- Direct client updates to `/orders` and `/sellerOrders` are restricted to Admin SDK in `firestore.rules`.
- Status updates run through `updateOrderStatusBySeller` with role validation (sellers restricted to `order_confirmed`, `order_packaged`, `sorting_station`; delivery/escrow stages restricted to Admin/Courier).

### D. Ad Promotions Wallet Deduction
- `submitAdPromotionCampaignAction` deducts promotion fees atomically from the seller's wallet with ledger entry `AD_PROMOTION_PAYMENT`.
- If unpaid, promotions remain in `pending_payment` state without active display in buyer streams.
