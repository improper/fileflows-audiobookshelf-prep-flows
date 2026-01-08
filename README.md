FileFlows Audiobookshelf Prep Flows
===================================

![Processed books example](documentation/assets/processed-books.png)

## Meet the flows
Flows that prep book folders/files (ebooks and audiobooks) for Audiobookshelf: clean up, enrich metadata, convert ebooks to EPUB, and place audio/ebook assets into your library structure.

### High-level summary
1. Intake + workspace prep: if the drop is a single folder inside another, pull the inner folder out and rerun it <img src="./resources/Recycle.png" alt="Recycle icon" width="14" height="14">; if it's only zip/rar junk, log it, mark it for deletion <img src="./resources/trash.png" alt="Trash icon" width="14" height="14">, and stop.
2. Derive title/author hints from folder/file names, then fetch/enrich metadata (Google Books, Open Library, optional OpenAI).
3. Build author/series folders and convert ebooks to EPUB where needed.
4. Move/rename into the Audiobookshelf layout and tidy the workspace.

### Entry flows
- `📒 Books by directory.json` (folder-triggered): takes a folder drop, prepares the workspace, enriches metadata, and shelves it in your Audiobookshelf layout.
- `📒 Books by file.json` (file-triggered): takes a single book file, fetches/enriches metadata, converts to EPUB if needed, and shelves it in your Audiobookshelf layout.

#### Subflows
`Subflow prep book workspace`, `Subflow set book from folder`, `Subflow fetch Google book info`, `Subflow find book series from Google info`, `Subflow set book from search`, `Subflow query OpenAI for book info`.

## Why use this
- Automates metadata lookup (Google Books, Open Library, optional OpenAI) and EPUB conversion.
- Normalizes author/series/book folders for Audiobookshelf.
- Handles both folder and single-file drop scenarios.

## Prerequisites
- Plugins Book and Web installed/enabled (Settings > Extensions > Plugins); provides nodes like `FileFlows.Book.EBookConverter` and Web Request.
- Variables (Settings > Extensions > Variables):
  - `audiobookshelfApiToken`
  - `audiobookshelfApiUrl` (e.g., `https://bookshelf.my.app/audiobookshelf/api/`)
  - `AudiobookshelfLibraryPath` (e.g., `/media/youruser/library/Books/Books/` with trailing slash)
- OpenAI (optional): get a key at https://platform.openai.com/api-keys and set `openAISecretKey` (leave blank to skip OpenAI; flow still runs). Example key format: `eyJh....w8j-tM`.
- Resources: upload `resources/Recycle.png` as `Recycle` and `resources/trash.png` as `Trash` via Settings > System > Resources > Add +.

## Creating/importing the flows
Import in this order to satisfy subflow dependencies:
1) `Subflow query OpenAI for book info`
2) `Subflow fetch Google book info`
3) `Subflow set book from search` (depends on the OpenAI subflow)
4) `Subflow set book from folder`
5) `Subflow prep book workspace`
6) `Subflow find book series from Google info`
7) `📒 Books by file.json`
8) `📒 Books by directory.json`

## Configure after import
- Library paths:
  - `📒 Books by directory.json` → node `Move eBook Directory`: set `DestinationPath` to your Audiobookshelf books path.
  - `📒 Books by file.json` → node `Set: authorDirectory`: update the base path for authors.
  - Both flows use `finalDestinationDirectory` variables; ensure they match your layout.
- Optional: confirm `AudiobookshelfLibraryPath` matches your environment.

## Set up libraries/watchers
- Create two FileFlows Libraries (titles shown in the screenshot) to start using the flows:
  - `Watch for Book Directories`: watches your book download folder for incoming folders and triggers `📒 Books by directory`.
    - Ensure the library is set to watch folders only. Reference: ![Watch folders only](documentation/assets/how-to-watch-folders-only.png)
  - `Watch for Book Files`: watches the same download folder for files and triggers `📒 Books by file`; restrict extensions to `m4b`, `wav`, `mobi`, `mp3`, `epub`, `flac`, `pdf`, `wma`, `cbr`, `m4a`, `cbz`, `aac`, `azw3`.
- Reference: ![Library watcher setup](documentation/assets/library-setup.png)
- Allowed extensions example: ![File extensions to watch](documentation/assets/file-extensions-to-watch.png)

## How it runs
- Folder-triggered (`📒 Books by directory.json`):
  - Prep workspace and detect unwanted archives.
  - Derive book metadata from folder name; validate mp3 layout; convert ebooks to EPUB.
  - Fetch metadata from Google Books; enrich series info from Open Library; build author/series folders.
  - Copy/rename into the Audiobookshelf structure and clean up the source.
- File-triggered (`📒 Books by file.json`):
  - Set search title from the file name; fetch metadata via Google Books/OpenAI.
  - Create author/series folders, convert to EPUB, move/rename into place.

## Usage
- Point a FileFlows file/folder watcher at your incoming source and assign the appropriate entry flow (folder vs file).
- If import errors mention missing subflows, import dependencies first, then the parent.
- Detecting books from folder structures is tricky because sources are inconsistent; if you improve detection/enrichment, PRs are welcome.
