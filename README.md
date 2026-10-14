# Hotel Tulip Inn — Full-stack booking platform

A production-oriented Express + SQLite hotel website with live availability, booking requests, contact inbox, admin authentication, content verification and link extraction.

## Verified seed data

The initial public dataset was source-checked on 15 October 2026. Public values are stored with `verified=1` and a source URL. Prices are deliberately `NULL` until hotel management approves them.

- Google Maps place supplied by the owner: business identity and location
- Google Hotels: phone, address, check-in/out and listed facilities
- Facebook business page: phone, email and page link
- Booking/property listings: room categories (Double, Twin, Triple, Quadruple)
- Current room imagery from active property listing search results

No review quotes, ratings or live third-party prices are published because those values change and/or conflict across sources.

## Local setup

```bash
cp .env.example .env
# Replace JWT_SECRET, ADMIN_EMAIL and ADMIN_PASSWORD
npm install
npm start
```

Open `http://localhost:3000`. Admin is at `/#admin`.

For a local demonstration only, when no `.env` exists the seeded login is `admin@tulipinn.local` / `ChangeMe123!`. Never use those defaults in production. The admin user is created only on first database initialization; changing environment variables later does not change an existing account.

## What works

- Mobile-responsive multi-page SPA experience (Home, About, Rooms, Gallery, Facilities, Location, Contact, FAQ)
- Date/occupancy availability queries with overlap-safe inventory checks
- Direct booking requests with unique reference numbers and pending/confirmed/cancelled/completed states
- Price calculation only when an administrator has approved a nightly rate
- Contact form stored in the admin inbox
- HTTP-only JWT session cookie, bcrypt password hashing, Helmet headers, rate limits and input sanitization
- Room creation, inventory, pricing, publish state, bookings and source-tracked business content management
- Image-library uploads (JPEG/PNG/WebP, 5 MB limit), approval controls, rights-source records, ordering and deletion
- Date-level room blocking for maintenance, offline bookings and temporary closures; blocks feed live availability
- URL extraction queue with SSRF protection: public links are fetched server-side; candidate data is queued and never auto-published
- Optional OpenAI structured extraction when `OPENAI_API_KEY` is configured; deterministic extraction remains available without it
- Privacy, website terms and cancellation-information drafts, clearly marked for management/legal approval
- SQLite WAL database with foreign keys
- Hotel JSON-LD, semantic metadata, accessible labels and lazy-loaded images

## Production deployment

### Render / Railway / Fly.io

1. Push the project to a private Git repository.
2. Create a Node 20+ service.
3. Build command: `npm ci`
4. Start command: `npm start`
5. Add all environment variables from `.env.example`.
6. Attach a persistent volume at `/app/data` (Railway) or your platform’s equivalent, then set `DATA_DIR=/app/data`. Both SQLite and administrator-uploaded images are stored there and must not live on an ephemeral filesystem.
7. Set `NODE_ENV=production` and use HTTPS (platform proxy is fine).
8. Back up `data/hotel.db` on a schedule.

For multi-instance/high-traffic deployment, migrate SQLite tables to managed PostgreSQL and use database-backed sessions or a shared Redis session store.

## External APIs and credentials

The core booking and admin application runs without paid APIs. These integrations are recommended for launch:

- **Google Places API** (`GOOGLE_PLACES_API_KEY`): compliant ongoing place-detail refresh. Do not scrape Google Maps in production. Route extracted changes into the approval queue.
- **OpenAI API** (`OPENAI_API_KEY`): optional structured extraction from difficult pages. Require strict JSON output and preserve source snippets. The included deterministic extractor works without it.
- **Resend / Postmark / SMTP** (`RESEND_API_KEY`, `EMAIL_FROM`): booking and inquiry notifications. Database persistence already works; email delivery is not enabled in this demo.
- **Cloudinary / S3**: recommended for production image upload, optimization and backups. Current approved images are local assets.
- **Stripe or local payment gateway**: not included because the owner did not provide merchant credentials or payment/cancellation policy. No payment should be collected before those are approved.

## Launch checklist

- Confirm legal ownership/permission for every imported image.
- Have hotel management approve room counts, prices, descriptions, facilities, policies and brand narrative.
- Replace the seeded admin and secret; delete the existing development database before first production boot.
- Configure transactional email and test booking notifications.
- Add privacy, cancellation and terms pages approved for Pakistan.
- Run an accessibility and security review; place the app behind HTTPS and a managed WAF.
- Verify Google Business data through the official Places API and schedule source re-checks.
