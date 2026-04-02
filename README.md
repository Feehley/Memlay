# Memlay

A self-contained, single-file interactive memory layout visualizer for embedded and firmware projects. No build step, no dependencies, no server — open `memlay.html` in a browser and start mapping.

-----

## Quick Start

1. Drop `memlay.html` anywhere — in your repo, your vault, your desktop
1. Open it in a browser
1. Edit the table; the diagram updates live

To embed in **Obsidian**:

```md
<iframe src="memlay.html" width="100%" height="500px" style="border:none"></iframe>
```

-----

## Features

### Live Memory Bar

The left panel renders a proportional vertical bar representing your address space. Segments are sized by their actual byte count relative to the total span. Gaps between regions are shown as hatched areas. Everything updates as you type — no save or refresh needed.

### Editable Table

The right panel is a fully editable table with one row per memory region. Each row has:

|Column   |Description                                                     |
|---------|----------------------------------------------------------------|
|Color    |Click the swatch to open the OS color picker                    |
|Offset   |Start address — hex (`0x400`), decimal, or suffixed (`4K`, `2M`)|
|Size     |Region size — same formats as Offset                            |
|Name     |Display label shown in the bar and tooltips                     |
|Reference|Optional URL (makes segment clickable) or note reference        |

Invalid offsets and sizes are highlighted red immediately.

### Gap Detection

Unallocated space between regions is automatically detected and rendered as a diagonal-stripe hatched zone in the bar. Hovering a gap shows its start address, end address, and total size. The header stats line shows total free bytes across all gaps.

### Overlap Detection

If any two regions overlap, the tool:

- Highlights both segments in the bar with a red diagonal stripe pattern
- Marks the affected table rows with a red background and `!` badge
- Shows a warnings panel below the table listing every conflicting pair with overlap size and start address
- Displays a pulsing `⚠ N` badge in the header

### Zoom

In proportional mode, clicking any segment zooms the bar to focus on that region (with 15% padding either side). Address ticks re-render against the zoomed range. An amber border on the bar indicates zoom is active. Click **Reset** or press **Escape** to zoom back out.

### Ruler Mode

Toggle **Ruler** in the viz panel to switch from proportional to equal-height rows. Every segment gets the same height regardless of size, so a 256-byte config region is just as readable as a 4 MB flash partition. In ruler mode, each row shows the region name and its offset + size as a sub-label. Gaps get their own labelled rows.

### Address Lookup

The **Lookup** field at the bottom of the viz panel accepts any address format. As you type:

- The containing segment is highlighted with a glowing blue outline
- All other segments are dimmed
- A crosshair line appears on the bar at the exact address
- The matching table row is highlighted and scrolled into view
- The result shows the region name and byte offset within it (e.g. `FIP0  +0x0620 of 16 KB`)
- Gap addresses are identified as unallocated
- Out-of-range addresses report clearly

Press **Escape** to clear the lookup.

### Memory Budget Bar

Set your device’s total memory in the **Budget** field (accepts `1M`, `512K`, `0x100000`, etc.). A utilization bar appears showing percentage used:

- Green — under 85%
- Amber — 85–100%
- Red + `⚠ OVER` — mapped regions exceed the budget

The budget persists across reloads.

### Named Presets

Save and switch between multiple named memlays — useful for separate Flash and RAM layouts, or different board variants.

- **Dropdown** — switch between saved presets (auto-saves current before switching)
- **Save** — save current state under a new or existing name
- **×** — delete the current preset (disabled when only one exists)

Presets persist in `localStorage`. Each preset stores its own complete set of rows independently.

### Document Title

Click the title in the header to rename the document. The title is used as the filename for PNG and CSV exports. Persists across reloads.

### CSV Import / Export

**Export CSV** downloads a `<title>.csv` file with columns:

```
Offset, Size, Name, Reference, Color
```

**Import CSV** opens a modal where you paste CSV data. Rules:

- Header row is optional and auto-skipped if present
- Offset and Size accept hex, decimal, or suffixed values
- Reference and Color columns are optional
- Colors must be 6-digit hex (`#539bf5`); missing colors get palette defaults
- Per-row error messages tell you exactly which row failed and why

### Export PNG

**Export PNG** captures the full tool (header + viz + table) at 2× resolution for sharp docs. The screenshot:

- Hides delete buttons and the export button itself during capture
- Uses the correct background color for both dark and light themes
- Saves as `<title>-<timestamp>.png`

Requires an internet connection on first use to load `html2canvas` from CDN. Subsequent uses in the same session are cached.

### Dark / Light Mode

Toggle with the **☾ / ☼** button in the header. The theme is applied to every element including the bar, tooltips, inputs, and warnings panel. Preference is saved to `localStorage`.

### Persistent State

Every change — edits, color picks, row additions, deletions — is saved to `localStorage` immediately. Closing and reopening the file restores exactly where you left off, including the active preset, budget, and title.

-----

## Address Formats

All numeric fields accept the same formats:

|Format   |Example     |Value  |
|---------|------------|-------|
|Hex      |`0x1400`    |5120   |
|Decimal  |`4096`      |4096   |
|Kilobytes|`4K` or `4k`|4096   |
|Megabytes|`2M` or `2m`|2097152|

-----

## Keyboard Shortcuts

|Key               |Action                                        |
|------------------|----------------------------------------------|
|`Escape`          |Close import modal / clear zoom / clear lookup|
|`Enter` (in title)|Confirm title edit                            |

-----

## Using in Obsidian

Place `memlay.html` inside your vault. Embed with an iframe in any note:

```md
<iframe src="memlay.html" width="100%" height="520px" style="border:none;border-radius:6px"></iframe>
```

**Notes:**

- Set the `src` path relative to the note, or use an absolute vault path
- If the iframe appears blank, enable **Allow iframes** in Obsidian’s settings, or install the **HTML Reader** community plugin
- The Export PNG button requires an internet connection the first time (loads `html2canvas` from CDN). If Obsidian’s sandbox blocks this, download [`html2canvas.min.js`](https://cdnjs.cloudflare.com/ajax/libs/html2canvas/1.4.1/html2canvas.min.js) into your vault and replace the CDN URL in the `<script>` tag near the bottom of `memlay.html`

-----

## Using in a Firmware Repository

The recommended workflow is to commit `memlay.html` alongside your source and update the table as your linker script evolves.

### With Make

```makefile
docs:
    open docs/memlay.html   # macOS
    # xdg-open docs/memlay.html  # Linux
```

### With a Build Script

If you want to auto-generate the initial table from a linker map file, parse the `.map` output into CSV and use the **Import CSV** feature to load it. The exported CSV can then be checked into source control separately from the HTML:

```
flash.csv  →  [Import CSV into memlay.html]  →  view / edit / export PNG
```

-----

## Data Storage

All data is stored in the browser’s `localStorage` under these keys:

|Key             |Contents                        |
|----------------|--------------------------------|
|`memmap-presets`|JSON object of all named presets|
|`memmap-current`|Name of the active preset       |
|`memmap-title`  |Document title string           |
|`memmap-budget` |Budget field value (raw string) |
|`memmap-theme`  |`"light"` or `"dark"`           |

`localStorage` is scoped to the file’s origin. If you open the file via `file://`, data is stored per file path. If you serve it via a local web server, data is stored per origin.

To reset everything, open the browser console and run:

```js
['memmap-presets','memmap-current','memmap-title','memmap-budget','memmap-theme'].forEach(k => localStorage.removeItem(k))
```

Then reload.

-----

## Limitations

- **localStorage only** — data does not sync across devices or browsers
- **No file I/O** — changes are not written back to the `.html` file itself; use Export CSV to save data portably
- **Export PNG requires CDN** — first-time screenshot needs `html2canvas` from `cdnjs.cloudflare.com`; see the Obsidian section above for offline workaround
- **Single file** — all state lives in one browser tab; opening the same file in two tabs simultaneously may cause one tab’s saves to overwrite the other’s

-----

## Feature Summary

|Feature                        |Status|
|-------------------------------|------|
|Live proportional bar          |✅     |
|Gap detection + hatching       |✅     |
|Overlap detection + warnings   |✅     |
|Per-region color picker        |✅     |
|Zoom to segment                |✅     |
|Ruler (equal-height) mode      |✅     |
|Address lookup with crosshair  |✅     |
|Memory budget bar              |✅     |
|Named presets                  |✅     |
|Editable document title        |✅     |
|CSV import                     |✅     |
|CSV export                     |✅     |
|Export PNG (2×)                |✅     |
|Dark / light theme             |✅     |
|Persistent state (localStorage)|✅     |
|Hover tooltips                 |✅     |
|Clickable reference links      |✅     |
|Keyboard shortcuts             |✅     |