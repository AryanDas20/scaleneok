# Scaleneok

**Scaleneok — Resume vs Job Description Gap Analyzer** helps students compare a resume with a target role, inspect matched and missing skills with source evidence, and create a practical 14-day learning plan for the top three gaps.

This repository is a runnable full-stack project scaffold with a working deterministic skill-matching engine, preview analysis mode, Firebase Authentication/Firestore integration points, Supabase private-file storage integration, and a responsive web interface.

## What is included

### Working in this repository

- Responsive public landing page and authenticated workspace shell.
- Firebase email/password sign-up and login, Google sign-in, password reset, verification email attempt, session observation and logout.
- Explicit preview workspace when cloud credentials are not configured.
- Resume/job-description text analysis in preview mode through the Node.js API.
- Canonical skill extraction and alias normalization, including ReactJS/React.js, NodeJS/Node.js, Postgres/PostgreSQL and Mongo DB/MongoDB.
- Separate matched, missing and partial evidence categories.
- Source line references for missing skills and resume evidence for matched skills.
- Weighted skill coverage with visible formula.
- Personalized 14-day plan builder for up to three gaps.
- Learning progress tracking in the browser.
- Preview-mode application tracker; Firebase-backed application, profile and learning-progress endpoints for configured accounts.
- Global search, skill library, role-aware question prompts, and basic report export as TXT.
- Node.js API with Firebase ID-token verification middleware.
- Supabase server-side storage adapter for private uploads, signed download URLs and deletion.
- PDF text extraction with coordinate grouping and a basic multi-column heuristic; DOCX and TXT extraction.
- Optional OCR.Space fallback for scanned/image-only PDFs.
- Skill-matching regression tests and an evaluation metric helper.

### Not yet complete / needs external configuration or further implementation

- Cloud login and storage cannot work until you create your own Firebase and Supabase projects and add credentials.
- OCR fallback requires an OCR.Space API key. Scanned document text is not extracted without it; the UI/API must provide the manual text recovery path.
- Multi-column extraction uses a coordinate-based heuristic. It needs a wider real-resume test corpus before being described as reliable for every layout.
- The current matcher and question prompts are deterministic; no AI-provider key is required, and no runtime Agent37 API is assumed.
- In preview mode, application tracking, self-reported skills and learning progress persist in the browser; in configured accounts, applications, the profile skill inventory and per-analysis learning progress synchronize with Firestore.
- PDF report generation, complete resume-version comparison, full degree/university profile editing, a full interview feedback engine and production analytics are not implemented in this first deliverable.
- The official Hackyard judge-made answer key was not supplied, so the competition's 70% acceptance threshold is **not verified**. The project includes metrics and tests, not a claim of judge-set performance.

## Technology

- Frontend: HTML, CSS, JavaScript, Vite.
- Authentication: Firebase Authentication.
- Application records: Cloud Firestore.
- File storage: Supabase Storage (private bucket, server-side service role only).
- Backend: Node.js, Express.
- Document processing: `pdfjs-dist`, `mammoth`, optional OCR.Space endpoint.

## Requirements

- Node.js 20 or later (Node.js 22 LTS recommended).
- npm 10 or later.
- For cloud mode: a Firebase project and Supabase project.

## Run locally — preview mode

1. Install dependencies from the project root:

   ```bash
   npm install
   ```

2. Copy the root environment template:

   ```bash
   cp .env.example .env
   ```

   Keep `DEMO_MODE=true` for preview analysis. No Firebase or Supabase credentials are needed for the text-based preview.

3. Copy the frontend environment template:

   ```bash
   cp client/.env.example client/.env.local
   ```

   Keep `VITE_API_BASE_URL=http://localhost:8787`.

4. Start both applications:

   ```bash
   npm run dev
   ```

5. Open the Vite address shown in the terminal (normally `http://localhost:5173`). Click **Explore preview**, open **Analyze a role**, paste resume text and a job description, and run the analysis.

The API health endpoint is `http://localhost:8787/api/health`.

## Configure Firebase

1. Create a Firebase project in the Firebase console.
2. Enable Email/Password and Google providers under Authentication.
3. Create a Cloud Firestore database.
4. Register a web app and copy its public web configuration into `client/.env.local`:

   ```dotenv
   VITE_FIREBASE_API_KEY=...
   VITE_FIREBASE_AUTH_DOMAIN=...
   VITE_FIREBASE_PROJECT_ID=...
   VITE_FIREBASE_STORAGE_BUCKET=...
   VITE_FIREBASE_MESSAGING_SENDER_ID=...
   VITE_FIREBASE_APP_ID=...
   ```

5. Create a service account for the Node.js server. Put its project ID, client email and private key in the root `.env`:

   ```dotenv
   FIREBASE_PROJECT_ID=...
   FIREBASE_CLIENT_EMAIL=...
   FIREBASE_PRIVATE_KEY="-----BEGIN PRIVATE KEY-----\\n...\\n-----END PRIVATE KEY-----\\n"
   ```

   Never put the Admin private key in `client/.env.local` or any `VITE_` variable.

6. Deploy `firestore.rules` from the Firebase console or Firebase CLI. These rules scope user subcollections to the authenticated UID. The server uses Firebase Admin after verifying the user's Firebase ID token.

## Configure Supabase private storage

1. Create a Supabase project.
2. In Storage, create a bucket named `scaleneok-documents` and make it **private**.
3. Put these values in the root `.env` only:

   ```dotenv
   SUPABASE_URL=https://YOUR_PROJECT.supabase.co
   SUPABASE_SERVICE_ROLE_KEY=YOUR_SERVICE_ROLE_KEY
   SUPABASE_STORAGE_BUCKET=scaleneok-documents
   ```

4. Never expose the service-role key in frontend variables, source code, browser requests or public repositories. The Express server verifies the Firebase ID token before accessing the private bucket.

## Optional OCR for scanned PDFs

Scanned PDFs contain page images rather than selectable text. Set an OCR.Space key in the root `.env` to enable the included OCR fallback:

```dotenv
OCR_SPACE_API_KEY=YOUR_OCR_SPACE_KEY
```

**Privacy note:** enabling this option sends the uploaded PDF to the OCR.Space service for processing. Review its terms and privacy policy and use it only when appropriate for the documents you process. Without this key, scanned PDFs are flagged for manual text entry rather than silently treated as parsed.

## Environment variables

| Variable | Location | Purpose |
|---|---|---|
| `PORT` | root `.env` | Express API port (default 8787) |
| `CLIENT_ORIGIN` | root `.env` | Allowed browser origin(s), comma-separated |
| `DEMO_MODE` | root `.env` | Enables the explicitly non-persistent preview analysis endpoint |
| `FIREBASE_PROJECT_ID` | root `.env` | Firebase Admin project ID |
| `FIREBASE_CLIENT_EMAIL` | root `.env` | Firebase Admin service account email |
| `FIREBASE_PRIVATE_KEY` | root `.env` | Firebase Admin service account private key |
| `SUPABASE_URL` | root `.env` | Supabase project URL |
| `SUPABASE_SERVICE_ROLE_KEY` | root `.env` | Server-only Supabase service key |
| `SUPABASE_STORAGE_BUCKET` | root `.env` | Private storage bucket name |
| `OCR_SPACE_API_KEY` | root `.env` | Optional scanned-PDF OCR provider key |
| `VITE_FIREBASE_*` | `client/.env.local` | Firebase web SDK config |
| `VITE_API_BASE_URL` | `client/.env.local` | Express API URL |

## API overview

- `GET /api/health` — service/configuration status.
- `POST /api/demo/analyze` — preview-only text analysis; only enabled when `DEMO_MODE=true`.
- `POST /api/documents` — authenticated private document upload (`multipart/form-data`, fields `file` and `kind`).
- `GET /api/documents` — authenticated user's document metadata.
- `GET /api/documents/:id/download` — short-lived signed URL for an owned document.
- `DELETE /api/documents/:id` — delete an owned document from storage and Firestore.
- `POST /api/analyze` — analyze an owned resume and job-description document by ID.
- `POST /api/analyze/text` — authenticated analysis from pasted text.
- `GET /api/analyses` — authenticated analysis history.
- `GET /api/profile`, `PUT /api/profile` — authenticated profile and self-reported skills.
- `GET /api/applications`, `POST /api/applications`, `PATCH /api/applications/:id`, `DELETE /api/applications/:id` — user-scoped application tracking.
- `GET /api/learning-progress`, `PUT /api/learning-progress/:analysisId` — user-scoped learning task progress.

All authenticated routes require `Authorization: Bearer <Firebase ID token>`.

## Run tests

```bash
npm test
```

The test suite covers aliases, common false-positive pairs, requirement-line evidence, learning plan length and evaluation metrics.

## Build the frontend

```bash
npm run build
```

The built static app is written to `client/dist`.

## Deployment notes

- Deploy the `client` build as a static Vite site (for example, on Vercel or Netlify).
- Deploy the Node server as a Node.js web service (for example, Render, Railway or another Node host).
- Set the server environment variables in the hosting provider's secret manager.
- Set `CLIENT_ORIGIN` to the deployed frontend origin.
- Set `VITE_API_BASE_URL` to the deployed API URL before building the frontend.
- Enable Firebase authorized domains for the deployed frontend.
- Keep the Supabase bucket private.
- Set `DEMO_MODE=false` in production.
- Configure rate limits, logging, backups, retention, monitoring and legal/privacy disclosures before accepting real student documents.

## Important accuracy note

The current matching engine is a deterministic baseline with a curated skill taxonomy and aliases. It is explainable and testable, but it is not an AI model and it has not been compared with the official competition answer key. To validate the 70% requirement, create a judge-format dataset, run a missing-skill evaluation against it, inspect false positives/negatives, and expand the taxonomy and extraction tests based on the results.
