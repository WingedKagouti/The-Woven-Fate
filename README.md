# The Woven Fate

A single-file, browser-based text RPG. An AI game master narrates an evolving story in second person; you type what you do; the story adapts. There is no build step, no backend, and no install — `The_Woven_Fate.html` is the entire application.

---

## Table of contents

- [Playing the game](#playing-the-game)
- [How it works, technically](#how-it-works-technically)
- [Hosting it](#hosting-it)
- [Known limitations](#known-limitations)

---

## Playing the game

### Starting a tale

On the setup screen you choose:

| Choice | What it affects |
|---|---|
| **Mood** (Tragic, Dark, Neutral, Whimsical, Comedic, Romantic, Mysterious, Heroic) | The emotional tone of the prose — how forgiving or brutal the story is. |
| **Setting** (Contemporary, Urban Fantasy, High Fantasy, Folk & Myth, Steampunk, Sci-Fi, Western, Historical, School & Academy) | The world your character exists in. |
| **Plot Engine** (Story-Driven, Detective & Mystery, Crime, Spy, Action, Adventure, Superhero, Gothic/Cosmic/Monster Horror, Swashbuckling, Political Intrigue) | The *kind* of plot that emerges — a detective story surfaces clues and suspects, an action story escalates set-pieces, and so on. "Story-Driven" lets the plot shape itself from Mood + Setting alone. |
| **Character** (optional) | Gender and a free-text description. Mentioning an occupation or crew (*"a grizzled detective"*, *"captain of a smuggling crew"*) shapes your starting gear and supporting cast — see **Tools of the Trade** below. |
| **API Target** | Which AI service answers your actions (see [Hosting it](#hosting-it) and the Technical section). You must select or add one before you can begin. |

Hit **Random ⚄** to roll Mood/Setting/Engine/Character for you, or **Begin Your Tale** once Mood, Setting, and an API Target are set.

### Playing a turn

Each turn, you either:

- **Type an action** in the box at the bottom and hit **Act** (or Enter — Shift+Enter for a newline), or
- Hit **Continue ⟳** to let the story advance on its own without you specifying an action, or
- Hit **Suggest 💡** for three AI-generated ideas of what you could do next, if you're stuck.

Behind the scenes, your action is judged as one of three things:

- **Plain narrative** — low-stakes actions (looking around, talking) just continue the story.
- **Item resolution** — if a carried item plausibly and directly resolves the action (using a bandage to tend a wound), it's consumed and the story continues — no dice roll needed.
- **A challenge** — anything with real stakes (a fight, a lie, a locked door) triggers **a moment of fate**.

### Moments of fate

When a challenge triggers, three symbols (✎) appear. One or more may be marked *favorable* or *unfavorable* first, based on your gear, background, and active conditions (see below) — the game tells you why. Pick one, and it reveals one of four outcomes:

| Outcome | Meaning |
|---|---|
| ✦ **Victory** | Complete success. |
| ✗ **Defeat** | Total failure. |
| ◈ **Costly Victory** | You succeed, but at a price. |
| ◇ **Near Miss** | You fail, but not catastrophically. |

The three symbols are a random mix of these outcomes (weighted toward success or failure if the moment was favorable/unfavorable), plus a wildcard **"The Weave"** option that resolves to a fully random outcome when picked. The story then continues based on what you chose.

### Quest, Items, and Conditions ("Quest & Items" / "Status" panels)

The game automatically tracks, without you managing anything directly:

- **Quest** — at most one active goal at a time, set/updated/completed by the narrative itself.
- **Key Items** (max 3) — plot-critical, irreplaceable items.
- **Items** (max 5) — general inventory; click **✕** to drop one manually.
- **Tools of the Trade** — a fixed baseline kit inferred from your character's occupation/setting at the start (a detective's badge and phone, a knight's sword and armor) — always carried, never counted against your item limits.
- **Conditions** (Status panel) — afflictions and buffs, in four tiers: **Superficial** (fades after a few turns), **Moderate**, **Severe** (needs active treatment), and **Positive** (buffs, cleared at scene changes). These feed back into whether your next challenge is favorable or unfavorable.

If a tier fills up, new entries of that severity escalate to the next tier instead of being silently dropped (repeated superficial injuries can cascade into something moderate, then severe) — and if severe conditions are completely full, a bad enough hit can outright fail your active quest.

### Journal & Books

- **Journal panel** — a running list of named characters you've met (with relationship notes and role), plus a log of notable events since the last "Book."
- **Merge Characters** — if the same person gets tracked as two separate entries (e.g. a masked vigilante and their later-revealed civilian identity), select two in the Journal and merge them.
- **Books** — periodically (or after finishing a quest, when the game will suggest it), you can close out a "Book": the AI writes a chapter title and summary condensing everything since the last one, optionally proposes a small addition to your character's description reflecting how they've grown, and archives the detailed event log so it doesn't grow forever. Past Books remain browsable in the Books panel.

### Saving

- **Save** / **Save As New** — manual snapshots, browsable and resumable from the **Saves** screen, where you can also **Rename**, **Export** (download as `.json`), **Import** (upload one back in), or **Delete** them.
- **Autosave** — on by default, saves automatically every 5 narrated turns to a dedicated autosave slot (toggle in the Saves screen).
- A tale too large to fit under the storage size limit will warn you to export and free up space rather than silently fail.

---

## How it works, technically

Everything — markup, styles, and logic — lives in **one HTML file** with no external dependencies and no build step. It's organized into clearly labeled sections inside a single `<script>` block:

```
CONSTANTS & PROMPTS      → moods/settings/engines, tuning limits, theme color tables
PROVIDER REGISTRY        → per-AI-service request/response adapters (see below)
STORAGE SHIM             → a window.storage-compatible API backed by localStorage
API SERVICE & JSON PARSER→ the fetch() call itself, plus a defensive JSON parser
STATE STORE (S)          → one plain object holding all application state
GAME ENGINE LOGIC        → turn resolution, quest/item/condition extraction, saves, books
RENDERING                → HTML-string templates + targeted DOM update functions
EVENT DELEGATION         → two listeners on the root element route every click/input
```

### State & rendering

There's no framework. A single global object `S` holds everything (current screen, mood/setting/engine, player state, message log, save-tracking, etc.), and a `setState(updater, updateFn)` helper is the standard way to mutate it: make the change, persist a lightweight session snapshot to `localStorage`, then run whichever UI refresh actually needs to happen.

Rendering is otherwise plain string-templating into `innerHTML` — but *not* a full-page rebuild on every change. The Setup and Saves screens are cheap enough to fully re-render each time. The **game screen** is mounted once and then only ever patched by small, targeted functions (`updateInventoryUI()`, `updateJournalUI()`, `updateSaveBar()`, `appendMessage()`, and so on) so that typing in the action box, mid-turn scroll position, and text selection all survive updates elsewhere on the screen without being wiped.

New narration is *appended* to the log rather than triggering a rebuild of the whole conversation; quest/inventory/condition tags attached after the fact patch just that one message in place.

### The AI provider system

There is no fixed AI backend. Under the gear icon (⚙), you configure one or more named **API Targets** — each a Service URL, a Service Type, an optional Model override, and an optional API key — and pick which one is active before starting a tale. Calls go **directly from your browser** to that URL; there's no server in between unless you build one yourself.

A small provider registry (`PROVIDERS` in the source) abstracts over the wildly different request/response shapes different AI services use, so the rest of the game just calls one function and gets a plain string back:

| Provider | Auto-detected from | Notes |
|---|---|---|
| Anthropic (Claude) | `api.anthropic.com` | Sends `x-api-key` + the Anthropic-specific headers required for direct browser calls. |
| OpenAI (GPT) | `api.openai.com` | Standard Chat Completions shape. |
| Google Gemini | `generativelanguage.googleapis.com` | Uses `x-goog-api-key`; maps message roles to Gemini's `user`/`model` convention. |
| Groq, Mistral, DeepSeek, xAI (Grok), OpenRouter, Together AI, Fireworks AI | their respective API hostnames | All speak an OpenAI-compatible Chat Completions shape; each just has its own endpoint path and a suggested default model. |
| Other (OpenAI-compatible) | never auto-detected | For Ollama, LM Studio, or any custom/self-hosted OpenAI-compatible server. |

**Detection details:**
- A bare origin (`https://api.openai.com`) gets the provider's conventional path appended automatically; paste a full URL *with* a path and it's used exactly as typed, so you can point at an unusual or custom route.
- `localhost`, `127.0.0.1`, and other numeric/local addresses are **never** auto-detected — the Service Type dropdown starts blank and you must pick one explicitly (Save is disabled until you do), since there's no way to guess what's running on a local port.
- Model names and guessed endpoint paths are best-effort and will go stale over time — the Model field is a free-text override for exactly this reason.

Everything (targets, keys, theme, autosave preference, and a session snapshot for crash-recovery) is stored **only in your browser's `localStorage`**, namespaced under `woven_fate_*` keys. Nothing is ever sent anywhere except the exact URL you configure.

### Turn resolution (what actually gets sent to the AI)

Each player action triggers a small pipeline of model calls:

1. **Judge call** — classifies the action as narrative / item-resolution / challenge, and (for challenges) whether the player's gear, background, or active conditions make it favorable or unfavorable.
2. **Narrative call** — writes the actual prose (2–3 paragraphs, second person, with formatting conventions for names/places/dialogue), given recent conversation history plus a condensed "story so far."
3. **Extraction call** — a separate structured-JSON pass reads the narration that was just written and decides what changed: quest updates, items gained/lost, character updates (including alias/identity merges), and conditions gained/resolved.
4. Periodically, older turns are **compacted** into short journal "events" (and, later, "acts") by a summarization call, so the context sent to the model stays bounded no matter how long the tale runs — this is also what "Books" archive wholesale when you close a chapter.

All model responses expected to be structured are requested as raw JSON and parsed defensively (`safeParseJSON`): code-fence markdown is stripped, and a parse failure logs to the console and falls back to a safe default rather than crashing the turn.

---

## Hosting it

Because it's one static HTML file with no build step, "hosting" just means "serving this file over HTTP(S)."

### Locally

**Simplest — just open it.** Double-click `The_Woven_Fate.html`, or drag it into a browser tab (`file://...`). Everything works this way, *except* that if your chosen AI Target's server enforces CORS restrictions on the `file://` origin, requests may be blocked by the browser — this mainly affects calling the real hosted Anthropic/OpenAI/Gemini APIs directly (see [Known limitations](#known-limitations)); a local server you control (below) generally won't have this problem.

**Slightly more robust — serve it locally**, which behaves closer to a real deployment and is required if you're pointing at a `localhost` API Target (like Ollama) that expects a browser origin rather than `file://`:

```bash
# Python (already installed on most systems)
cd path/to/folder
python3 -m http.server 8080
# then open http://localhost:8080/The_Woven_Fate.html

# Node, if you have it
npx serve .

# or literally any other static file server
```

### Online

Any static hosting provider works — there's no server-side code to run. A few common options:

- **GitHub Pages** — push the file to a repo, enable Pages in the repo settings, done.
- **Netlify / Vercel / Cloudflare Pages** — drag-and-drop deploy of a single file works on all three's free tiers.
- **A plain S3/Cloud Storage bucket with static website hosting enabled**, or literally any web server (nginx, Apache, Caddy) with the file dropped in its web root.

There is nothing to configure server-side — no environment variables, no API keys to set on the host. Every visitor configures their **own** API Target and key (or none, for a keyless local/self-hosted target) from within the app itself, stored only in their own browser.

### A note on CORS if you're hosting your own AI-compatible proxy

If you're running your own backend (to keep a shared API key server-side, add logging, rate-limit, etc.) so that visitors don't need their own key, that server needs to send permissive CORS headers (`Access-Control-Allow-Origin`, etc.) for whatever origin you're hosting this file on, since the request is made directly from each visitor's browser. This is the standard way to work around the [known limitation](#known-limitations) below.

---

## Known limitations

- **Direct browser calls to a commercial AI API may be blocked by CORS.** Anthropic's (and generally, most major providers') API does not currently allow arbitrary browser origins to call it directly, regardless of any headers sent — this is a server-side policy, not something this file can work around. If you hit this, your options are: point an API Target at a small proxy/relay you host yourself (which can add its own permissive CORS headers), or use a target that's designed for this (a local server like Ollama/LM Studio typically allows local browser origins by default).
- **No server-side moderation, rate-limiting, or usage tracking.** Whoever's key is configured in a given browser pays for and is responsible for that browser's usage.
- **Model names and guessed API paths will drift over time** as providers update their lineups — override them via the target's Model field / a full Service URL if a default stops working.
- **State lives in that one browser.** Saves, targets, and keys are all `localStorage`-only; clearing site data or switching browsers/devices starts fresh (use **Export**/**Import** on the Saves screen to move a tale between them).
