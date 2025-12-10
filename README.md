<p align="center"><img src="/public/logo.png" alt="ProtectMyDesign logo" width="220"/></p>

# ProtectMyDesign

> **A small, focused tool to share watermarked design previews.**
> Upload originals, auto-generate low-res watermarked previews using the project's logo, share passwordless signed preview links, and get view logs. Built as a lean open-source Next.js app — opinionated, minimal, and easy to run.

---

## TL;DR

ProtectMyDesign is a lightweight preview-and-watermark utility for designers and freelancers. The app automatically generates low-resolution preview derivatives from your originals, composites the designer’s brand logo as a watermark (or uses a default), and issues short-lived, signed preview links that require no client login. Originals remain private and are delivered only after the designer authorizes.

This repository is set up with:

* Next.js v16 (app router) + TypeScript
* React 19 with the React compiler enabled
* Turbopack dev server
* Tailwind CSS
* pnpm as package manager

---

## Key Features

* Folder/file uploads (originals stored privately)
* Low-res preview generation (resize + recompress)
* Auto watermark with designer logo (tiled or smart placement)
* Signed, short-lived preview links (no client signup required)
* Canvas-based preview rendering (reduces trivial save-as)
* View logging (timestamps, IP/User-Agent)
* Link revocation
* Minimal, intentional UX for fast adoption

---

## Who this is for

* Freelance designers and photographers who need a fast way to show drafts without handing over originals.
* Small agencies that want a lightweight, branded preview link flow.
* Developers who want a minimal, self-hosted proofing layer.

---

## Tech Stack

* Next.js (React framework)
* TypeScript (static typing)
* Drizzle (ORM)
* Supabase PostgreSQL (database)
* AWS S3/R2 (object storage)
* Sharp (image processing)
* Clerk (authentication)
* React Query (data fetching)
* Tailwind CSS (styling)
* ShadCN UI (UI components)
* Vercel (deployment)

---

## Project structure

```
/ (repo root)
├─ public/
│  ├─ logo.png                 # logo used in README / landing
│  └─ ...                      # static assets
├─ src/
│  ├─ app/                     # Next.js app routes + pages
│  │  ├─ api/                  # Next.js API route handlers (server)
│  │  ├─ preview/              # preview page route (GET /preview?t=...)
│  │  └─ ...
│  ├─ components/              # React components (PreviewCanvas, UploadForm, etc.)
│  ├─ lib/                     # shared library code (storage, auth, tokens)
│  │  ├─ storage.ts            # S3/R2 helpers
│  │  ├─ watermark.ts          # Sharp watermark generation pipeline
│  │  └─ tokens.ts             # sign/verify preview tokens
│  ├─ services/                # background jobs, queuing, workers
│  ├─ styles/                  # Tailwind globals + utilities
│  └─ types/                   # TypeScript types and interfaces
├─ drizzle/
├─ .env.example                # example env vars required
├─ .gitignore
├─ biome.json
├─ next-env.d.ts
├─ next.config.ts
├─ package.json
├─ pnpm-lock.yaml
├─ postcss.config.mjs
├─ tsconfig.json
└─ README.md
```

Notes:

* The `lib/watermark.ts` file is the single source of truth for how previews are generated using Sharp. Keep the logic minimal: resize -> composite tiled logo -> compress.
* The `src/app/api` folder contains the endpoints required by the MVP: `/upload`, `/generate-preview`, `/create-preview-link`, `/preview` and `/preview/image`.

---

## Getting started (developer)

### Prerequisites

* Node.js (v20+ recommended)
* pnpm installed globally
* An S3-compatible storage (AWS S3, Cloudflare R2, DigitalOcean Spaces) or local mock for dev
* A Postgres DB (or SQLite for local dev)
* Optional: Redis (for background jobs/queue)

### Install

```bash
pnpm install
```

### Environment

Copy `.env.example` to `.env` and fill values.

`.env.example` (minimal)

```
NEXT_PUBLIC_BASE_URL=http://localhost:3000
STORAGE_PROVIDER=s3
S3_BUCKET=my-protectmydesign-bucket
S3_REGION=ap-southeast-1
S3_ACCESS_KEY_ID=xxx
S3_SECRET_ACCESS_KEY=xxx
DATABASE_URL=postgres://user:pass@localhost:5432/protectmydesign
SIGNING_SECRET=replace-with-secure-secret
PREVIEW_DEFAULT_EXPIRES_SECONDS=604800  # default 7 days
```

### Run (dev)

This repo is configured to use Turbopack as the dev server for fast refresh. Use the `dev` script.

```bash
pnpm dev
```

### Build

```bash
pnpm build
pnpm start
```

---

## API overview (MVP)

* `POST /api/upload` - upload original file(s). Returns assetId and kicks off preview generation job.
* `POST /api/generate-preview` - (internal) generate a watermarked low-res preview for an asset.
* `POST /api/create-preview-link` - create a short-lived signed preview link (returns full preview URL).
* `GET /preview?t=SIGNED_TOKEN` - public preview page that loads the watermarked preview image via a proxied endpoint.
* `GET /api/preview/image?previewId=...&sig=...` - proxied image bytes (no direct storage URL exposure).
* `POST /api/preview/revoke` - revoke a preview link (designer only).

> Authentication: Designer management uses passwordless email magic links or management tokens (designer-only). Clients require no login.

---

## Preview generation notes (important)

* Previews must be generated from the original as a new derivative. Never down-sample and overwrite the original.
* Watermarking strategy for the MVP: tiled PNG logo overlay with configurable opacity and spacing. This is a good balance between deterrence and aesthetics.
* Use `Sharp` (Node) for image processing. Keep the pipeline atomic and idempotent (generate the same preview given same inputs).
* Cache control: returned preview responses should set `Cache-Control: no-store` to avoid long-lived caches of derivative assets.

---

## Security considerations

* Always serve previews over HTTPS.
* Signed tokens must use a secure secret and be short-lived. Do not accept unsigned requests for preview assets.
* Store originals with private ACLs. Only allow server-side code to fetch originals.
* Rate-limit preview endpoints and detect automated scraping.
* Keep logs for accountability (timestamp, IP, UA). If you ever offer public forensic claims, expand this to include sha256 hashes and auditable anchors.

---

## Deployment notes

* Serverless platforms (Vercel, Cloudflare Pages + Workers) work for the front-end and serverless API routes, but you will likely need a worker or server process to handle Sharp pipeline jobs (Edge functions often have image processing limits). Consider a small worker (Fly, Railway, Render) or background job on a simple EC2/VM for processing.
* Use object storage (R2) for originals and previews. Keep originals private and generate previews on upload.
* If using Vercel, host static front-end there and run a lightweight processing worker elsewhere (or use serverless function with large memory / timeout supported).

---

## Roadmap (lean, prioritized)

1. MVP: upload -> auto-preview generation -> create signed share link -> view logs -> revoke link
2. UX polish: folder upload, bulk link creation, email notifications
3. Integrations: Dropbox / Google Drive import
4. Forensics (paid): invisible watermarking, SHA anchoring, optional blockchain anchoring
5. White-label / custom domain for agencies

---

## Contributing

This repo is intentionally small and open-source. Pull requests welcome for bug fixes, docs, and small features. For major features (e.g., integrations or a new watermarking pipeline), open an issue first describing the proposal.

---

## License

MIT — do whatever, but don’t pretend you made it if you didn’t.

---

## Attribution

ProtectMyDesign — by **Izier**
