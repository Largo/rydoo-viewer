# Rydoo Receipt Downloader

A single-file, zero-dependency browser tool for browsing exported [Rydoo](https://www.rydoo.com/) expenses and downloading their receipts, images, and attachments. Everything lives in [index.html](index.html) — no build step, no server, no npm install.

## What it does

1. **Connects** to the Rydoo API using OAuth2 `client_credentials` (Client ID + Secret).
2. **Lists** exported expenses over a chosen time window, with search-by-`refId`.
3. **Previews** each expense's files inline (images and PDFs) and **downloads** them.

For every expense it exposes up to three file types:

| File | Source | Delivery |
|------|--------|----------|
| Image | `expense.image` (URL or inline base64) | direct load / data URL |
| Attachment(s) | `expense.attachments[]` | direct load from `streaming.rydoo.com` |
| Receipt PDF | `GET /v3/receipts/by-expense/{id}` | fetched as a blob from `api.rydoo.com` |

## Usage

Because it's a static page, you can just open it:

```
# open index.html directly in a browser, or serve the folder
python -m http.server 8000
```

Then:

1. Enter your Rydoo **Client ID** and **Client Secret**, optionally tick **Use sandbox host**, and click **Connect**. Credentials can be remembered in `localStorage` on the device (**Remember on this device**, on by default).
2. Set **Past days** and **Max results**, then click **Load expenses**.
3. Use **👁 View** for an inline preview or **⬇ Download** to save a file.

## Endpoints used

- `POST https://accounts.rydoo.com/connect/token` — OAuth token (scopes: `receipts:read expenses:read company`)
- `GET  {api}/v2/expenses/exported` — paged expense list (page size 100)
- `GET  {api}/v3/receipts/by-expense/{id}` — receipt PDF

`{api}` is `https://api.rydoo.com` (production) or `https://sandbox-api.rydoo.com` (sandbox).

## CORS caveat

This is a **pure browser tool** — it calls the Rydoo API directly from your browser, so cross-origin behaviour depends on Rydoo's CORS headers for your origin:

- **Token** and **receipt PDF** requests are CORS-friendly and work in a normal browser.
- **Images/attachments** on `streaming.rydoo.com` block cross-origin `fetch()`. The tool works around this by loading the resource URL directly (image/iframe) with the token passed as an `access_token` query param ([RFC 6750 §2.3](https://datatracker.ietf.org/doc/html/rfc6750#section-2.3)). If a file still fails to load, the endpoint requires header-based auth that a browser can't supply — a small proxy would be needed.

If token or receipt requests are blocked, use a CORS-enabled browser profile/extension.

## Security

Credentials never leave the page except to `accounts.rydoo.com` (token) and the Rydoo API hosts. If **Remember on this device** is enabled, the Client ID and Secret are stored in plain text in the browser's `localStorage` — clear it or untick the box on shared machines.
