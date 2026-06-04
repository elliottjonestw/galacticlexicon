# 星際大典 — Star Wars English–Chinese Dictionary

A lightweight, single-page web dictionary of Star Wars terminology, providing English to Traditional Chinese translations sourced directly from official subtitles. Supports both Taiwan (繁體中文－台灣) and Hong Kong (繁體中文－香港) variants of Traditional Chinese.

---

## Features

### Live Search
- Instant results as you type — no page reloads.
- Search works across **all three language columns simultaneously** (English, TW Chinese, HK Chinese), regardless of which languages are currently selected for display.
- **Accent-insensitive** English search: typing `Padme` will find `Padmé`, `Qui-Gon` finds `Qui-Gon Jinn`, etc.

### Smart Search Ranking
Results are ranked by match quality, closest match first:

| Priority | Match type | Example query `jedi` |
|----------|-----------|----------------------|
| 1st | Exact match | `Jedi` |
| 2nd | Starts with query | `Jedi Council`, `Jedi Temple` |
| 3rd | Word within term starts with query | `Master (Jedi title)` |
| 4th | Contains query anywhere | — |

### Language Selector
Two independent dropdowns — **語言一** (Language 1) and **語言二** (Language 2) — let users choose which languages to display for each card. Each dropdown offers:

- `English`
- `繁體中文－台灣`
- `繁體中文－香港`

Entries that do not have data for one of the selected languages are automatically hidden from results, ensuring only complete translation pairs are shown.

### Translation Source
Every entry displays its translation source in the format:

```
翻譯來源：[Source Title] @ [Timestamp]
```

This records the exact piece of Star Wars media and the timestamp at which the Chinese translation first appears, providing verifiable sourcing for every term.

### Randomised Order
On each page load, all entries are displayed in a **random order** (Fisher–Yates shuffle). This makes browsing feel fresh and encourages discovery of unfamiliar terms.

### Mobile-Friendly
The site is fully responsive:
- Single-column card layout on all screen widths.
- Search bar and language dropdowns are touch-friendly.
- Typography and spacing are tuned separately for mobile and desktop via CSS media queries.

---

## Dictionary Data (`dictionary.csv`)

The dictionary is stored as a plain CSV file with the following columns:

| Column | Description |
|--------|-------------|
| `English` | The Star Wars term in English |
| `Traditional Chinese (Taiwan)` | Official Taiwan Traditional Chinese translation |
| `Traditional Chinese (Hong Kong)` | Official Hong Kong Traditional Chinese translation (may be empty for some entries) |
| `Source Content` | The Star Wars title (in Traditional Chinese) where the translation originates |
| `Timestamp` | Timecode (`HH:MM:SS`) of the subtitle in the source content |

### Current Coverage

**456 total entries** drawn from the following Star Wars titles:

**Films**
- 威脅潛伏 — *The Phantom Menace*
- 複製人全面進攻 — *Attack of the Clones*
- 西斯大帝的復仇 — *Revenge of the Sith*
- 曙光乍現 — *A New Hope*
- 帝國大反擊 — *The Empire Strikes Back*
- 絕地大反攻 — *Return of the Jedi*
- 俠盜一號 — *Rogue One*
- 韓索羅 — *Solo*
- 原力覺醒 — *The Force Awakens*
- 最後的絕地武士 — *The Last Jedi*
- 天行者的崛起 — *The Rise of Skywalker*

**Television Series**

| Title | Episodes covered |
|-------|-----------------|
| 複製人之戰 (*The Clone Wars*) | Feature film |
| 反抗軍起義 (*Rebels*) | S01E01 |
| 瑕疵小隊 (*The Bad Batch*) | S01E01 |
| 曼達洛人 (*The Mandalorian*) | S01E01 |
| 安道爾 (*Andor*) | S01E01 |
| 亞蘇卡 (*Ahsoka*) | S01E01 |
| 侍者 (*The Acolyte*) | S01E01 |
| 暗影之王 | S01E01 |
| 絕地小武士大冒險 (*Young Jedi Adventures*) | S01E01 |
| 骨幹小隊 (*Skeleton Crew*) | S01E01 |

### Adding or Updating Entries

Open `dictionary.csv` in any spreadsheet application or text editor. Each row must have all five columns. The `Traditional Chinese (Hong Kong)` column may be left empty if no Hong Kong translation exists — entries with empty fields will be hidden whenever that language variant is selected in the UI.

The site re-reads the CSV on every page load, so no rebuild step is needed after edits.

---

## Project Structure

```
starwars_en-zh_dictionary/
├── index.html        # Entire site — markup, styles, and JavaScript
├── dictionary.csv    # All dictionary data
└── README.md         # This file
```

The project has **zero external dependencies**. All CSS and JavaScript is embedded directly in `index.html`.

---

## Running the Site

Because `index.html` fetches `dictionary.csv` at runtime using the Fetch API, the files must be served over HTTP rather than opened directly as `file://` URLs (browsers block cross-origin fetch requests on the file protocol).

### Option 1 — Python (built-in, no install needed)

```bash
cd starwars_en-zh_dictionary
python3 -m http.server 8765
```

Then open [http://localhost:8765](http://localhost:8765) in your browser.

### Option 2 — Node.js (`npx serve`)

```bash
cd starwars_en-zh_dictionary
npx serve .
```

### Option 3 — Any static file host

Upload both `index.html` and `dictionary.csv` to any static hosting provider (GitHub Pages, Netlify, Vercel, etc.). No build step required.

---

## Design Notes

| Element | Value |
|---------|-------|
| Primary accent | `#a07800` (dark gold) |
| Background | `#f2f3f5` (light grey) |
| Card background | `#ffffff` |
| Font | System UI stack (`-apple-system`, `BlinkMacSystemFont`, `Segoe UI`, `Roboto`) |
| Search highlight | Amber tint `rgba(160, 120, 0, 0.15)` |
| Mobile breakpoint | `480px` |

---

## Technical Details

### CSV Parsing
The CSV is fetched as plain text and split on newlines and commas. Column positions are fixed (0–4), so commas within field values are not currently supported. All five columns are stored in memory; the UI reads the appropriate column index based on the user's language selection.

### Search Implementation
Search is performed client-side on the in-memory `DATA` array. For each entry, the query is tested against all three language columns regardless of the current display selection. Matching uses:
- **Latin scripts**: `String.normalize('NFD')` to strip diacritics, then lowercase comparison.
- **CJK scripts**: Direct substring match on the original string.

Scoring assigns a numeric priority (0–3) per field, and entries are sorted by their best score across all columns.

### Randomisation
On each page load, the parsed data array is shuffled using the [Fisher–Yates algorithm](https://en.wikipedia.org/wiki/Fisher%E2%80%93Yates_shuffle) before the first render. This shuffle order is preserved when filtering — only the sort-by-relevance pass (active during a search) overrides it.
