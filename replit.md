# MedScan - Medical Document OCR Application

## Overview

MedScan is a medical document scanning and OCR (Optical Character Recognition) web application designed primarily for Arabic-speaking users. Users can upload or photograph medical documents (lab reports, prescriptions, handwritten notes), and the app uses OpenAI's vision API to extract text from the images. Extracted scans are stored in a PostgreSQL database and can be browsed, searched, and shared via QR codes.

The app follows a monorepo structure with a React frontend (client), an Express backend (server), and shared code (schemas, routes, types) used by both.

## User Preferences

Preferred communication style: Simple, everyday language.

## System Architecture

### Directory Structure
- `client/` — React SPA frontend (Vite-bundled)
- `server/` — Express 5 backend API
- `shared/` — Shared database schemas (Drizzle ORM) and API route definitions (Zod-validated)
- `migrations/` — Drizzle-kit generated SQL migrations
- `server/replit_integrations/` — Pre-built Replit AI integration modules (audio, chat, image, batch processing). These are utility modules, not all actively used by the core app.

### Frontend Architecture
- **Framework**: React 18 with TypeScript
- **Routing**: Wouter (lightweight client-side router)
- **State/Data Fetching**: TanStack React Query for server state management
- **UI Components**: shadcn/ui (new-york style) built on Radix UI primitives
- **Styling**: Tailwind CSS with CSS variables for theming, dark theme by default
- **RTL Support**: The entire app is configured for right-to-left layout (`direction: rtl` in CSS) with Tajawal Arabic font
- **QR Codes**: `react-qr-code` for generating QR codes from extracted text
- **Build Tool**: Vite with React plugin
- **Path Aliases**: `@/` maps to `client/src/`, `@shared/` maps to `shared/`

### Key Frontend Pages
- **Home** (`/`) — File upload area with image cropping, scan statistics grid, recent scans list, feedback dialog
- **History** (`/history`) — Searchable list of all past scans
- **ScanDetail** (`/scan/:id`) — View extracted text (collapsible sections), QR code, delete with confirmation, WhatsApp sharing, export as PNG image, image zoom modal
- **NotFound** — 404 page

### Features
- **Image Cropping**: Before OCR, users can crop captured/uploaded images using react-easy-crop to select specific document regions
- **Delete Scans**: Delete with confirmation dialog, navigates back to home after deletion
- **WhatsApp Sharing**: Share extracted text via wa.me URL (max 2000 chars)
- **Export as PNG**: Uses html2canvas to export results section as downloadable PNG image
- **Image Zoom**: Full-screen zoom modal with pinch/scroll zoom controls and pan support
- **Dark/Light Theme**: Toggle via useTheme hook, persisted in localStorage, CSS variables for both themes
- **Scan Statistics**: Grid showing total scans, diagnoses, tests, and medications counts
- **Feedback System**: Dialog with name/email/message, sends via SendGrid to developer email
- **QR Code Sharing**: Generate QR codes from extracted text for easy sharing

### Backend Architecture
- **Runtime**: Node.js with Express 5
- **Language**: TypeScript, executed via `tsx` in development
- **API Pattern**: RESTful JSON API under `/api/` prefix
- **AI Integration**: OpenAI API (via Replit AI Integrations) for vision-based OCR — sends medical document images to GPT for text extraction
- **Request Limits**: 50MB body limit for base64 image uploads
- **Static Serving**: In production, serves the Vite-built frontend from `dist/public`
- **Dev Server**: Vite dev server middleware with HMR in development mode

### API Routes
| Method | Path | Purpose |
|--------|------|---------|
| POST | `/api/scans` | Upload image, extract text via OpenAI, save scan |
| GET | `/api/scans` | List all scans (newest first) |
| GET | `/api/scans/:id` | Get a single scan by ID |
| DELETE | `/api/scans/:id` | Delete a scan by ID |
| POST | `/api/feedback` | Send feedback email via SendGrid |

The route contracts are defined in `shared/routes.ts` using Zod schemas for type-safe validation.

### Database
- **Database**: PostgreSQL (required, provisioned via Replit)
- **ORM**: Drizzle ORM with `drizzle-zod` for schema-to-Zod integration
- **Schema Location**: `shared/schema.ts` (main app tables), `shared/models/chat.ts` (chat integration tables)
- **Migration Tool**: `drizzle-kit push` for schema synchronization
- **Connection**: `node-postgres` Pool via `DATABASE_URL` environment variable

### Database Tables
- **scans** — `id` (UUID, auto-generated), `image_url` (text), `extracted_text` (text, nullable), `created_at` (timestamp)
- **conversations** — For chat integration (serial id, title, created_at)
- **messages** — For chat integration (serial id, conversation_id FK, role, content, created_at)

### Storage Pattern
The `server/storage.ts` file implements a `DatabaseStorage` class conforming to an `IStorage` interface, providing a clean abstraction over database operations.

### Build Process
- **Development**: `tsx server/index.ts` with Vite middleware for HMR
- **Production Build**: Two-step — Vite builds the client to `dist/public`, esbuild bundles the server to `dist/index.cjs`
- **Server bundling**: Select dependencies are bundled (allowlisted) to reduce cold start times; others are externalized

### Replit Integration Modules (in `server/replit_integrations/`)
Pre-built utility modules for AI features. Not all are actively used by the core scan flow:
- **audio/** — Voice recording, playback, speech-to-text, text-to-speech
- **chat/** — Conversation management with OpenAI streaming
- **image/** — Image generation via `gpt-image-1`
- **batch/** — Rate-limited batch processing with retries

## External Dependencies

### Required Environment Variables
- `DATABASE_URL` — PostgreSQL connection string (must be provisioned)
- `AI_INTEGRATIONS_OPENAI_API_KEY` — OpenAI API key (via Replit AI Integrations)
- `AI_INTEGRATIONS_OPENAI_BASE_URL` — OpenAI base URL (via Replit AI Integrations)

### Third-Party Services
- **OpenAI API** — Used for vision-based OCR text extraction from medical document images (model: `gpt-5.2` for chat completions with image input)
- **PostgreSQL** — Primary data store, connected via `node-postgres`
- **Google Fonts** — Tajawal (Arabic font), DM Sans, Fira Code, Geist Mono

### Key NPM Dependencies
- **Frontend**: React, Wouter, TanStack React Query, shadcn/ui (Radix UI), Tailwind CSS, react-qr-code, react-easy-crop, html2canvas, date-fns, lucide-react, framer-motion
- **Backend**: Express 5, OpenAI SDK, Drizzle ORM, node-postgres, connect-pg-simple
- **Shared**: Zod, drizzle-zod
- **Build**: Vite, esbuild, tsx