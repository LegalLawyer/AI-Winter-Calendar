# CLAUDE.md — AI-Winter-Calendar

## Project Overview

**AI-Winter-Calendar** is a single-page, interactive advent calendar web application. It presents 24 daily tips for using ChatGPT in a professional legal setting, targeted at Sidley law firm. Each calendar "door" opens to reveal a tip via a 3D flip animation and modal dialog.

## Repository Structure

```
AI-Winter-Calendar/
├── index.html      # Entire application — HTML, CSS, and JavaScript in one file
└── CLAUDE.md       # This file
```

There is no build system, no package manager, and no backend. The entire application is a single self-contained HTML file.

## Technology Stack

| Layer       | Technology                                          |
|-------------|-----------------------------------------------------|
| Markup      | HTML5                                               |
| Styling     | Tailwind CSS (CDN) + custom `<style>` block         |
| Logic       | Vanilla ES6 JavaScript (inline `<script>`)          |
| SDK         | Element SDK (`/_sdk/element_sdk.js`, `/_sdk/data_sdk.js`) for dynamic config |
| Security    | Cloudflare challenge script (last line of file)     |

## Key Sections in `index.html`

| Lines       | Purpose                                                                 |
|-------------|-------------------------------------------------------------------------|
| 1–8         | DOCTYPE, `<head>`, external script tags                                 |
| 8–253       | `<style>` block — all custom CSS (calendar grid, door 3D flip, modal, snowflakes) |
| 258–274     | `<body>` HTML structure — calendar grid container, modal, snowflake container |
| 275–320     | `defaultConfig` object — 25 tips (tip_1…tip_25) and door colors         |
| 324–354     | `createCalendar()` — builds 24 clickable doors in the DOM               |
| 356–396     | `openDoor(day)` / `showModal(day)` — interaction handlers               |
| 398–440     | `onConfigChange(config)` — applies SDK-driven theming to CSS variables  |
| 442–523     | Element SDK initialization and property mappings                        |
| 525–540     | `createSnowflakes()` — animated falling snowflake particles             |
| 542–544     | Initialization calls                                                    |
| 546         | Cloudflare security script                                              |

## Key Functions

- **`createCalendar()`** — Generates the 24-door grid; uses `defaultConfig.colors` for door coloring.
- **`openDoor(day)`** — Marks the door as opened (persists in a `Set`), triggers flip animation.
- **`showModal(day)`** — Renders the tip HTML inside the modal and shows it.
- **`closeModal()`** — Hides the modal.
- **`onConfigChange(config)`** — Consumes the Element SDK config object; applies colors, fonts, and tip content via CSS custom properties and direct DOM updates.
- **`createSnowflakes()`** — Injects animated snowflake elements for visual effect.

## Configuration

Configuration is managed in two ways:

1. **`defaultConfig` (hardcoded)** — Lives in the `<script>` block. Contains:
   - `title`, `subtitle`
   - `background_color`, `surface_color`, `text_color`, `primary_action_color`, `secondary_action_color`
   - `font_family`, `font_size`
   - `tip_1` … `tip_25` — HTML strings for each day's content

2. **Element SDK (runtime)** — `window.elementSdk?.config` overrides defaults at runtime, enabling external theming and content customization without code changes.

To add or change a tip, modify the corresponding `tip_N` property in `defaultConfig` (or via the SDK config).

## Development Workflow

### Running Locally

No build step required. Open `index.html` directly in a browser:

```bash
# Option 1: open directly
open index.html

# Option 2: serve with any static server
npx serve .
python3 -m http.server 8080
```

Note: The Element SDK scripts (`/_sdk/...`) resolve against the server origin, so running through a server is preferred over opening as a `file://` URL if SDK features are needed.

### Making Changes

1. Edit `index.html` directly — the single source of truth.
2. Reload the browser to see changes.
3. No compilation, transpilation, or bundling needed.

### Testing

There is no automated test suite. Manual browser testing is the only testing method. When making changes:
- Verify all 24 doors open and display the correct tip.
- Confirm modal open/close behavior.
- Check responsive layout at mobile (≤768px) and desktop widths.
- Test that snowflake animation runs without errors.

## Coding Conventions

- **No external dependencies added** — keep the project dependency-free beyond the CDN links already in place.
- **Vanilla JS only** — do not introduce frameworks (React, Vue, etc.).
- **Self-contained** — all logic, styles, and markup stay in `index.html`.
- **HTML content in tips** — tip strings may contain HTML (links, `<strong>`, etc.); keep them safe and avoid unsanitized user input.
- **CSS custom properties** — theming relies on CSS variables set in `onConfigChange()`; respect this pattern when adding new styled elements.
- **`openedDoors` Set** — tracks which doors have been opened in-memory (resets on page reload); do not introduce external persistence without understanding the SDK data layer.

## Deployment

Deploy by copying `index.html` to any static web server or CDN. No build artifacts are generated. Ensure the host can serve the Element SDK paths (`/_sdk/element_sdk.js`, `/_sdk/data_sdk.js`) if SDK functionality is required.

## Git Conventions

- **Default branch:** `main` (remote), `master` (local alias)
- **Feature branches:** Use the `claude/` prefix (e.g., `claude/add-claude-documentation-wIAJ5`)
- Commit messages should be short and descriptive (e.g., `"Update tip 12 with new link"`)
- GPG signing is configured — commits are signed automatically via SSH key

## What NOT to Do

- Do not add a build system or package manager unless explicitly requested.
- Do not split the application into multiple files without a clear reason.
- Do not introduce `eval()`, `innerHTML` with unsanitized external input, or other XSS vectors.
- Do not remove the Cloudflare challenge script at the bottom of the file.
- Do not modify the Element SDK integration without understanding how the SDK config merges with `defaultConfig`.
