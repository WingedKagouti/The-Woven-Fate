# The Woven Fate

An AI-narrated, text-based RPG that runs entirely in your browser, plus the tools for preparing and testing the lorebooks that feed it.

Everything here is a **single, self-contained HTML file**. There is no build step, no server, and nothing to install. Download a file and open it in a modern browser.

## What's in the box

| File | What it is |
|---|---|
| `The_Woven_Fate.html` | The game. Story memory is a running journal and chapter "books". |
| `The_Woven_Fate_Dynamic.html` | The same game, with the journal replaced by a **dynamic lorebook** the AI writes and maintains as you play. |
| `Lorebook_Converter.html` | Converts lorebooks from other tools into the Woven format. |
| `Lorebook_Testbench.html` | Offline analysis of how a lorebook behaves against sample scenes and a character budget. |

A typical workflow:

1. **Convert** an existing lorebook with the Converter (or write one by hand).
2. **Test** it in the Testbench to see which entries fire and which get squeezed out by the budget.
3. **Play** it in The Woven Fate (or the Dynamic variant).

---

## The Woven Fate (`The_Woven_Fate.html`)

You describe what your character does. An AI game master narrates the result in second person.

### Getting started

1. Open the file and choose **+ Add API Target** (or the ⚙ icon).
2. Pick a service type, enter the URL and your API key, and save. A model name is optional and overrides the default.
3. On the setup screen pick a **setting**, a **plot engine**, a **mood**, and optionally a character gender and description. **Random ⚄** picks for you.
4. Optionally load a lorebook with **📖 Load / manage**.
5. Press begin and start acting.

### Supported API services

Anthropic (Claude), OpenAI (GPT), Google Gemini, Groq, Mistral, DeepSeek, xAI (Grok), OpenRouter, Together AI, Fireworks AI, and any OpenAI-compatible endpoint (Ollama, LM Studio, custom proxies). You can save several named targets and switch between them. The default model is `claude-sonnet-4-6`.

Calls go **directly from your browser** to the service you choose. There is no backend proxy. Local or self-hosted endpoints may need CORS enabled.

### Story options

- **Settings (9):** Contemporary, Urban Fantasy, High Fantasy, Folk & Myth, Steampunk, Sci-Fi, Western, Historical, School & Academy
- **Plot engines (12):** Story-Driven, Detective & Mystery, Crime, Spy & Espionage, Action, Adventure, Superhero, Gothic Horror, Cosmic Horror, Monster Horror, Swashbuckling, Political Intrigue
- **Moods (8):** Tragic, Dark, Neutral, Whimsical, Comedic, Romantic, Mysterious, Heroic

### How a turn works

For each action you type, the game master decides one of three things:

- **Narrative:** nothing is at stake, so the story simply continues.
- **Item resolution:** a specific carried item plausibly solves the problem and is used up.
- **Challenge:** something meaningful is at risk, so Fate gets a say.

On a challenge you are shown three face-down buttons: **Fortune Smiles**, **Fate Turns Away**, and **The Weave**. The mix depends on how well your gear, skills, and conditions suit the task:

| Situation | Buttons on offer |
|---|---|
| Neutral odds | Fortune Smiles, Fate Turns Away, The Weave |
| Favourable | Fortune Smiles, The Weave, The Weave |
| Unfavourable | Fate Turns Away, The Weave, The Weave |

The positions are shuffled. The Weave is a wildcard that resolves to a random one of the four outcomes below.

| Outcome | Meaning |
|---|---|
| Victory | Complete triumph |
| Costly Victory | Won, but at a price |
| Near Miss | Failed, but not broken |
| Defeat | Total failure |

The AI then narrates the chosen outcome.

### Tracked state

After each narration, the game extracts and tracks:

- Your current **quest** and completed quests
- **Key items** and **general items** (gained and lost)
- **Tools of the Trade**, a baseline kit derived from your character
- **Characters** you meet, including role, status (ally, hostile, romantic, and so on), aliases, and merges
- **Conditions**: superficial, moderate, severe, and positive. Moderate and severe conditions, and positive ones, feed back into later challenge odds.

### Other features

- **Suggestions** for what to do next, with re-roll
- **Continue the story** and **Regenerate** for the last response
- **Books:** when enough has happened the game offers to close a chapter, summarise it, and carry the summary forward
- **Saves:** manual saves, an **autosave** (every 5 turns, toggleable), rename, delete, and export/import as JSON
- Light and dark themes
- A **Debug stats** toggle that shows lorebook and API stats under each response

### Lorebooks in the game

A lorebook is a JSON file (see [Lorebook format](#lorebook-format)). On every turn the game scans the **last 4 messages plus your current action** for entry keys, then adds matching entries to the narrator's context until the **character budget** runs out. The budget can be adjusted in the lorebook dialog. The narrator is told to treat lore as canon but never to quote it or mention a lorebook.

---

## The Woven Fate: Dynamic (`The_Woven_Fate_Dynamic.html`)

Identical gameplay, with a different long-term memory system.

| | Standard | Dynamic |
|---|---|---|
| Story memory | Journal of events + chapter books | AI-maintained **story-so-far** summary + **dynamic lorebook** |
| Extra AI call per turn | no | yes (the memory update) |
| Memory budget | n/a | **Story lore budget**, default 4,000 chars (Settings ⚙), separate from the static lorebook |
| Chat history window | 20 messages | 15 messages |

After each turn the AI is asked to update a rolling summary (up to 1,500 characters) and to create or revise entries for people, places, and things that have appeared (up to 150 entries, 600 characters each). These entries are keyed by name and fire like any other lorebook entry, so the game remembers things *when they become relevant* rather than carrying everything all the time. The story-so-far is always included and is charged to the dynamic budget first.

- If a memory update fails, it is retried together with the next turn.
- Your own static lorebook still works alongside it, with its own budget.
- Because it makes an additional API call each turn, it costs somewhat more and is slower per turn than the standard version.
- Dynamic lore is stored inside your saves. Saves from the standard version can be loaded; their journal and book content is used to seed the story-so-far.

Pick Dynamic for long campaigns where continuity matters. Pick the standard version for cheaper, simpler sessions.

---

## Lorebook Converter (`Lorebook_Converter.html`)

Converts lorebooks exported from other tools into the Woven format. Per its interface, it targets SillyTavern, NovelAI, JanitorAI, Agnai, SpicyChat, and similar JSON exports.

**Usage:** drop a `.json` lorebook onto the window (or click to browse), check the preview, and press **Download Converted JSON**. The output is saved as `<original name>.woven.json`.

What it does:

- Detects the shape of the input (object-keyed entries as in SillyTavern, or array-based entries)
- Merges primary and secondary keys into one de-duplicated key list
- Skips entries that are disabled
- Maps the source's priority or insertion order onto the game's **1–10** scale (1 is highest priority). Values already between 1 and 10 pass through unchanged, larger values are scaled down, and entries with no priority get **5**.
- Carries over the name, description, and budget (defaults to **8,000** characters if the source has none)

Things to check after converting:

- **Review the result.** Formats vary a lot between tools, and the converter only reads common field names.
- **Entries need at least one key.** The game discards entries with no keys, so "always-on" or "constant" entries from other tools won't carry over as always-on. Give them keys, or fold them into the story setup.
- **Budget.** A budget found in the source file is carried over as-is (a source `token_budget` is copied as a character count, not converted). Otherwise the converter writes 8,000, matching the game's own default. You can change it in the game's lorebook dialog, and the Testbench can help you pick a value.
- **Priorities from other tools.** Order-style values above 10 are rescaled approximately (higher source value becomes a lower, stronger priority number), so skim the results and adjust any entries that matter.
- Conversion runs locally. Your file is not uploaded anywhere.

---

## Lorebook Testbench (`Lorebook_Testbench.html`)

A deterministic simulator for answering "will my lorebook actually do what I expect?" without spending any API calls. It reproduces the game's keyword matching and budget selection, then shows how entries compete for space.

It opens with a small embedded sample (the *Atheron* lorebook and one test scene). Use **Reset to embedded data** to return to it.

### Loading your own data

- **Load lorebook (.json):** the Woven format
- **Load test scenes (.md / .json):** pieces of sample text to scan

Scene files in Markdown look like this:

```markdown
## scene1: First-years gather in the great hall
*Setting:* Atheron Magical Academy, start of term
*Tests:* Whether academy entries outrank minor lore
*Cast:* Mira, Headmaster Voss
*Length:* 1200
> The scene text goes here, as a blockquote.
> Continue the quote on every line.
```

Each scene starts with a `## sceneN:` heading, optional `*Setting:*`, `*Tests:*`, `*Cast:*`, and `*Length:*` lines, and the text as `>` blockquote lines. A JSON array of `{id, title, setting, tests, cast, text}` objects also works.

### Controls

- **Budget:** characters available for lore, with a live **estimated token range** next to it
- **Simulated turns:** 50, 200, 500, or 1,000 random runs per scene. **Re-run simulation** repeats with the same settings.
- **Theme toggle** (dark and light)

### Tabs

- **Overview:** one row per scene showing how many matched entries made it in, how much of the budget was used, whether everything fit, and counts of *contested* and *never in* entries
- **Scenes:** the scene text with matched keys highlighted, a list of matched entries with their hit keys and inclusion probability, and the assembled context from one simulated turn
- **Lorebook:** every entry with its priority, size, and statistics, sortable and filterable, with a note of which scenes use it

### How selection is modelled

This mirrors the game:

- Keys match as **whole words or phrases**, case-insensitively, with curly quotes normalised. `*` matches any word characters (at most two per key).
- Matched entries are tried in **priority order, 1 first and 10 last**, in random order within a priority level. An entry that doesn't fit the remaining budget is skipped, and the rest are still tried.
- The budget counts **entry content only**. The `[Name]` labels are free.
- An entry is **contested** if it makes the cut in some simulated turns but not others, and **never in** if it matched but practically never fits.

### Limitations

- Each scene is scanned on its own, as if it were a first turn. In the real game the last 4 messages of history are scanned too.
- Token figures are estimates (roughly 3–4.5 characters per token for Latin text, with a separate range for Chinese, Japanese, and Korean). They are not a tokenizer count.
- It tests matching and budgeting only. It does not call any AI, so it cannot tell you how the narrator will *use* the lore.

---

## Lorebook format

The format both the game and the Testbench read:

```json
{
  "name": "Atheron",
  "description": "A land of magic, adventure and intrigue.",
  "character_budget": 8000,
  "entries": [
    {
      "name": "Atheron Magical Academy",
      "keys": ["academy", "atheron academy", "magic school"],
      "content": "A centuries-old school of magic perched above the harbour town...",
      "priority": 2
    }
  ]
}
```

| Field | Notes |
|---|---|
| `name`, `description` | Labels only |
| `character_budget` | Maximum characters of entry content added per turn. Falls back to 8,000 if missing. |
| `entries[].name` | Shown as the `[Name]` label in context |
| `entries[].keys` | Words or phrases that trigger the entry. At least one is required. `*` is a wildcard. |
| `entries[].content` | The lore text the narrator sees |
| `entries[].priority` | 1 to 10, where **1 is highest** and is added first when space is tight. Defaults to 5. |

Entries without keys or content are dropped on load.

---

## Privacy and storage

- API targets (including **API keys**), saves, session state, and theme preferences are stored in your browser's `localStorage`. They never leave your machine except in requests to the API service you configured.
- Your lorebooks and story text *are* sent to that service as part of prompts.
- Clearing site data for the page removes your keys and saves. Use **Export** on a save to keep a backup.
- Individual saves are limited to roughly 4.8 MB.
- Both game files use the same storage namespace (`woven_fate::`), so when they are served from the same browser origin they can see each other's saves and API targets. Browsers treat `file://` pages differently, so if saves don't appear in the other version, use **Export** and **Import**.

## Requirements

A current desktop or mobile browser (Chrome, Edge, Firefox, Safari) and, for the game, an API key or a reachable local model endpoint. The Converter and Testbench need no API access and work offline.
