# Sistema de Gestão de Projetos (2026)

## Overview
Single-page app (Portuguese, pt_PT) for Fernando Costa ("Nando") to run a personal executive coaching / project-management system through 2026: a daily AI-generated briefing, decision log, KPI tracking, weekly review, project analysis, and reusable templates. It is client-side only — one `index.html` file, no backend, no build step. Data persists in `localStorage` and can optionally sync via a GitHub Gist.

## Architecture
Everything lives in `index.html`:
- **`<style>` block** — all CSS, dark theme via CSS custom properties (`--bg`, `--amber`, `--teal`, etc.), tab/nav layout, cards, forms, lock screen, AI disclosure banner.
- **`<body>`** — static markup for all tabs (rendered/hidden via `showTab()`), the password lock screen, and the API-key modal.
- **`<script>` block** — all app logic (vanilla JS, no framework, no bundler).

### Tabs (nav)
`briefing` (Hoje) · `decisoes` · `kpis` · `revisao` · `projectos` · `templates` · `config` — switched client-side by `showTab(id, btn)`.

### Data & storage
- `store` object (`localStorage`, prefix `g26_`) wraps get/set with JSON serialize/parse.
- Keys of note: `g26_apikey`, `g26_model`, `g26_anchor`, `g26_pw` (lock screen password, default `1979`), `g26_ghtoken`, `g26_gistid`, `g26_gdrive_client_id`, `g26_gdrive_api_key`, plus feature data (chat history, decisions, KPIs, projects, reviews).
- `sessionStorage.g26_unlocked` gates the lock screen per session.
- Optional GitHub Gist sync (`syncToGist` / `syncFromGist` / `createGist`) mirrors local data across devices using a personal GitHub token.
- Optional Google Drive picker integration for attaching files (`getGdriveClientId`/`getGdriveApiKey`, Google Picker API).

### AI integration
- Calls the Gemini API directly from the browser (`callGemini`, `callGeminiMultiturn`) using the user-supplied API key stored in `localStorage`. Default model: `gemini-3-flash-preview`.
- `BASE_SYSTEM_PROMPT` (index.html:893) hardcodes the coaching persona to a specific user: Fernando Costa (Nando), his role (E-Shop & Digital Marketing Manager, Norauto Portugal), his anchor project (Norauto VisionAI+), and his 2026 directives. The app is not generic out of the box — reusing it for another person/project means editing this prompt.
- Used for: daily briefing generation (`generateBriefing`), briefing chat (`sendChatMessage`), project analysis (`analyseProject`), and weekly review feedback (`generateReviewFeedback`).
- An EU AI Act Article 50 disclosure banner (`.ai-banner`) is shown because the app is AI-mediated.
- **API key handling**: the key is entered via a modal (`saveApiKey()`, index.html:2326) and stored only in the browser's `localStorage` — never hardcode it into `index.html`, since the file may end up in a shared/public repo. To let someone else test the app, give them a separate, budget-limited API key to paste in themselves rather than embedding one in the code.

### Auth
The lock screen UI still renders on load (`checkAndShowLock`), but `unlockApp()` no longer checks a password — the button reads "Entrar sem pass" and clicking it just sets `sessionStorage.g26_unlocked` and enters the app. `getStoredPw()`/`changePassword()` and the `g26_pw` key are dead code kept in place but unused by the entry flow.

## Commands
No build tooling. Open `index.html` directly in a browser, or serve statically, e.g.:
```
npx serve .
```

## Conventions
- Single-file, vanilla JS/CSS/HTML — no dependencies beyond Chart.js (CDN) and Google Fonts.
- All UI copy and AI prompts are in European Portuguese (pt_PT).
- Functions are global (no modules/bundler); grouped by feature area within the one `<script>` block in this order: storage/prompt → Gemini calls → briefing/chat → file/project handling → Google Drive → decisions → KPIs → weekly review → templates → header/lock/config/sync → `init()`.
- `localStorage` keys are namespaced with the `g26_` prefix.
- Init logic wrapped in try/catch inside `init()`, run on `DOMContentLoaded`.

## Key files
- `index.html` — the entire application (HTML + CSS + JS).

## Notes (Q&A log)
- **App scope**: not a generic project-management tool — `BASE_SYSTEM_PROMPT` is hardcoded to Nando's role, anchor project (Norauto VisionAI+), and 2026 directives. Reusable for other people/projects only by editing that prompt (index.html:893).
- **Explaining to the professor**: "É uma aplicação web pessoal com IA (Gemini) que atua como coach executivo diário — gera briefings, decisões, KPIs e revisões semanais, com disclosure ao abrigo do Art. 50 do AI Act — e é facilmente reaproveitável para outros perfis ou projetos bastando trocar o prompt-base do coach (index.html:893)."
- **Lock screen password**: removed from the entry flow — the button now says "Entrar sem pass" and `unlockApp()` no longer validates a password (see Auth section). The old default was `1979`.
- **Letting the professor test it**: don't embed an API key in `index.html` (it would be exposed if the repo is shared/public). Instead, create a separate, budget-capped Gemini API key at aistudio.google.com/apikey and give it to the professor to paste into the app's API-key modal (`saveApiKey()`, index.html:2326); revoke it after testing.
- **Cost estimate**: Gemini Flash usage for a short demo (a few briefings/chat messages) costs a small fraction of a euro; a 4€ budget is comfortable headroom. Set a budget alert on the key's Google Cloud project as a safety net.
