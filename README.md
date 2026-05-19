# betterToC

[![vellum](https://img.shields.io/badge/vellum-bettertoc-purple)](https://vellum.delivery/#/package/bettertoc/)

[Video Walkthrough](https://www.youtube.com/watch?v=WlVVxxCql80)

Adds the ability to add, delete*, and edit* Table of Contents entries in notebooks, PDFs, and EPUBs.  
Supports cross-document linking.  
On-disk, ToC data is stored inside the UUID directory for the document in a toc.rm file.  
This syncs between devices via cloud sync.  
EPUB reflows can result in small amounts of drift due to limitations in progress calculation.

<table>
  <tr>
    <td><img src="assets/betterToc-pro.png" height="400"/></td>
    <td><img src="assets/betterToc-move.png" height="400"/></td>
    <td><img src="assets/betterToc-notebook.png" height="400"/></td>
    <td><img src="assets/betterToc-edit.png" height="400"/></td>
    <td><img src="assets/betterToc-ebook.png" height="400"/></td>
  </tr>
</table>

*Editing and deleting are limited to user-created ToC entries only

## Installation

Installation via the [Vellum package manager](https://github.com/vellum-dev/vellum) is recommended. Dependencies are handled automatically.

### Manual

Requires [xovi](https://github.com/asivery/rmpp-xovi-extensions) and qt-resource-rebuilder.

Download `betterToc.qmd` from the [latest release](https://github.com/rmitchellscott/xovi-bettertoc/releases/latest).

Copy to `/home/root/xovi/exthome/qt-resource-rebuilder/` and restart xovi.

## Companion extensions

- [betterTocCollapse](https://github.com/rmitchellscott/xovi-qmd-extensions) — adds chevrons to collapse/expand ToC entries based on indent
- [tocFromSelection](https://github.com/rmitchellscott/xovi-qmd-extensions) — create TOC entries from text selection

## rmhacks migration

rmhacks users can use the `scripts/migrate-bookmarks-to-toc.sh` script to migrate from `.bookm` to `toc.rm` format. `.bookm` files are retained. The script should be executed from the reMarkable tablet, and `chmod +x migrate-bookmarks-to-toc.sh` may be required.

## toc.rm Format

The `toc.rm` file stores table of contents data for a single document at:

```
/home/root/.local/share/remarkable/xochitl/{document-id}/toc.rm
```

The file has a fixed binary header followed by a JSON object containing entries and metadata.

### Entry Keys

Each key in the JSON identifies a TOC entry. There are four key formats:

| Key format | Example | Description |
|------------|---------|-------------|
| Page number | `"3"` | Native entry for a numbered page |
| Page UUID | `"af32c1d0-..."` | Native entry for a UUID-identified page |
| `p:{ratio}` | `"p:0.500000"` | Ebook content position — `ratio = pageNumber / (totalPages - 1)` |
| `toc:{id}` | `"toc:k7f2x9ab"` | User-created entry |
| `xref:{id}` | `"xref:m3p1q8yz"` | Cross-document link |

### Entry Objects

**Standard entry** (page-number, UUID, `p:`, and `toc:` keys):

```json
{
  "name": "Chapter 1",
  "level": 0,
  "targetPageId": "page-uuid-or-p:ratio",
  "afterEntry": "key-of-previous-entry"
}
```

| Field | Description |
|-------|-------------|
| `name` | Display title |
| `level` | Indent depth (0 = top-level) |
| `targetPageId` | Target page. Only present on `toc:` entries; for other key types the key itself identifies the page. |
| `afterEntry` | Key of the preceding entry in display order, or `null` for the first entry (linked-list ordering) |

**Cross-document entry** (`xref:` keys):

```json
{
  "name": "See other doc",
  "level": 0,
  "targetDocId": "document-uuid",
  "targetPageKey": "page-key-in-target-doc",
  "targetDocName": "Target Document Name",
  "afterEntry": "key-of-previous-entry",
  "pageDeleted": false
}
```

| Field | Description |
|-------|-------------|
| `targetDocId` | UUID of the linked document |
| `targetPageKey` | Entry key within the target document (page number, UUID, or `p:` ratio) |
| `targetDocName` | Cached display name of the target document |
| `pageDeleted` | `true` if the target page no longer exists |

### Metadata Keys

Underscore-prefixed keys at the top level control display options:

| Key | Values | Default |
|-----|--------|---------|
| `_numberStyle` | `"arabic"`, `"decimal"`, `"outline"` | absent (no numbering) |
| `_filterMode` | `"all"`, `"built-in"`, `"user"` | `"all"` |
| `_autoNumber` | `true` / `false` | Legacy; if `true`, treated as `_numberStyle: "arabic"` |

### Example

```json
{
  "_numberStyle": "arabic",
  "3": { "name": "Introduction", "level": 0, "afterEntry": null },
  "7": { "name": "Methods", "level": 0, "afterEntry": "3" },
  "toc:k7f2x9ab": { "name": "Results Summary", "level": 1, "targetPageId": "7", "afterEntry": "7" },
  "xref:m3p1q8yz": { "name": "See Appendix Doc", "level": 0, "targetDocId": "ab12cd34-...", "targetPageKey": "1", "targetDocName": "Appendix", "afterEntry": "toc:k7f2x9ab", "pageDeleted": false }
}
```
