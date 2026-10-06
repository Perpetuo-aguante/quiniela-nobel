# Quiniela Nobel de Literatura 2026 · Perpetuo

Static voting page ("boleta electoral") where readers predict the 2026 Nobel Prize in Literature winner.
Live at **https://nobel.perpetuo.global** via GitHub Pages.

## Files
- `index.html` — the whole app (HTML + CSS + JS in one file, no build step).
- `assets/` — Perpetuo logo and the four punctuation characters from the brandbook (webp, transparent).
- `CNAME` — custom domain for GitHub Pages (`nobel.perpetuo.global`).
- `.nojekyll` — serve files as-is.

## How it works
1. Voter marks one candidate (crayon-style X animation) or writes in an unlisted name.
2. Form asks for name, email and the "Consentimiento" toggle (subscribed / agrees to be subscribed).
3. Vote is POSTed as JSON to an n8n webhook, then the ballot folds and drops into the urn.
4. One vote per person: localStorage on the client + email dedupe on the server (n8n answers 409 with the existing vote).
5. Voting closes 2026-10-08 11:00 UTC (5:00 a.m. Mexico City), before the announcement in Stockholm. Server answers 410 after that.
6. Candidate photos are fetched at runtime from the Wikipedia REST API (`/api/rest_v1/page/summary/<title>`) and shown as brand-color duotones. If a photo can't load, the initials stay.

## Config (top of the `<script>` in index.html)
```js
const CONFIG = {
  WEBHOOK_URL: "https://esperpetuo.app.n8n.cloud/webhook/quiniela-nobel-2026",
  CIERRE: "2026-10-08T11:00:00Z",
  CLAVE_LOCAL: "perpetuo-nobel-2026"
};
```
Candidates live in `CANDIDATOS` (alphabetical by surname) and their Wikipedia titles in `WIKI`.

### Payload sent to the webhook
```json
{ "candidatoId": "rivera-garza", "candidato": "Cristina Rivera Garza", "otro": "",
  "nombre": "…", "correo": "…", "consentimiento": true, "folio": "123456", "fecha": "ISO-8601" }
```
Expected responses: `200` saved · `409` already voted (body: `{candidato, correo, folio}`) · `410` closed · `400` invalid.
The webhook must allow CORS from `https://nobel.perpetuo.global`.

## Deploy (GitHub Pages)
1. Push this folder to the repo root (branch `main`).
2. Settings → Pages → Source: Deploy from a branch → `main` / `(root)`.
3. Custom domain: `nobel.perpetuo.global` (already in `CNAME`).
4. DNS at the perpetuo.global provider: `CNAME  nobel  →  <github-user-or-org>.github.io`.
5. Once the certificate is issued, enable **Enforce HTTPS**.
6. Recommended: verify `perpetuo.global` under the account/org Settings → Pages → Verified domains.

## Backend
The n8n workflow (`n8n-quiniela-nobel-2026.json`, kept outside this public repo) validates the vote,
dedupes by email and stores it in the Notion database "Quiniela Nobel 2026 · Votos".
Do not commit credentials or the workflow file here.
