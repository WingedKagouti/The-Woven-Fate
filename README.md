# The Woven Fate

An AI-narrated, text-based RPG that runs entirely in your browser, plus the tools for preparing, editing, and testing the lorebooks and rulebooks that feed it.

Everything here is a **single, self-contained HTML file**. There is no build step, no server, and nothing to install. Download a file and open it in a modern browser. `index.html` is a start page that links to everything.

## Contents

- [What's in the box](#whats-in-the-box)
- [The Woven Fate (the game)](#the-woven-fate)
- [The Woven Fate: Dynamic](#the-woven-fate-dynamic)
- [Rulebooks](#rulebooks)
- [Lorebook format](#lorebook-format)
- [Woven Fate Editor](#woven-fate-editor)
- [Lorebook Converter](#lorebook-converter)
- [Lorebook Test Editor](#lorebook-test-editor)
- [Lorebook Testbench](#lorebook-testbench)
- [Privacy and storage](#privacy-and-storage)
- [Requirements](#requirements)

## What's in the box

| File | What it is |
|---|---|
| `index.html` | Start page linking to every tool and to this README. |
| `The Woven Fate.html` | The game. Story memory is a running journal and chapter "books". |
| `The Woven Fate Dynamic.html` | The same game, with the journal replaced by a **dynamic lorebook** the AI writes and maintains as you play. |
| `Woven Fate Editor.html` | A form-based editor for writing, checking, and exporting **lorebooks and rulebooks**. |
| `Lorebook Converter.html` | Converts lorebooks from other tools into the Woven format. |
| `Lorebook Test Editor.html` | A form-based editor for writing and validating the test scenes the Testbench reads. |
| `Lorebook Testbench.html` | Offline analysis of how a lorebook behaves against sample scenes and a character budget. |
| `README.md`, `README.html` | This document, in two formats. |
| `License.txt` | The Woven Fate License. |

There are two kinds of "book" you can load into the game, and they do different jobs:

| | Lorebook | Rulebook |
|---|---|---|
| Purpose | Facts the narrator should know: places, people, rules of the world | Meters that track the story (health, trust, money, time, dice…) and tell the narrator how to behave at each level |
| Triggered by | Keywords appearing in the recent story | Your actions, the AI's judgement, the clock, dice, and other meters |
| File | `.json`, see [Lorebook format](#lorebook-format) | `.json`, schema `rulebook/v1-draft`, see [Rulebooks](#rulebooks) |
| Made with | Converter or Editor | Editor |

A typical workflow:

1. **Convert** an existing lorebook with the Converter, or **write** one (and any rulebook) in the Woven Fate Editor.
2. **Write test scenes** in the Test Editor: short passages that should trigger particular entries.
3. **Test** the lorebook against those scenes in the Testbench to see which entries fire and which get squeezed out by the budget.
4. **Preview** a rulebook in the Editor's Preview & test page to check its levels and what the narrator will be told.
5. **Play** it in The Woven Fate (or the Dynamic variant).

---

## The Woven Fate

*(`The Woven Fate.html`)*

You describe what your character does. An AI game master narrates the result in second person.

### Getting started

1. Open the file and choose **+ Add API Target** (or the ⚙ icon).
2. Pick a service type, enter the URL and your API key, and save. A model name is optional and overrides the default.
3. On the setup screen pick a **setting**, a **plot engine**, a **mood**, and optionally a character gender and description. **Random ⚄** picks for you.
4. Optionally load a lorebook with **📖 Load / manage**, and a rulebook with the **Rulebook → Load / manage** button below it. Without one, the built-in *Health & Fatigue* rulebook is used.
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

Your character's physical and emotional state (health, fatigue, trust, and anything else you define) is handled by the **rulebook**, shown as meters above the story. See [Rulebooks](#rulebooks).

### Other features

- **Suggestions** for what to do next, with re-roll
- **Continue the story** and **Regenerate** for the last response. Continue also runs the rulebook, because the last reply can change the meters.
- **Books:** when enough has happened the game offers to close a chapter, summarise it, and carry the summary forward
- **Saves:** manual saves, an **autosave** (every 5 turns, toggleable), rename, delete, and export/import as JSON. Meter values are part of the save.
- Light and dark themes
- A **Settings hub** (the ⚙ icon) that gathers the API target, the lorebook, the rulebook, and the **Debug stats** toggle, which shows lorebook, rulebook, and API stats under each response

### Lorebooks in the game

A lorebook is a JSON file (see [Lorebook format](#lorebook-format)). On every turn the game scans the **last 4 messages plus your current action** for entry keys, then adds matching entries to the narrator's context until the **character budget** runs out. The budget can be adjusted in the lorebook dialog. The narrator is told to treat lore as canon but never to quote it or mention a lorebook.

Files you load are checked first: oversized files, control characters, and unsafe keys are rejected or cleaned, and entries without keys or content are dropped.

---

## The Woven Fate: Dynamic

*(`The Woven Fate Dynamic.html`)*

Identical gameplay, rulebooks included, with a different long-term memory system.

| | Standard | Dynamic |
|---|---|---|
| Story memory | Journal of events + chapter books | AI-maintained **story-so-far** summary + **dynamic lorebook** |
| Extra AI call per turn | no | yes (the memory update) |
| Memory budget | n/a | **Story lore budget**, default 4,000 chars (Settings ⚙), separate from the static lorebook |
| Chat history window | 20 messages | 15 messages |

After each turn the AI is asked to update a rolling summary (up to 1,500 characters) and to create or revise entries for people, places, and things that have appeared (up to 150 entries, 600 characters each). These entries are keyed by name and fire like any other lorebook entry, so the game remembers things *when they become relevant* rather than carrying everything all the time. The story-so-far is always included and is charged to the dynamic budget first. The **Lore** panel lets you browse the entries and the story-so-far.

- If a memory update fails, it is retried together with the next turn.
- Your own static lorebook still works alongside it, with its own budget.
- Because it makes an additional API call each turn, it costs somewhat more and is slower per turn than the standard version.
- Dynamic lore is stored inside your saves. Saves from the standard version can be loaded; their journal and book content is used to seed the story-so-far.

Pick Dynamic for long campaigns where continuity matters. Pick the standard version for cheaper, simpler sessions.

---

## Rulebooks

A **rulebook** is a set of **meters** that track things about the story, plus rules for how they change and what the narrator is told at each level. It replaces the old fixed "conditions" list. One rulebook is active at a time. You can edit one in the [Woven Fate Editor](#woven-fate-editor).

### Loading a rulebook

Open **Rulebook → Load / manage** on the setup screen, or **⚙ Settings → Rulebook** during play. Choose **Load rulebook…** and pick a `.json` file. The game checks it against the schema and tells you what is wrong if it is refused. **Use built-in** returns to *Health & Fatigue* (a Health pip meter and a Fatigue bar).

The same dialog has a **Character name** field. It sets what `{{char}}` means in the rulebook's instructions. Leave it blank and it means the character the scene focuses on. `{{user}}` always means the player (shown as "you" on screen). Changes apply from the next message.

### What happens each turn

After each message you send (and each press of Continue) the rulebook runs in this order:

1. **Dice** set to roll *before the reply* are rolled.
2. **Keyword rules** are checked against your message.
3. **AI checks** (an extra short call) answer the rulebook's yes/no questions about what just happened, and may detect time skips, item changes, and journal events.
4. **Decay and the clock** step, if nothing else changed a meter.
5. **Dice** set to roll *after the meters* are rolled.
6. **Triggers** run, reacting to the new values.
7. The **narrator** is told the instruction of the level each meter is in, plus any combined rules that apply.

If a call fails or you retry, the game restores the meters to where they were, so a retry doesn't count twice.

### Meter types

| Type | What it is | Typical use |
|---|---|---|
| **Bar** | A value in a range, drawn as a bar | Energy, trust, attraction |
| **Pips** | A small count drawn as icons | Health, lives, charges |
| **Number** | Any number, with optional unit and decimals | Money, score, days |
| **Story clock** | Date and time in the story | Day and night, deadlines, seasons |
| **Dice** | A die (2–1000 sides) rolled on conditions you set | Luck, random events |
| **Select** | One of several states, switched as the story moves | Location, mood, disguise |
| **Collection** | Held items and counted amounts | Inventory, ammunition |
| **Journal** | Events that have happened, remembered by the narrator | Promises made, secrets learned |

Every meter has a name, icon, and colour, and chooses **which text the AI reads** for it (your messages, the character's behaviour, or both). It can be shown to the player or hidden, and in the HUD strip, the panel only, or nowhere. It can list "what raises it" and "what lowers it" for the player, and can **tell the narrator the exact value** instead of just the level.

### Levels

Bars, pips, numbers, dice, and the clock's time-of-day and day-number parts are divided into **levels** (2 to 12). Each level has:

- a **name** and an **upper limit** (the last level may have none),
- an **instruction** for the narrator ("{{char}} is wary and keeps their distance…"),
- a **caption** shown to the player,
- **Shown to the player**, which can hide a level's name and caption until reached,
- **Override**, which makes that level's instruction take priority over what the player asks the character to do. Use it for states like unconsciousness.

### What changes a meter

| Mechanism | How it works |
|---|---|
| **Keyword rules** | Words in *your* message (matched like lorebook keys, `*` wildcard) add to or set the value. |
| **AI checks** | Yes/no questions ("Does {{user}} comfort {{char}}?") that add, set, multiply, divide, or roll a random amount. Each has a cooldown and can be limited to when other meters match. Alternatively the AI **extracts** a signed amount directly. Per-turn limits cap how far one reply can move a meter, and *Negative wins* settles replies that trigger both a rise and a fall. |
| **Decay / per-message change** | A bar drifts after quiet turns; a number can change every message. |
| **Triggers** | On a bar, pips, or number meter: when *other* meters meet conditions, change this one (for example, "when Wallet reaches 0, set Broke to 1"). They fire when the condition becomes true, every turn, every N turns, or once. They never set off each other. |
| **Combined rules** | When several meters' conditions all hold at once, the narrator gets an extra instruction (for example, "trust is high **and** it is night"). |
| **The clock** | Moves a set number of minutes per message and can notice time skips ("the next morning"). |
| **Dice** | Roll every message, every N messages, on keywords, on AI questions, when meters are in a state, or when a meter rises. The result lasts until the next roll or for a set number of messages. |
| **Player editing** | If *Player can edit this* is on, you can change the value yourself: click an option for a Select, or click the value for other types (a calendar picker for dates, numbers for time). |

Conditions can test meter values, the story day or time of day, the weekday or date, which option is selected, how much of an item is held, and how long ago a journal event happened.

### Seeing the meters

The meters appear above the story in three views, switched from the HUD: **Minimal** (small value chips with a hover card), **Expanded** (cards with levels and captions), and the open **Meters panel** with everything. Up to 8 meters show in the strip. There are 86 icons and 28 colours to choose from.

### Limits

| | |
|---|---|
| Meters per rulebook | 15 (only one story clock) |
| Levels per meter | 2–12 |
| Keyword rules / words per rule | 20 / 50 |
| AI questions per meter | 60 |
| Items per Select / Collection / Journal | 20 / 20 / 30 |
| Rules per Collection / Journal | 20 each |
| Combined rules | 30 |
| Triggers per meter | 10 |
| Instruction / caption length | 600 / 120 characters |
| File size | about 1.5 MB |

### Rulebook format

A rulebook file looks like this (shortened):

```json
{
  "schema": "rulebook/v1-draft",
  "exported_at": "2026-11-12T10:00:00.000Z",
  "rulebook": {
    "id": "my-rulebook",
    "name": "My rulebook",
    "description": "What it does.",
    "meters": [
      {
        "id": "trust",
        "type": "bar",
        "label": "Trust",
        "icon": "shield",
        "color": "blue",
        "tracks": "chat",
        "visible": true,
        "min": 0,
        "max": 100,
        "def": 20,
        "keywords": [{ "words": ["promise", "swear"], "delta": 5 }],
        "bands": [
          { "label": "Guarded", "to": 49, "instr": "{{char}} keeps their guard up." },
          { "label": "Open", "to": 100, "instr": "{{char}} speaks freely to {{user}}." }
        ]
      }
    ]
  }
}
```

Meters, rules, and triggers all have their own fields. You rarely need to write them by hand: the [Editor](#woven-fate-editor) builds and checks them. If you use the Rulebook Creator skill to have an AI draft one, open the result in the Editor to verify it. Ids must start with a letter and use only lowercase letters, digits, `-` and `_`.

---

## Lorebook format

The format the game, the Editor, and the Testbench read:

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
| `entries[].keys` | Words or phrases that trigger the entry. At least one is required. `*` is a wildcard (at most two per key). |
| `entries[].content` | The lore text the narrator sees |
| `entries[].priority` | 1 to 10, where **1 is highest** and is added first when space is tight. Defaults to 5. |

Entries without keys or content are dropped on load.

---

## Woven Fate Editor

*(`Woven Fate Editor.html`)*

A form-based editor for **lorebooks and rulebooks**. It is a separate app on purpose: it never edits a game in progress, so a half-finished rulebook can't break a running story. You build the file here, save it, and load it in the game.

Switch between the **Lorebook** and **Rulebook** tabs at the top. Each keeps its own document, so you can work on both at once.

### Top bar and file handling

- **New** starts a fresh document. **Open…** loads a `.json` file, and the editor detects whether it is a lorebook or a rulebook.
- **Download** saves the file. Rulebooks are saved as `<rulebook id>.json` and lorebooks under the lorebook's name. If there are errors you are asked before saving.
- **JSON** shows the raw file. You can edit it and apply it, which is the way to repair a rulebook that the forms refuse to open.
- **Light / Dark** toggles the theme.
- The status bar shows the number of **errors** and **warnings**. Click it to see the list, and click an item to jump to the place it refers to.
- Your work is kept as a **draft** in the browser while you edit, and the editor offers to restore it next time. Save a file before closing the tab, and the editor warns about unsaved changes.

**Exports match the game's own format exactly.** Opening a file and saving it without changes gives the same file byte for byte (apart from the `exported_at` timestamp, which is refreshed). New fields are placed in the order the rulebook creator uses, and existing ones stay where they were. Only a rulebook that passes the schema check opens in the forms.

### Lorebook tab

- A list of entries with **search** (by name or key) and a **priority filter**, and buttons to add entries or **sort by priority**.
- Per entry: name, priority, keys (one per line), and content, with a character count. Move, copy, or remove entries.
- **Lorebook settings**: name, description, and character budget.
- Warnings that mirror what the game does: entries without keys or content (dropped), unusable keys, over-long content or names, bad priorities, and duplicate names.
- **Try it:** paste some story text to see which entries would trigger, in what priority order, and how much of the budget they use. Entries that don't fit are marked as skipped. (For many scenes and statistics, use the [Testbench](#lorebook-testbench).)

### Rulebook tab

The left side lists the sections; the right side is the form.

- **Details:** name, file id, and description, with counters for meters, HUD-strip meters, and combined rules.
- **Meters:** add (with a chooser that describes each type), reorder, copy, or remove. Each meter shows an error or warning badge when something is wrong.
  - Common settings: name, id, type, what the AI watches, visibility and HUD placement, icon picker, colour swatches, *Player can edit this*, *Tell the narrator the exact value*, and what raises or lowers it.
  - **Changing the id** updates every rule that refers to it, and so does changing an item's id. A new meter's id follows its name until you set it yourself.
  - **Changing the type** keeps the shared settings and asks before removing settings that only belong to the old type.
  - Type-specific forms: levels (name, limit, instruction, caption, shown, override), keyword rules, AI checks (questions, limits, extract mode, optional "only when" conditions), triggers, clock settings (mode, start, step, skips, level sets), dice and their roll conditions, and the item and rule lists for Select, Collection, and Journal.
  - A **condition builder** whose choices follow the meter type, and a **change** editor (adds, sets, multiplies, divides, random amount, keep between) with an optional "× another meter's value".
- **Combined rules:** conditions plus an instruction and caption.
- **Preview & test:** set every meter to any value (sliders, dates and times, dice results, items, events) and see the cards as the player sees them and the exact text the narrator would receive. This uses the game's own code, so it matches play. It never changes your rulebook.

Some checks go beyond the schema, for example: a trigger can't watch the meter it changes (so triggers need a second meter), there can be only one story clock, and references to meters or items that don't exist are errors.

---

## Lorebook Converter

*(`Lorebook Converter.html`)*

Converts lorebooks exported from other tools into the Woven format. Per its interface, it targets SillyTavern, NovelAI, JanitorAI, Agnai, SpicyChat, and similar JSON exports.

**Usage:** drop a `.json` lorebook onto the window (or click to browse), check the preview, and press **Download Converted JSON**. The output is saved as `<original name>.woven.json`.

What it does:

- Detects the shape of the input (object-keyed entries as in SillyTavern, or array-based entries)
- Merges primary and secondary keys into one de-duplicated key list
- Skips entries that are disabled
- Maps the source's priority or insertion order onto the game's **1–10** scale (1 is highest priority). Values already between 1 and 10 pass through unchanged. Larger order-style values (higher means stronger) are rescaled smoothly on a log scale: 100 becomes 5, 200 becomes 3, and anything from about 630 up becomes 1. Entries with no usable priority get **5** in every source format.
- Carries over the name, description, and budget (defaults to **8,000** characters if the source has none)

Things to check after converting:

- **Review the result.** Formats vary a lot between tools, and the converter only reads common field names. The Editor is a good place to tidy it up.
- **Entries need at least one key.** The game discards entries with no keys, so "always-on" or "constant" entries from other tools won't carry over as always-on. Give them keys, or fold them into the story setup.
- **Budget.** A budget found in the source file is carried over as-is (a source `token_budget` is copied as a character count, not converted). Otherwise the converter writes 8,000, matching the game's own default. You can change it in the game's lorebook dialog or the Editor, and the Testbench can help you pick a value.
- **Priorities from other tools.** Rescaled order-style values are approximate, so skim the results and adjust any entries that matter. A source value of exactly 10 or less is assumed to already be a 1-10 priority, so it is not rescaled.
- Conversion runs locally. Your file is not uploaded anywhere.

---

## Lorebook Test Editor

*(`Lorebook Test Editor.html`)*

A simple editor for building the scene files that the Testbench loads, so you don't have to hand-write the Markdown format. Its limits mirror the Testbench's, so a file that passes here should also load there. Like the other tools, it runs entirely offline in your browser.

### Using it

- **New** starts an empty file, **Load…** opens an existing `.md` or `.json` scenes file, and **+ Add scene** inserts a scene after the one you have selected.
- The left-hand list shows each scene's id, title, and "tests" note, with a search box to filter by id, title, setting, cast, or tests. A marker flags scenes with errors or warnings.
- Select a scene to edit its **Id**, **Title**, **Setting**, **Tests** (which entries or behaviours the scene should exercise), **Cast**, and the **scene text** itself. A live character and word count sits above the text.
- Per scene you can **Move up**, **Move down**, **Duplicate**, or **Delete**.
- **Declared length** is optional metadata. **Match text** sets it to the actual text length, and if you leave it at 0 it is filled in when you save.
- **Save .json** and **Save .md** download the file, and Ctrl/Cmd+S saves JSON. **Theme** toggles light and dark, sharing its setting with the other tools.

### Built-in checks

Each scene is validated as you type, and the status bar shows the scene count and total characters against the Testbench's limits (100 scenes, 400,000 characters in total).

| Level | Examples |
|---|---|
| **Error** | Missing or duplicate id, empty scene text, text over 20,000 characters, a single word longer than 100 characters |
| **Warning** | A title, setting, tests, or cast field over 2,000 characters (it would be truncated), an id over 100 characters |
| **Info** | An id that isn't of the form `sceneN`, a declared length that doesn't match the text, a missing "tests" note |

Saving with errors, or with more than 100 scenes, asks for confirmation first, since the Testbench would reject or skip those scenes.

### Things to know

- **Scene ids and Markdown:** the Testbench's Markdown reader only recognises ids shaped like `scene1`, `scene2`, and so on. Other ids are fine in `.json` but are skipped from a `.md` save, and you are warned before saving.
- **No autosave:** nothing is stored between sessions apart from the theme. Save your file before closing the tab. The editor warns you about unsaved changes when you load a file, start a new one, or leave the page.
- **Loading:** files over 2 MB are refused, matching the Testbench. JSON files must be an array of scene objects.

---

## Lorebook Testbench

*(`Lorebook Testbench.html`)*

A deterministic simulator for answering "will my lorebook actually do what I expect?" without spending any API calls. It reproduces the game's keyword matching and budget selection, then shows how entries compete for space. It tests **lorebooks only**; for rulebooks use the Editor's Preview & test page.

It opens with a small embedded sample (the *Atheron* lorebook and one test scene). Use **Reset to embedded data** to return to it.

### Loading your own data

- **Load lorebook (.json):** the Woven format
- **Load test scenes (.md / .json):** pieces of sample text to scan. You can write these by hand or use the [Test Editor](#lorebook-test-editor).

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

## Privacy and storage

- API targets (including **API keys**), saves, session state, and theme preferences are stored in your browser's `localStorage`. They never leave your machine except in requests to the API service you configured.
- Your lorebooks, rulebooks, and story text *are* sent to that service as part of prompts. Rulebooks with AI checks send a short extra request after each message.
- Clearing site data for the page removes your keys and saves. Use **Export** on a save to keep a backup.
- Individual saves are limited to roughly 4.8 MB.
- Both game files use the same storage namespace (`woven_fate::`), so when they are served from the same browser origin they can see each other's saves and API targets. Browsers treat `file://` pages differently, so if saves don't appear in the other version, use **Export** and **Import**.
- The Editor stores only unsaved drafts and the theme in your browser; nothing is sent anywhere.

## Requirements

A current desktop or mobile browser (Chrome, Edge, Firefox, Safari) and, for the game, an API key or a reachable local model endpoint. The Editor, Converter, Test Editor, and Testbench need no API access and work offline.
