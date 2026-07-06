# ReadEra PC

A browser-based desktop ebook reader — inspired by ReadEra, built for your own PDF/EPUB collection.
Runs locally on your machine. No account, no cloud, your books stay on your disk.

## Requirements

- **Node.js 22.5.0 or newer** (check with `node -v`) — this app uses Node's built-in SQLite module, so there's nothing extra to compile or install for the database.

## Setup

```bash
cd readera-pc
npm run setup
```

This installs both the backend and frontend dependencies. No native build tools (Visual Studio, Xcode, etc.) are required — everything here is pure JavaScript plus Node's built-in SQLite.

## Run it

```bash
npm run dev
```

Then open **http://localhost:5173** in your browser. Leave the terminal running — this *is* your server. You'll see a one-line "SQLite is an experimental feature" warning on startup — that's expected and harmless.

## Troubleshooting

**"npm install" fails partway through, or a previous install left things in a broken state (Windows)**
Close any editor, terminal, or antivirus scan that might have a file open inside the project folder, then delete `node_modules` folders manually and retry:
```powershell
Remove-Item -Recurse -Force node_modules, client\node_modules -ErrorAction SilentlyContinue
npm run setup
```

**"node:sqlite" / "--experimental-sqlite" errors on startup**
This means your Node version predates 22.5.0. Check with `node -v` and upgrade at https://nodejs.org (get the current LTS).

**"concurrently is not recognized" after running `npm run dev`**
This means `npm install` didn't finish successfully in the root folder (often because an earlier step failed). Re-run `npm run setup` from the project root and make sure it completes with no errors before trying `npm run dev` again.

## First-time use

1. Go to **Settings**
2. Either:
   - Click **"Choose files to import"** to upload individual PDF/EPUB files, or
   - Click **"Browse for folder"** to add a folder that gets watched automatically
3. Go to **Library** and hit **Scan** if books don't appear immediately
4. Double-click any book to start reading

## Feature status

| Feature | Status |
|---|---|
| Homepage (continue reading, favourites, recent) | ✅ |
| Sidebar navigation | ✅ |
| Library grid/list view, search, filters | ✅ |
| Import PDF | ✅ |
| Import EPUB | ✅ |
| Next/previous page | ✅ (buttons + arrow keys) |
| Zoom (PDF) | ✅ |
| Dark / light / sepia / console themes | ✅ |
| Remember last page | ✅ |
| Search inside book | ✅ |
| Highlight text | ✅ (both EPUB and PDF now — select text and it's auto-saved) |
| Notes on highlights | ✅ — click the note icon next to any highlight in the sidebar |
| Bookmarks | ✅ |
| Reading progress | ✅ |
| Book covers | ✅ Real covers — first page rendered (PDF) or embedded cover extracted (EPUB) and cached, generated automatically in the background the first time your library loads |
| Categories / tags | ✅ (basic — click the tag icon on a book) |
| Favourites | ✅ |
| Settings | ✅ |
| Multiple libraries | ✅ — switcher at the top of the Library page, per-folder and per-import library assignment in Settings, "move to library" action on each book |
| Sync reading progress | ⚠️ Not implemented as cloud sync. The server binds to `0.0.0.0`, so if you run it on one machine, other devices on the same Wi-Fi/LAN can reach it at `http://<your-computer-ip>:5173` and share the same library/progress. True multi-device sync (e.g. via a hosted database) is a bigger architecture decision — happy to scope it if you want it. |
| Dictionary | ✅ (double-click a word) |
| AI summary / chat with book | ✅ (needs your own Anthropic API key, added in Settings → AI Features) |
| Text-to-speech | ✅ (uses your browser's built-in voices via Web Speech API) |
| OCR for scanned PDFs | ✅ (basic — button runs Tesseract.js on the current page, shows extracted text in a popup; doesn't yet inject text back into the PDF for searching) |
| Export notes | ✅ (Markdown download, from the Bookmarks/Highlights sidebar) |
| Keyboard shortcuts | ✅ — ← / → page nav, +/- zoom, `b` bookmark, `Ctrl/Cmd+F` search, `Esc` close panels |
| PDF highlighting | ✅ Select text in a PDF and it's saved automatically, same as EPUB |
| Highlight notes & color | ✅ — click the note icon on a highlight to add/edit a note, click the color bar to cycle colors |

## Known limitations to be upfront about

- **MOBI**: routed through the EPUB renderer (epub.js). Classic `.mobi` files are a different binary format and will likely fail to open — AZW3/newer MOBI variants may partially work. True MOBI support needs a dedicated parser.
- **CBZ/CBR/DOCX/RTF/FB2/DJVU**: detected and catalogued in the library, but the reader shows an honest "not supported yet" screen rather than trying to render them.
- **PDF highlight positions**: stored as percentages of the page, so they stay aligned at any zoom level — but if the underlying PDF's text layout is unusual (multi-column, rotated pages), rect capture can occasionally be imprecise.
- **Cover generation**: happens once per book, the first time your library loads with that book missing a cover. It runs in the browser (rendering PDF page 1 / reading the EPUB's embedded cover), so a very large library will take a few seconds per book the first time.

## Project structure

```
readera-pc/
├── server/           # Express backend + SQLite
├── client/           # React + Vite frontend
│   └── src/
│       ├── components/
│       │   ├── Home/       # Dashboard
│       │   ├── Library/    # Grid/list, book cards
│       │   ├── Reader/     # PDF/EPUB viewers, AI, search, dictionary
│       │   ├── Settings/
│       │   └── Stats/
│       └── store/          # Zustand global state
└── data/             # SQLite DB lives here (auto-created)
```
