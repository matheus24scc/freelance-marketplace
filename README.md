# Freelance Marketplace

A full-stack freelance marketplace monorepo (client + server) for connecting clients and
freelancers through gigs, orders, payments, real-time messaging and notifications.

## Repository Layout

```
freelance-marketplace/
├── client/                # Next.js 16 + React 19 frontend
└── server/                # NestJS 10 backend
```

| Service  | Path      | Stack                                              |
|----------|-----------|----------------------------------------------------|
| Frontend | `client/` | Next.js 16, React 19, TailwindCSS, Zustand         |
| Backend  | `server/` | NestJS 10, TypeORM, Prisma, Stripe, Socket.io      |

The backend exposes a REST API plus a Socket.io gateway for live chat and notifications.

## Features

- **Authentication** — JWT (passport-jwt) with bcrypt-hashed passwords and role-based access
  (clients vs freelancers)
- **Gigs** — Freelancers publish service offerings with pricing, contract type, delivery
  window, and skill tags
- **Orders** — Clients create orders from a gig (or directly); status lifecycle:
  `PENDING → IN_PROGRESS → COMPLETED | CANCELLED`
- **Payments** — Stripe integration with `Payment` records tied to an order and a user
- **Messaging** — Real-time chat between two users via Socket.io
- **Notifications** — Per-user notification feed with a `read` flag
- **Security** — `helmet`, `express-rate-limit`, `cookie-parser`, configurable CORS

## Prerequisites

- Node.js 20+
- npm or yarn (workspaces not required; install each side separately)
- PostgreSQL 15+
- Stripe API keys (test or live)

## Quick Start

### 1. Backend

```bash
cd server
cp .env.example .env             # then edit DATABASE_URL, JWT_SECRET, STRIPE_SECRET_KEY
npm install
npx prisma generate
npx prisma migrate deploy
npm run start:dev                # http://localhost:3000
```

### 2. Frontend

```bash
cd client
npm install
npm run dev                      # http://localhost:3001
```

The frontend points at the backend API through the proxy in `next.config.ts`.

## Database Schema

The Prisma schema (`server/prisma/schema.prisma`) defines:

- `User` (CLIENT | FREELANCER)
- `Gig` (with `ContractType`: FIXED | HOURLY, deliveryDays, skills[])
- `Order` (OrderStatus: PENDING | IN_PROGRESS | COMPLETED | CANCELLED)
- `Payment` (Stripe id, amount, order, user)
- `Message` (sender, receiver, content, read flag)
- `Notification` (user, title, body, read flag)

TypeORM entities mirror the same domain in `server/src/**/*.entity.ts`.

## API Surface

| Path             | Method | Description                |
|------------------|--------|----------------------------|
| `/auth/register` | POST   | Create account             |
| `/auth/login`    | POST   | Exchange credentials for JWT |
| `/users`         | CRUD   | Manage users (auth)        |
| `/gigs`          | CRUD   | Manage gigs                |
| `/orders`        | CRUD   | Manage orders              |
| `/payments`      | CRUD   | Stripe-backed payments     |
| `/messages`      | WS     | Real-time chat via Socket.io |
| `/notifications` | GET    | List notifications         |

The Socket.io gateway at `GatewayModule` powers `/socket.io` with rooms bound to user ids.

## Build

```bash
# backend
cd server && npm run build && npm run start:prod

# frontend
cd client && npm run build && npm start
```

## Testing

```bash
# backend unit tests
cd server && npm test

# backend e2e (requires configured DB and JWT_SECRET)
cd server && npm run test:e2e
```

## Tech Stack

- **Frontend** — Next.js 16, React 19, TypeScript, TailwindCSS v4, Zustand
- **Backend** — NestJS 10, TypeORM 0.3, Prisma 5, Passport (JWT/local), Stripe SDK,
  Socket.io server, Helmet, express-rate-limit
- **Data** — PostgreSQL 15, Prisma migrations (with TypeORM entity mirror)

## License

MIT
