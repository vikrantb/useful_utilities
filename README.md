# useful_utilities

Single-file dev utilities. Open the HTML in a browser; nothing else.

## md_viewer.html — Markdown viewer with inline comments

Local-first Markdown / Mermaid viewer that lets you **right-click any block to comment**, save the comments **inline in the source file**, and round-trip the file with an AI agent.

### What it does

- Renders `.md` / `.mmd` files (Markdown, GFM, Mermaid blocks, syntax highlighting)
- Right-click a heading / paragraph / list-item / blockquote / code-block → "Comment on this"
- Comments support **markdown body** (links + images via URL field)
- Each comment carries a **timestamp** + **author** identity (`vikrant` or `ai`)
- **Reply** to any comment; threads are 2-deep
- **Resolve** / reopen comments
- Right rail shows all threads on the current document; click a card to jump
- "Save" writes back to the original file (Chromium File System Access API) — comments are stored as hidden `<!-- KFU-COMMENT { ... json ... } -->` blocks placed immediately after the anchored element
- "Copy AI export" puts a plaintext rendering of the comments on the clipboard so you can paste straight into a Claude / Codex prompt
- ⌘S / Ctrl-S to save; warns on unsaved-close

### How AI agents see your comments

Two paths:

1. **Direct file read.** When the saved file is on disk, any AI with file access reads the markdown and sees the `<!-- KFU-COMMENT … -->` blocks inline. Each block is a single-line JSON object with `id / author / ts / parent / body`. Trivial to parse with a regex.
2. **Clipboard export.** Click "Copy AI export" → paste into your prompt. The dump is anchor-grouped, parent → replies, plain text.

### How AI agents reply

Two paths:

1. **You paste.** Ask the AI a question, copy its response, click the comment's "Reply" button, **change Author to `ai`**, paste, post, save.
2. **Agent edits the file directly.** Have the AI append `<!-- KFU-COMMENT {...,"author":"ai",...} -->` blocks immediately after the relevant block in the source file. Re-open the file in the viewer to see the reply rendered.

### Storage format

Comments are stored inline at the anchor position:

```markdown
This is a paragraph that I commented on.

<!-- KFU-COMMENT {"v":1,"id":"cmt-abc123","author":"vikrant","ts":"2026-05-02T10:00:00Z","parent":null,"body":"thinking about this"} -->
<!-- KFU-COMMENT {"v":1,"id":"cmt-def456","author":"ai","ts":"2026-05-02T10:05:00Z","parent":"cmt-abc123","body":"good catch — see [docs](https://example.com)"} -->

The next paragraph (no comments).
```

Hidden in normal markdown renders. AI-readable. Survives copy / paste / git operations. No sidecar files, no backend service, no auth.

### Anchoring

Comments anchor by **source position** — they live inline immediately after the block they're commenting on. Edits to other blocks don't orphan; edits to the SAME block keep the comment attached. If a block is deleted entirely, the comment becomes a free-floating block of HTML comments at that position (still readable).

### Browser compatibility

| Feature | Chromium (Chrome/Edge/Brave) | Safari / Firefox |
|---|---|---|
| Open file | ✓ via `showOpenFilePicker` (writable handle) | ✓ via `<input type=file>` (read-only) |
| Save in place | ✓ writes back to original file | Falls back to download |
| Comment / reply / resolve | ✓ | ✓ (state lives in memory until you download) |

For full round-trip workflow use a Chromium-based browser.

### Privacy

Local-only. No network calls except the four CDN script tags (DOMPurify, marked, highlight.js, mermaid). Your file content + comments never leave the browser.
