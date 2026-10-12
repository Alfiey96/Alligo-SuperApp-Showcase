# ALLIGO RIDE-HAILING ECOSYSTEM: MASTER WORK MAP & ARCHITECTURE SPECIFICATION
**Version:** 2.0.0 — Production-Ready Standard  
**Status:** Canonical Source of Truth  
**Target Modules:** Rider Client (`/ride`), Driver Dashboard (`/dashboard/driver`), Admin Control Console (`/admin/dashboard/ride-hailing`), Backend APIs & Firestore State Machine.

---

## 1. ECOSYSTEM ARCHITECTURE & WORKFLOW MATRIX

```
┌─────────────────────────────────────────────────────────────────────────────────────────┐
│                                ALLIGO RIDE-HAILING SYSTEM                                │
└─────────────────────────────────────────────────────────────────────────────────────────┘
         │                                   │                                  │
         ▼                                   ▼                                  ▼
┌──────────────────┐               ┌───────────────────┐              ┌───────────────────┐
│   RIDER CLIENT   │               │  DRIVER DASHBOARD │              │   ADMIN CONSOLE   │
│  (/ride)         │               │(/dashboard/driver)│              │(/admin/dashboard) │
└──────────────────┘               └───────────────────┘              └───────────────────┘
         │                                   │                                  │
         │ Request Ride                      │ Duty Online                      │ Live Radar
         │ Select Tier                       │ Scan Radius                      │ Fare Overrides
         │ Choose Payer                      │ Accept Job                       │ Driver Approval
         │ Track Driver GPS                  │ Arrival / Start / Done           │ Dispute Settle
         │ Pay / Rate Driver                 │ Cash / M-Pesa Settle             │ Revenue Audit
         │                                   │                                  │
         └─────────────────┬─────────────────┴──────────────────────────────────┘
                           │
                           ▼
         ┌───────────────────────────────────┐
         │     CORE STATE ENGINE (FIRESTORE) │
         │   `rides` / `drivers` / `wallets` │
         │   Atomic Transactions & Pusher    │
         └───────────────────────────────────┘
```

---

## 2. ROUTES & COMPONENT MAP

### A. Rider Workspace
* **Route:** `/ride`
  * **Top Bar / Navigation:** `PrimaryHeader`, `TopNavigationTabs` (Active tab: Ride), `MobileNavBar`.
  * **Map Viewport:** Interactive `LeafletMap` with pickup/dropoff markers, live driver vehicle pin, and dynamic route polylines.
  * **Route Setup Card:** `PlaceAutocomplete` (Pickup & Destination), GPS Quick-Locate button (`Crosshair`), swap addresses.
  * **Vehicle Catalog:** 8 Ride Tiers (`moto`, `e-moto`, `lite`, `basic`, `x`, `comfort`, `xl`, `xxl`) and 5 Courier Tiers.
  * **Who is Riding Selector:** "For Myself" vs. "For Someone Else" (Contact picker / manual name & phone).
  * **Payment Responsibility Modal:** Booker Pays Now (M-Pesa STK / Card) vs. Booker Pays After Trip vs. Passenger Pays Cash to Driver.
  * **Live Dispatch Radar:** Pulsing radar, 15s driver match timeout, direct cancellation.
  * **Active Trip Card:** Driver profile (Photo/Avatar, Name, Phone, Vehicle Model, Plate, Rating), Emergency SOS, Live distance/ETA to pickup/destination.
  * **Completion & Rating Modal:** 5-star rating, driver tip, receipt breakdown, M-Pesa payment verification.

### B. Driver Workspace
* **Route:** `/dashboard/driver` (Main Dispatch & Live Map)
  * **Sub-routes:**
    * `/dashboard/driver/inbox`: Job inbox, available requests, accepted assignments, past history.
    * `/dashboard/driver/earnings`: Real-time driver wallet balance, 80/20 net split breakdown, daily/weekly metrics, payout request.
    * `/dashboard/driver/performance`: Acceptance rate, on-time arrivals, driver rating score (5.0 scale), safety index.
    * `/dashboard/driver/account`: Vehicle documents, license plate, inspection status, payout phone number, emergency contacts.
    * `/dashboard/driver/support`: 24/7 SOS panic broadcast, help tickets, live dispatch support.
  * **Duty Toggle:** Instant ONLINE / OFFLINE switch with browser GPS location broadcast to `driverLocations` and `drivers/{uid}`.
  * **Dispatch Offer Overlay:** High-priority job card with 15s timer, pickup distance, gross fare, net earnings (80%), Accept & Decline buttons.
  * **Active Trip Workflow Controls:**
    1. `ACCEPTED` → Button: **"Arrived at Pickup"** (notifies rider).
    2. `ARRIVED` → Button: **"Start Trip"** (starts route navigation).
    3. `IN_PROGRESS` → Button: **"Complete Trip"** (triggers fare settlement & geofence validation).
  * **Post-Trip Settlement:**
    * If Paid (M-Pesa / Card): Green "Payment Settled" confirmation with wallet credit.
    * If Cash Unpaid: Red "Customer Not Yet Paid" prompt with "Confirm Cash Received" button or "Ask Customer to Pay in App".

### C. Admin Operations Console
* **Route:** `/admin/dashboard/ride-hailing`
  * **Top Metrics:** Total Rides, Active In-Progress Rides, Gross Platform Revenue, Platform 20% Net Take, Average Platform Rating.
  * **Live Demand Analytics:** Hourly ride distribution bar chart, status breakdown pie chart, 7-day revenue trend line chart.
  * **Rides Management Table:** Filter by status (`all`, `searching`, `accepted`, `arrived`, `in-progress`, `completed`, `cancelled`), full search by Rider/Driver/Ride ID, inspect complete ride telemetry, override ride state.
  * **Drivers Fleet Table:** Filter by duty status (`online`, `offline`, `busy`, `pending_approval`), view vehicle details, license plate, driver rating, lifetime trips, total earnings, payout verification.
  * **Ride Inspection Modal:** Full route metrics, booker vs rider breakdown, payer responsibility, exact timestamps (`requestedAt`, `acceptedAt`, `arrivedAt`, `startedAt`, `completedAt`), payment transaction hash.

---

## 3. FIRESTORE DATA MODELS & SCHEMAS

### Collection: `rides/{rideId}`
```typescript
interface RideDocument {
  id: string;
  clientId: string;             // Rider account UID
  passengerId: string;          // Passenger UID or guest ID
  bookingType: 'self' | 'other';
  booker: {
    name: string;
    phone: string;
    email?: string;
  };
  passenger: {
    name: string;
    phone: string;
    email?: string;
  };
  serviceType: 'ride' | 'courier';
  category: string;             // e.g. 'AlliGo-X', 'AlliGo-Comfort', 'AlliGo-Moto'
  vehicleTier: string;
  pickup: {
    lat: number;
    lng: number;
    address: string;
  };
  destination: {
    lat: number;
    lng: number;
    address: string;
  };
  routeMetrics: {
    distanceKm: number;
    estimatedDurationMins: number;
  };
  pricing: {
    baseFare: number;
    distanceFare: number;
    timeFare: number;
    totalFare: number;          // Gross customer fare (100%)
    platformFee: number;        // Alligo commission (20%)
    driverEarnings: number;     // Driver net earnings (80%)
    commissionPercent: 20;
    currency: 'KES';
    pricingVersion: string;
    paymentMethod: 'mpesa' | 'card' | 'cash';
    paymentPhone: string;
    isPaid: boolean;
    isSettled: boolean;
  };
  payment: {
    responsibility: 'booker_now' | 'booker_after' | 'passenger_cash';
    timing: 'now' | 'after_trip';
    status: 'pending' | 'successful' | 'failed' | 'cancelled';
    method: 'mpesa' | 'card' | 'cash';
    phone: string;
    reference?: string | null;
    paidAt?: string | null;
  };
  rideStatus: 'searching' | 'accepted' | 'arrived' | 'in_progress' | 'completed' | 'cancelled';
  driverId?: string;
  driverName?: string;
  driverPhone?: string;
  driverRating?: number;
  driverVehicle?: {
    model: string;
    color: string;
    plateNumber: string;
    type: string;
  };
  driverLocation?: {
    lat: number;
    lng: number;
    heading?: number;
  };
  cancellationReason?: string;
  cancelledBy?: string;
  timestamps: {
    requestedAt: Timestamp;
    acceptedAt?: Timestamp;
    arrivedAt?: Timestamp;
    startedAt?: Timestamp;
    completedAt?: Timestamp;
    cancelledAt?: Timestamp;
  };
}
```

### Collection: `drivers/{driverId}`
```typescript
interface DriverDocument {
  id: string;                   // Firebase Auth UID
  userId: string;
  name: string;
  phone: string;
  email?: string;
  status: 'ONLINE' | 'OFFLINE' | 'ON_TRIP' | 'BUSY';
  approvalStatus: 'APPROVED' | 'PENDING' | 'REJECTED';
  isApproved: boolean;
  walletBalance: number;        // Accrued driver net earnings (KES)
  tripsCompleted: number;
  ratingScore: number;          // Out of 5.0
  totalRatingsCount: number;
  vehicle: {
    type: 'Motorbike' | 'Sedan' | 'SUV' | 'Van';
    model: string;
    color: string;
    licensePlate: string;
    year?: string;
  };
  currentLocation?: {
    lat: number;
    lng: number;
    updatedAt: Timestamp;
  };
  activeTripId?: string | null;
  payoutDetails: {
    mpesaPhone: string;
    accountName: string;
  };
}
```

---

## 4. TRIP LIFECYCLE STATE MACHINE & TRANSITION RULES

```
    [ RIDER CREATES RIDE ]
              │
              ▼
        `searching`  ◄────── Driver rejects / 15s timeout
              │
     Driver accepts job
              │
              ▼
         `accepted`   ────── Rider / Driver cancels (pre-arrival)
              │
     Driver arrives at pickup
              │
              ▼
         `arrived`    ────── Driver notifies rider
              │
     Passenger on board & start trip
              │
              ▼
       `in_progress`  ────── Live GPS tracking & route navigation
              │
     Arrival at destination & complete
              │
              ▼
        `completed`   ────── Atomic wallet credit (80%), commission deduction (20%), receipt generation
```

---

## 5. FARE CALCULATION & REVENUE FORMULA
* **Source of Truth:** `/src/lib/ride-pricing-engine.ts` & `/src/lib/courier-pricing-engine.ts`.
* **Standard Formula:**
  $$\text{Gross Fare} = \max(\text{Base Fare} + (\text{Distance Km} \times \text{Per-Km Rate}) + (\text{Duration Mins} \times \text{Per-Minute Rate}), \text{Minimum Fare})$$
* **Platform Economics Split:**
  * **Alligo Platform Take:** $20\%$ of Gross Customer Fare.
  * **Driver Net Earnings:** $80\%$ of Gross Customer Fare.
  * **Booking Fee:** Included in gross fare breakdown.

---

## 6. BUTTONS, HANDLERS & NAVIGATION AUDIT

| Component | Element / Button | ID / Selector | Action / Outcome |
|---|---|---|---|
| `/ride` | "Detect Location" | `pickup-gps-btn` | Queries Geolocation API, reverse geocodes, sets pickup pin. |
| `/ride` | "Select Vehicle Tier" | `ride-tier-[id]` | Calculates tier fare, opens "Who is Riding" selector. |
| `/ride` | "Request Ride" | `request-ride-submit-btn` | Writes new doc to `rides`, initiates dispatch radar. |
| `/ride` | "Cancel Search" | `cancel-search-btn` | Cancels ride in Firestore, resets view to route setup. |
| `/ride` | "Call Driver" | `call-driver-btn` | Opens `tel:` link with driver's verified phone number. |
| `/dashboard/driver` | "Duty Status Switch" | `duty-toggle` | Toggles driver state between `ONLINE` and `OFFLINE`. |
| `/dashboard/driver` | "Accept Ride" | `accept-ride-btn` | Invokes atomic accept transaction, assigns `driverId`, sets status `accepted`. |
| `/dashboard/driver` | "Arrived at Pickup" | `driver-arrived-btn` | Updates status to `arrived`, sends alert to rider. |
| `/dashboard/driver` | "Start Trip" | `driver-start-btn` | Updates status to `in_progress`, begins trip timer. |
| `/dashboard/driver` | "Complete Trip" | `driver-complete-btn` | Completes trip, validates payment, credits driver wallet. |
| `/dashboard/driver` | "Confirm Cash" | `confirm-cash-btn` | Marks ride `pricing.isPaid: true`, closes completion modal. |
| `/admin/dashboard` | "Mark as Completed" | `admin-complete-ride-btn` | Overrides ride status to `completed` in administrative ledger. |
| `/admin/dashboard` | "Route Optimization"| `admin-route-opt-btn` | Evaluates dispatch density and active zone coverage. |

---

## 7. VERIFICATION & QUALITY STANDARDS
1. **0 Unused Code / Dead Endpoints:** Every button, tab, modal, and route is backed by real Firestore listeners or server actions.
2. **Atomic Consistency:** Driver wallet increments and trip status updates execute safely.
3. **TypeScript Strict Compliance:** All interfaces adhere strictly to project typing with 0 ESLint or Next.js build errors.
