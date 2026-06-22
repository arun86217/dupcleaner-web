# DupCleaner Pro — Web Edition

Find and remove duplicate files directly in your browser. No install, no Python, no npm. Just open `index.html` in Chrome or Edge.

![DupCleaner Pro](https://img.shields.io/badge/platform-Chrome%20%7C%20Edge%20110%2B-6366f1?style=flat-square)
![License](https://img.shields.io/badge/license-MIT-22c55e?style=flat-square)
![Zero dependencies](https://img.shields.io/badge/dependencies-zero-f59e0b?style=flat-square)

---

## Quick start

```bash
git clone https://github.com/YOUR_USERNAME/dupcleaner-web.git
cd dupcleaner-web
# Then open index.html in Chrome or Edge
```

Or use the live version on GitHub Pages: `https://YOUR_USERNAME.github.io/dupcleaner-web`

---

## How it works

1. **Add sources** — click "+ Add folder" to grant access to one or more folders or drives
2. **Configure** — set an extension filter (e.g. `jpg,png`) or leave `*` for all files; edit the exclusion list to skip venv/node_modules/etc.
3. **Scan** — click Scan; files are grouped by size first (free), then SHA-256 hashed in parallel
4. **Act** — move duplicates to `.dupcleaner/duplicates/` (recoverable) or delete them permanently

Reports (JSON + CSV) are auto-saved to `<first source>/.dupcleaner/reports/` after every scan.

---

## Features

| Feature | Detail |
|---------|--------|
| **Multi-source** | Add multiple folders or entire drives as sources |
| **Exact deduplication** | SHA-256 hash — byte-for-byte match only |
| **Original selection** | Oldest file by modification time is kept; newer copies are duplicates |
| **Parallel hashing** | Web Workers — one per CPU core (`navigator.hardwareConcurrency`) |
| **Hash cache** | IndexedDB cache keyed on path + size + mtime — rescans are near-instant |
| **Exclusion list** | Skip `venv`, `node_modules`, `.git`, `__pycache__` and anything you add |
| **Extension filter** | Scan only `jpg,png,mp4` or leave `*` for everything |
| **Move duplicates** | Copies to `.dupcleaner/duplicates/` preserving folder structure, then deletes source |
| **Delete duplicates** | Permanent removal — confirm dialog required |
| **Reports** | JSON + CSV saved into `.dupcleaner/reports/` inside the scanned folder |
| **Zero install** | Pure HTML + JS, no build step, no server required |

---

## Browser requirements

| Browser | Support |
|---------|---------|
| Chrome 110+ | ✅ Full support |
| Edge 110+ | ✅ Full support |
| Firefox | ❌ `showDirectoryPicker` not supported |
| Safari | ❌ Not supported |

The app uses the [File System Access API](https://developer.mozilla.org/en-US/docs/Web/API/File_System_API) — specifically `showDirectoryPicker()`, `FileSystemDirectoryHandle`, and `FileSystemFileHandle.createWritable()`. These are Chrome/Edge only as of 2025.

---

## Output structure

All output is written inside the folder(s) you scan — nothing goes outside it.

```
<your-folder>/
└── .dupcleaner/
    ├── duplicates/          ← moved duplicates (preserves original structure)
    │   └── FolderName/
    │       └── subfolder/
    │           └── duplicate-file.jpg
    └── reports/
        ├── report.json
        └── report.csv
```

The `.dupcleaner/` folder is always excluded from future scans automatically.

---

## Report format

**report.json**
```json
[
  {
    "sha256": "a1b2c3...",
    "original": "Photos/2023/vacation/img.jpg",
    "duplicate": "Photos/backup/img.jpg",
    "size_bytes": 3145728
  }
]
```

**report.csv**
```
sha256,original,duplicate,size_bytes
a1b2c3...,"Photos/2023/vacation/img.jpg","Photos/backup/img.jpg",3145728
```

---

## Deploy to GitHub Pages

1. Push this repo to GitHub
2. Go to **Settings → Pages → Source** → select `main` branch, `/ (root)`
3. Your app is live at `https://YOUR_USERNAME.github.io/dupcleaner-web`

Users open the URL in Chrome, click "+ Add folder", and grant permission. No server-side code ever runs — everything happens in the browser.

---

## Architecture

Single file, zero dependencies, zero build step.

```
index.html
├── CSS          — dark theme, grid layout, component styles
└── JavaScript
    ├── Cache    — IndexedDB wrapper (path+size+mtime → sha256)
    ├── WorkerPool — inline blob Worker, one per CPU core
    ├── Scanner  — recursive directory walk via FileSystemDirectoryHandle
    ├── Finder   — size-group → parallel hash → duplicate groups
    ├── Actions  — move (copy+delete) / delete / report export
    └── UI       — state, rendering, event handling
```

The Web Worker is created from a Blob URL so it works with both `file://` and HTTP origins — no separate worker file needed.

---

## Comparison with Python version

| | Web Edition | Python Edition |
|--|-------------|----------------|
| Install | None — open in browser | `pip install streamlit` |
| Runs on | Chrome / Edge | Any OS with Python |
| Parallelism | Web Workers (cores) | ThreadPoolExecutor (cores) |
| Cache | IndexedDB | SQLite |
| Reports | Written to scan folder | Written to scan folder |
| Offline | ✅ | ✅ |
| Multi-drive | ✅ | ✅ |

---

## License

MIT — use it, fork it, ship it.
