# Charted Waters

Charter booking software for independent operators across the Bahamas, Caribbean, and United States. Built for bonefishing guides, boat charter operators, and marina activity desks.

**[chartedwaters.app](https://chartedwaters.app)**

---

## Features

### For Guests
- Browse and search a directory of verified charter operators by location, charter type, and rating
- View operator profiles with trip listings, fleet details, photo galleries, and guest reviews
- Book trips without creating an account — guest-authenticated via email only
- Live marine conditions widget on every operator profile
- Self-service trip cancellation with automatic refund calculation based on the operator's cancellation policy
- Hurricane and weather refund protection via operator-configured blackout policies

### For Operators
- **Listings** — Create and manage bookable trips with pricing, duration, guest capacity, included amenities, and safety equipment details
- **Availability** — Block dates, set custom pricing windows, and manage a booking calendar
- **Fleet** — Showcase vessels with specs and photos
- **Bookings** — Full booking management from inquiry through confirmation, with inline guest communication
- **Guests** — Guest directory automatically built from booking history
- **Payouts** — Track completed booking payouts; auto-send via Stripe Connect, PayPal Payouts, or Wise, or receive manual bank wire instructions
- **Credentials** — Upload required documents (Captain's License, Insurance, Government ID, Boat Safety Certificate, Nautical Tourism Permit) for verified operator status
- **Settings** — Manage profile, location, payout method, and cancellation policy

### For Admins
- Operator approval and management
- Credential document review with per-document approve/reject and automated email notifications
- Review moderation
- Payout processing with one-click auto-send for PayPal and Wise operators
- Revenue dashboard
- Waitlist and inquiry management

---

## Tech Stack

| Layer | Technology |
|---|---|
| Framework | Next.js 14 App Router + React Server Components |
| Database | Supabase Postgres — multi-tenant via RLS, `operator_id` on every tenant-scoped table |
| Auth | Clerk — org-based roles, `sessionClaims.metadata.role` |
| Payments | Stripe Connect (Express accounts) — marketplace model with platform fee |
| Cache | Upstash Redis — availability holds (10-min TTL), tide data (24h TTL), rate limiting |
| Background jobs | Inngest — booking events, stale booking expiry, hurricane alerts, weather polling |
| Email | Resend — transactional templates for bookings, credentials, payouts, and cancellations |
| Styling | Tailwind CSS — Signal & Slate design system |
| Image uploads | Supabase Storage |
| SEO | ISR via `generateStaticParams`, JSON-LD (`LocalBusiness`, `TouristDestination`, `AggregateRating`), dynamic sitemap |
| Analytics | Google Analytics 4 (full booking funnel) + Facebook Pixel |

---

## Booking Flow

```
inquiry → pending → hold (Redis, 10 min) → confirmed → completed
                                                      ↘ cancelled / refunded
```

Stale `pending` and `hold` bookings older than 15 minutes are auto-expired by an Inngest cron job, which also voids the associated Stripe PaymentIntent.

---

## Payout Modes

Operators without a Stripe Connect account are paid via a configurable payout method:

| Method | Behaviour |
|---|---|
| Stripe Connect | Direct transfer at confirmation |
| PayPal | Auto-sent via PayPal Payouts API |
| Wise | Auto-sent via Wise Payouts API |
| Bank wire | Manual — details shown to admin |

---

## Local Development

```bash
# Install dependencies
npm install

# Start local Supabase stack (port 54321)
npx supabase start

# Start Inngest dev server (port 8288)
npx inngest-cli@latest dev

# Start Next.js dev server (port 3000)
npm run dev
```

Copy `.env.example` to `.env.local` and fill in the required keys before starting.

### Key Commands

| Command | Description |
|---|---|
| `npm run dev` | Start dev server on port 3000 |
| `npm run build` | Production build |
| `npm run typecheck` | TypeScript check (no emit) |
| `npm run lint` | ESLint |
| `npm run db:generate` | Regenerate Supabase types from local schema |

---

## Environment Variables

See `.env.example` for the full list. Key variables:

- `NEXT_PUBLIC_SUPABASE_URL` / `NEXT_PUBLIC_SUPABASE_ANON_KEY` / `SUPABASE_SERVICE_ROLE_KEY`
- `NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY` / `CLERK_SECRET_KEY`
- `STRIPE_SECRET_KEY` / `NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY` / `STRIPE_WEBHOOK_SECRET`
- `UPSTASH_REDIS_REST_URL` / `UPSTASH_REDIS_REST_TOKEN`
- `RESEND_API_KEY`
- `INNGEST_EVENT_KEY` / `INNGEST_SIGNING_KEY`
- `PAYPAL_CLIENT_ID` / `PAYPAL_CLIENT_SECRET`
- `WISE_API_KEY` / `WISE_PROFILE_ID`
