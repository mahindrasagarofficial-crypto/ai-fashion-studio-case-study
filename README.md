# AI Fashion Studio: AI On-Model Photoshoots for Indian Garment Sellers (Case Study)

> Upload one photo of a garment and get professional on-model catalogue images, ready to share with buyers. Built for small Indian garment sellers who can't afford models, studios or photographers.

**Status:** Working end to end in live browser testing · **Role:** Founder, product owner & technical director · **Code:** private

---

## Why I built it

I ran a ladies' garment manufacturing business for over 12 years, supplying 55+ retail shops and dozens of B2B buyers. Every new design needed photos before anyone would order, and a proper photoshoot (model, studio, photographer) cost more than many small sellers earn on the design itself. So most sellers send buyers flat, badly lit phone photos, and lose orders.

AI Fashion Studio turns a single garment photo into on-model catalogue images in minutes, at a fraction of the cost.

## How it works

```
Garment photo ─▶ Quality check ─▶ Route to best AI model ─▶ Generate on-model images
      ─▶ Automatic quality check ─▶ Package ─▶ Deliver to catalogue + public share page
```

The seller's flow:
1. **Log in with a phone OTP.** No passwords; the UI is Hindi-first.
2. **Upload a garment photo.** It goes straight to storage using a secure presigned upload.
3. **Generate a photoshoot.** On-model images are created, quality-checked, then previewed.
4. **Accept or reject.** Credits are only spent on images the seller **accepts**.
5. **Edit:** remove the background, change pose or brightness, reframe for square, story or portrait.
6. **Share** a public catalogue page with buyers on WhatsApp.

## Architecture

A TypeScript monorepo (pnpm + Turborepo) with a clean, hexagonal design: domain logic stays separate from the vendors it talks to, so any vendor can be swapped through configuration.

| Layer | Technology |
|---|---|
| App | Next.js 14 PWA (React 18), installable, with an honest offline page |
| API | NestJS on Fastify, JWT access and refresh tokens |
| Background jobs | BullMQ + Redis worker running the generation pipeline |
| Database | PostgreSQL, SQL migrations, **Row Level Security** for multi-tenancy |
| Storage | S3-compatible (MinIO locally; designed for Supabase, R2 or S3) |
| AI providers | FASHN (primary), Gemini (secondary), behind a provider port with automatic fallback and cost tracking |
| Payments | Razorpay and Cashfree behind a payment port with a config switch |
| Messaging | WhatsApp Business provider and SMS fallback behind a messaging port |
| Tests & local dev | Vitest; Docker Compose (Postgres, Redis, MinIO) |

## Engineering highlights

- **Pay only for what you keep.** A credit wallet charges on *accept*, not on generate, so sellers never pay for bad images.
- **An idempotent pipeline.** Each generation step (intake QC, route, generate, auto-QC, package, deliver) can be retried safely without double-charging or duplicate images.
- **A quality evaluation system ("Golden Set").** Versioned test batches, evaluation runs, scores and human reviews, so AI quality is measured rather than guessed when switching or tuning providers.
- **Security built in:** webhook HMAC verification on the raw request bytes, tenant isolation with RLS, a fail-closed admin area, and a privacy-oriented data export and delete design.

## Real bugs found and fixed during live testing

- Background job IDs containing `:` were rejected by Redis
- The worker silently did nothing when a database setting was missing
- The AI provider was being sent `localhost` image URLs it couldn't reach (fixed by sending the image data directly)
- The provider fallback chain hid every error except the last one
- A webhook with a bad signature returned **500** instead of **401**
- A race condition between *reject* and *regenerate*
- The public catalogue page clashed with forced Row Level Security (fixed with a narrow, read-only policy)

## How it was built

I wrote the product requirements, UX principles and architecture decisions (tracked as ADRs in a separate planning repo with a PRD, UX guide, data design and sprint plans). I then built the product with **AI pair-programming (Cursor)**, reviewing every architecture decision and running the live end-to-end tests myself.

---

**Want something similar?** I build AI-powered products and automations for small businesses.
[Upwork](https://www.upwork.com/freelancers/~01c726d9217f6052ca) · [LinkedIn](https://www.linkedin.com/in/sagar-m-902965416) · mahindrasagarofficial@gmail.com
