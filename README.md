# AI Browser Copilot

AI Browser Copilot is a Chrome extension that reads useful text from the current browser tab and sends it to a small Node.js backend for Gemini-powered actions:

- Summarize
- Explain
- Give hints
- Improve writing

It is designed for technical reading and learning workflows across GitHub READMEs, documentation pages, YouTube transcripts, and LeetCode problem statements.

## Tech stack

- Chrome Extension Manifest V3
- `chrome.scripting.executeScript` for on-demand page extraction
- Popup UI with HTML, CSS, and vanilla JavaScript
- Node.js backend using the built-in `http` module
- Google Gemini API for NLP tasks via prompt engineering
- Render.com deployment configuration for the backend

## Project structure

```text
.
├── manifest.json
├── popup.html
├── popup.js
├── content.js
├── styles.css
├── render.yaml
└── server
    ├── server.js
    ├── package.json
    ├── package-lock.json
    └── .env.example
```

## Local development setup

1. Install Node.js 18 or newer.
2. Create a Gemini API key from Google AI Studio: https://aistudio.google.com/apikey
3. Create `server/.env`:

   ```env
   GEMINI_API_KEY=your_gemini_api_key_here
   GEMINI_MODEL=gemini-2.5-flash
   PORT=3000
   ```

4. Start the backend:

   ```powershell
   cd server
   npm install
   npm start
   ```

5. Load the extension:
   - Open `chrome://extensions`
   - Turn on Developer mode
   - Click Load unpacked
   - Select the top-level `AI Browser Copilot` folder

6. Test on a normal webpage:
   - Open a GitHub README or documentation page
   - Click the AI Browser Copilot extension icon
   - Choose a mode
   - Click Run

## Using the deployed backend

After deploying the backend on Render, update `popup.js`:

```js
const BACKEND_URL = "https://your-render-service.onrender.com";
```

`manifest.json` already allows local development URLs and Render-hosted URLs:

```json
"host_permissions": [
  "http://localhost:3000/*",
  "http://127.0.0.1:3000/*",
  "https://*.onrender.com/*"
]
```

Reload the extension in `chrome://extensions` after changing `popup.js`.

## Render deployment

The backend is configured with `render.yaml`:

```yaml
services:
  - type: web
    name: ai-browser-copilot-server
    runtime: node
    rootDir: server
    buildCommand: npm install
    startCommand: npm start
```

The server reads Render's dynamic port from `process.env.PORT`, so it can run both locally and on Render.

Do not commit `server/.env`. Add `GEMINI_API_KEY` in the Render dashboard as an environment variable.

## Site support

- GitHub and generic docs pages: uses visible DOM text.
- LeetCode: waits for dynamic problem description content before extraction.
- YouTube: reads transcript segments when the transcript panel is open, otherwise falls back to visible description/page text.
- LinkedIn: fails safely if it cannot read the post composer instead of scraping unrelated profile content.

## Known limitations / future work

- Gmail integration is not built. It would require OAuth 2.0 and a separate authentication flow beyond this project's scope.
- LinkedIn composer extraction is partially working. The extension correctly detects when it cannot read the post composer and fails safely with a clear error rather than falling back to scraping unrelated profile data. This was a deliberate privacy-conscious fix, not an oversight.
- PDF/PPT file support is not built. Chrome's native PDF viewer and canvas-rendered slide content require different extraction techniques than DOM text scraping.
- YouTube transcript extraction works, but depends on the transcript panel being manually opened by the user first.

## Security notes

- Keep `server/.env` local only.
- Never commit Gemini API keys.
- The extension sends extracted page text to your configured backend, which then sends it to Gemini.

## Quick verification checklist

- Backend off: popup should show a clear backend-not-running error.
- Backend on: GitHub README should summarize successfully.
- YouTube: open transcript panel first, then summarize.
- LeetCode: open a problem page and summarize/explain the problem statement.
- LinkedIn: if composer extraction fails, the popup should show a safe error instead of summarizing profile/page text.
