# Photo Culler

A single-page web app for cleaning up after a cull. You point it at your **culled photos** and at the folder of **photos to be culled**. It keeps only the counterpart files (JPG ↔ ARW) of your keepers and moves everything else out of the way.

Typical use: you culled the JPGs and now want to keep only the matching ARW raw files, or the other way round.

## Quick start

1. Open `index.html` in **Google Chrome, Microsoft Edge, Brave, Arc, Opera or Vivaldi** on a desktop computer.
2. **Step 1, Culled photos:** drop a folder onto the left box, or click anywhere in it to choose one. This folder is only read, never changed.
3. **Step 2, Photos to be culled:** do the same in the right box. Click **Allow** when the browser asks for permission to edit files.
4. **Step 3, Review and clean up:** check what will be kept and removed, how much space that is, and why each file goes. Switch to **Thumbnails** to spot-check the photos. Read any warnings.
5. Click **Move to _rejected folder** and confirm. Changed your mind? Click **Undo move**.
6. Look through `Photos to be culled/_rejected/` in Finder or File Explorer. When you're happy, drag the `_rejected` folder to the Trash or Recycle Bin.

No installation, internet connection or other files are needed. `index.html` works on its own and can be copied to any computer.

## Matching rules

- **Comparing names:** file names are compared **without the extension** and **ignoring upper/lower case**, so `DSC01234.JPG` matches `dsc01234.arw`.
- **Culled JPG:** if "Culled photos" contains `DSC01234.jpg`, only `DSC01234.arw` is kept.
- **Culled ARW:** if "Culled photos" contains `DSC01234.arw`, only `DSC01234.jpg` (or `.jpeg`) is kept.
- **Everything else is removed:** every other JPG/ARW file in "Photos to be culled" goes, including one in the same format as the culled file (for example `DSC01234.jpg` when you culled `DSC01234.jpg`).
- **Mixed folders:** if "Culled photos" has both JPGs and ARWs, the rule is applied name by name.
- **Sidecar files** (`.xmp`, `.dop`, `.pp3`): with **Also remove sidecar files** ticked (the default), a sidecar goes with its photo. `DSC01234.ARW.xmp` follows `DSC01234.ARW`. `DSC01234.xmp` is removed only if a `DSC01234` photo is removed and none is kept. Sidecars of kept photos, and sidecars with no photo at all, are left alone.
- **Left alone:** other file types (`.txt`, `Thumbs.db` and so on) and hidden system files (`.DS_Store`, `._DSC01234.ARW`).
- **Top level only:** only files directly inside each folder are considered. Subfolders are ignored.

### Example

| Culled photos | Photos to be culled | Result |
|---|---|---|
| `DSC0001.JPG` | `DSC0001.ARW` | kept |
| | `DSC0001.JPG` | removed (same format as the culled file) |
| | `DSC0002.ARW` | removed (not in culled photos) |
| `DSC0003.JPG` | `DSC0003.ARW` | kept |
| | `DSC0001.xmp` | left alone (sidecar of the kept `DSC0001.ARW`) |
| | `DSC0002.xmp` | removed with `DSC0002.ARW` (sidecar option on) |

## Remove modes

| Button | What happens |
|---|---|
| **Move to _rejected folder** (recommended) | Removed files are moved to a `_rejected` subfolder inside "Photos to be culled". Nothing is deleted, so you can review them and trash the folder yourself. If a file with the same name is already there, the new one is saved as `name (1).ext`. **Undo move** puts every file back under its original name, and removes `_rejected` again if the app created it and it's now empty. |
| **Delete permanently** | Files are deleted right away. They **do not go to the Trash** and cannot be recovered. This button is turned off when none of your culled photos match. |

Browsers can't use the system Trash or Recycle Bin, which is why the `_rejected` folder exists.

## Long runs

- **Progress:** a progress bar shows files and gigabytes done and the estimated time left. The browser tab title shows the percentage, so you can switch to other work.
- **Cancel:** stops after the current file. Anything already moved can still be undone.
- **Closing the tab:** the browser asks you to confirm if you try to close or reload the tab mid-run.

## Safety checks

- **You always get a preview:** it lists what will be kept and removed, with file counts and sizes, before anything changes. A sentence explains what the app detected, for example "Your culled photos are JPGs, so only the matching ARW files are kept."
- **Warnings for likely mistakes:**
  - **Nothing matched:** none of the culled photos has a counterpart, which usually means the wrong folder. Delete permanently is turned off in this case.
  - **Few matches:** fewer than half of the culled photos have a counterpart.
  - **Same name:** both folders have the same name, such as two `DCIM` folders. Browsers only show folder names, not full paths.
- **Folders are re-read automatically:** whenever you come back to the tab, and again right before files are moved or deleted, so the app always acts on what is on disk.
- **You confirm first:** a confirmation dialog shows the number of files, their size and any warnings.
- **Buttons are disabled when needed,** with a line underneath explaining why: "Culled photos" contains no JPG/ARW files, the same folder is chosen twice, or there is nothing to remove.
- **"Culled photos" is never changed.**
- **Moves are checked:** when the browser can't move a file directly, it copies it, checks the size of the copy and only then removes the original.
- **Errors appear where they happen:** for example, dropping a file instead of a folder shows a message inside that box.

## Browser support

| Browser | Supported |
|---|---|
| Chrome, Edge, Brave, Arc, Opera, Vivaldi (desktop: Mac, Windows, Linux, ChromeOS) | ✅ |
| Safari | ❌ |
| Firefox | ❌ |
| Any browser on iPhone, iPad or Android | ❌ |

The app relies on the [File System Access API](https://developer.mozilla.org/docs/Web/API/File_System_API), which only Chromium-based desktop browsers support. In an unsupported browser, the folder boxes are greyed out and a banner names that browser and says what to use instead.

**Thumbnails:** JPG thumbnails are made from the files themselves. ARW thumbnails use the preview image that the camera stores inside the raw file. If a file has no readable preview, its thumbnail shows "No preview".

On company-managed computers, IT policy may block this API. If **Choose folder** does nothing, that is the likely cause.

## Running from a local server (optional)

Opening `index.html` directly should work. If your browser refuses to open folders from a local file, serve it instead:

```bash
cd photo-culler
python3 -m http.server 8765
```

Then open <http://localhost:8765>. Python 3 comes with macOS. On Windows, install it from [python.org](https://www.python.org/downloads/) or the Microsoft Store.

## Tests

`tests.html` runs 147 automated tests against the real app. They use throwaway folders in the browser's private sandbox storage, so your own files are never touched. The tests cover:

- **Matching:** both directions (JPG → ARW, ARW → JPG), `.jpeg`, mixed case, dots in names, hidden and non-photo files, and sidecar files on and off
- **Folder boxes:** clicking anywhere in a box, Change folder, Clear, drag-highlighting, and error messages shown in the box
- **Sizes and formatting:** file sizes, thousands separators, singular/plural wording, time estimates, Finder vs File Explorer wording, and browser detection
- **Moving to `_rejected`:** direct move, copy fallback, name clashes, content preserved, and re-runs
- **Undo:** full, after a name clash, and after a cancelled run
- **Long runs:** progress text, tab title, Cancel, and the warning when closing the tab
- **Permanent delete**
- **Automatic rescans:** new files are picked up, open lists stay open, and a folder that disappears is reported
- **Warnings and safety checks:** nothing matched, few matches, same folder name, no photos in the culled folder, and the same folder picked twice
- **Thumbnails:** JPG, ARW embedded preview, a broken file, and "Show more" paging
- **Large folders:** a 1,005-file folder

The tests need a local server, because browsers block them on a double-clicked file:

```bash
cd photo-culler
python3 -m http.server 8765
```

Then open <http://localhost:8765/tests.html>. The top line should read **147/147 passed**. Keep the tab in front while the tests run.

## Files

| File | Purpose |
|---|---|
| `index.html` | The app (self-contained) |
| `tests.html` | Automated tests |
| `README.md` | This file |
