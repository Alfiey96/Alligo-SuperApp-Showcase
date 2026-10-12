# Alligo SuperApp

### One Platform. Endless Possibilities.

**Alligo — All In One Go** is a multi-service SuperApp concept designed to bring shopping, professional services, food and parcel delivery, and on-demand mobility together in one unified digital ecosystem.

Explore the live application: **[alligo-super-app.vercel.app](https://alligo-super-app.vercel.app/)**

[Portfolio](https://alfieydigitalhq.vercel.app/) · [GitHub Profile](https://github.com/Alfiey96)

---

## Overview

Alligo aims to simplify everyday digital commerce by connecting customers, merchants, freelancers, service providers, drivers, and delivery partners through a unified platform.

Instead of switching between separate applications for different services, users can access multiple service categories through one application experience.

The platform combines a multi-service interface with account-based experiences, online service workflows, mapping capabilities, and payment integrations.

## Explore the Alligo Ecosystem

### 1. Alligo Shop — Marketplace & Retail

Discover products from local sellers, brands, and independent stores.

- Fashion and accessories
- Electronics
- Home and office products
- Supermarket and grocery categories
- Marketplace browsing and purchasing workflows

**Explore:** [Alligo Shop](https://alligo-super-app.vercel.app/shop)

### 2. Hire a Pro — Professional Services

Connect customers with professionals and independent service providers for projects, specialist work, and everyday services.

- Software and IT
- Design and creative services
- Marketing and communications
- Business consulting
- Transport and logistics services

**Explore:** [Hire a Pro](https://alligo-super-app.vercel.app/hire-a-pro)

### 3. Alligo Delivery — Food, Groceries & Couriers

Bring food ordering, grocery shopping, and parcel delivery into one service experience.

- Restaurant and meal ordering
- Grocery and supermarket delivery
- Courier and parcel dispatch
- Delivery coordination and tracking workflows

**Explore:** [Alligo Delivery](https://alligo-super-app.vercel.app/delivery)

### 4. Alligo Ride — City Mobility & Chauffeurs

A mobility experience for customers seeking convenient transport and ride-booking services.

- Ride requests and trip workflows
- Pickup and destination selection
- Map-based route visualisation
- Driver-facing application workflows
- Chauffeur and transport services

**Explore:** [Book a Ride](https://alligo-super-app.vercel.app/ride)

---

## How Alligo Works

The intended service journey follows four stages:

1. **Discover** — Browse products, services, delivery options, and rides.
2. **Connect** — Communicate with relevant merchants, professionals, or service providers.
3. **Order or Hire** — Initiate purchases, service requests, deliveries, or rides.
4. **Track and Complete** — Follow the relevant workflow through fulfilment and completion.

The exact functionality available to users depends on the implementation and configuration of each module.

## Technology Stack

Alligo is built around a modern web application architecture with mobile application support.

| Area | Technologies |
|---|---|
| Application framework | Next.js, React |
| Programming language | TypeScript |
| User interface | Tailwind CSS, Radix UI, Lucide React |
| Backend and API layer | Next.js API routes and server-side application logic |
| Authentication and database | Firebase Authentication, Cloud Firestore, Firebase Admin SDK |
| Mapping and routing | MapLibre GL JS, OpenStreetMap-based mapping, OSRM |
| Real-time functionality | Pusher, Stream Video |
| AI integrations | Google GenAI, Genkit |
| Payments | Paystack integration |
| Media management | Cloudinary |
| Validation and forms | Zod, React Hook Form |
| Mobile application | Capacitor with Android support |
| Deployment | Vercel |

*This is a summary of the project's technology stack, not a claim that every integration is active on every deployed route. Exact package versions and current implementation status should be verified against the repository before publication.*

## Architecture Overview

Alligo follows a modular application approach, separating the user interface, service workflows, server-side operations, and external integrations.

```text
Alligo SuperApp
│
├── User Interface
│   ├── Shared Components
│   ├── Navigation and Layouts
│   └── Responsive Experiences
│
├── Service Modules
│   ├── Alligo Shop
│   ├── Hire a Pro
│   ├── Alligo Delivery
│   └── Alligo Ride
│
├── Application Services
│   ├── Authentication and Roles
│   ├── API Routes and Validation
│   ├── Marketplace and Order Workflows
│   ├── Mapping and Routing
│   └── Notifications and Real-Time Features
│
├── External Integrations
│   ├── Firebase
│   ├── Paystack
│   ├── Mapping and Routing Services
│   ├── Media Services
│   └── AI Services
│
└── Deployment
    ├── Web Application
    └── Android Application Support
```

This diagram represents the high-level architecture. It does not imply that every module is fully production-ready or that every external service is enabled in the live environment.

## Security and Trust

Trust and transaction protection are important parts of Alligo's product direction.

The platform is designed around concepts including:

- Role-based experiences for different user categories.
- Authentication and access-control mechanisms.
- Payment integration and transaction workflows.
- Validation of application inputs.
- Secure handling of server-side credentials.
- Provider and merchant verification workflows.

**Escrow, identity verification, payment settlement, dispute resolution, and support guarantees must be confirmed against the implemented workflows before being described as fully operational guarantees.** Payment processing alone does not establish that a complete escrow system is implemented.

## Getting Started

This showcase repository is intended to document selected, sanitised parts of the Alligo project. Setup instructions apply only to the source code and modules actually included in this repository.

### Prerequisites

- Node.js and npm, using versions compatible with the included project configuration.
- Git.
- Access to any external services required by the specific module being tested.

### Installation

```bash
git clone https://github.com/Alfiey96/Alligo-SuperApp-Showcase.git
cd Alligo-SuperApp-Showcase
npm install
```

The commands above assume that the public repository contains a compatible `package.json`. If the repository is documentation-only initially, there will be no application to install until the selected source modules and their required project configuration are added.

### Environment Configuration

Where a module requires external services, create a local environment file based on the documented variable names and the example template supplied with that module.

For example:

```dotenv
# Example placeholders only.
# Replace these with the exact variables required by the included code.

NEXT_PUBLIC_FIREBASE_API_KEY=
NEXT_PUBLIC_FIREBASE_AUTH_DOMAIN=
NEXT_PUBLIC_FIREBASE_PROJECT_ID=

PAYSTACK_SECRET_KEY=
GOOGLE_GENAI_API_KEY=
```

These names are illustrative and must be reconciled with the actual source code before use. Never commit real API keys, service-account files, production credentials, or private environment files.

### Running and Testing

The available commands depend on which application files are included. Once the relevant `package.json` and source modules are published, document and verify the applicable commands, for example:

```bash
npm run dev
npm run typecheck
npm run lint
npm test
npm run build
```

These are conventional examples, not a claim that every command currently exists or passes. Use only scripts that are defined in the repository and record the actual test results.

## Screenshots and Demonstrations

The live application can be explored here:

**[Open Alligo SuperApp](https://alligo-super-app.vercel.app/)**

Screenshots of the main application and its Shop, Hire a Pro, Delivery, and Ride experiences can be added to a `docs/screenshots/` directory as they are captured and reviewed.

Screenshots should represent the actual deployed application and must not expose private customer information, credentials, or internal administration data.

## Project Status

Alligo is an evolving multi-service application. This repository is intended to showcase selected implementation details, architecture, and engineering work in a form suitable for public review.

The public demonstration and individual modules may have different levels of functionality. Features should be described according to what has been implemented and verified.

## About the Developer

Alligo is developed by **Alfred Musyoka**, founder of AlfieyTech, with a focus on full-stack application development, systems integration, and digital product engineering.

- **Portfolio:** [alfieydigitalhq.vercel.app](https://alfieydigitalhq.vercel.app/)
- **GitHub:** [github.com/Alfiey96](https://github.com/Alfiey96)
- **Live application:** [alligo-super-app.vercel.app](https://alligo-super-app.vercel.app/)

## License

The licensing terms for this showcase and its included source code should be specified before public distribution. Do not assume the private production repository's code is open source simply because a public showcase repository exists.
