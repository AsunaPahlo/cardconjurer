# Card Conjurer - CLAUDE.md

## Project Overview

Card Conjurer is a custom Magic: The Gathering card generator built as a vanilla JavaScript web application. Originally a popular online tool, it was taken offline after a Wizards of the Coast cease-and-desist in November 2022. This repo preserves the application for local use.

Users can create custom MTG cards with extensive customization: multiple frame styles/templates, custom artwork, set symbols, watermarks, collector info, advanced text layout, and support for various card types (tokens, sagas, planeswalkers, DFCs, etc.).

## Tech Stack

- **Vanilla JavaScript** (no framework - no React/Vue/Angular)
- **HTML5 Canvas API** for all card rendering
- **HTMX v1.8.4** for SPA-like page navigation (content swapping via `hx-get`/`hx-target`)
- **CSS3** with CSS custom properties for theming
- **No build system** - static files served directly
- **No package manager** - no npm/node_modules
- **No tests** - no test framework or test files exist

## Running Locally

- `launcher.py` starts a Python 3 HTTP server on port 8080
- `make start` builds and runs via Docker (Nginx on port 4242)
- Or use any static file server (WAMP, XAMPP, etc.)

## Directory Structure

```
cardconjurer/
├── js/                     # All JavaScript
│   ├── creator-23.js       # Main card creator engine (~5,500 lines) - THE core file
│   ├── main-1.js           # Global utilities (menu toggle, notifications, drag/drop)
│   ├── themes.js           # Theme system with localStorage persistence
│   ├── themeEditor.js      # Theme editor UI
│   ├── frameSearch.js      # Frame search index (170+ name mappings)
│   ├── htmx.min.js         # HTMX library (DO NOT MODIFY)
│   ├── qrious.min.js       # QR code library (DO NOT MODIFY)
│   └── frames/             # ~296 frame configuration files
│       ├── group*.js       # Frame group definitions (categories)
│       ├── pack*.js        # Frame pack configs (specific frame sets)
│       └── manaSymbols*.js # Mana symbol style sets
├── creator/index.html      # Card creator UI (the main interface, ~55KB)
├── css/
│   ├── style-9.css         # Main stylesheet with CSS variables
│   └── reset.css           # CSS reset
├── fonts/                  # 40+ custom fonts (Beleren, Gotham, Plantin, Matrix, etc.)
├── img/                    # UI images, frame masks, mana symbols, watermarks, set symbols
├── data/images/            # Card frame template images (~196MB, 687 files)
├── print/                  # Printing tool (print.js + index.html)
├── converter/              # Card image converter
├── askurza/                # Planeswalker ability generator
├── phyrexian/              # Phyrexian text generator
├── theme/                  # Theme editor page
├── gallery/                # Card gallery viewer
├── globalHTML/             # Shared header/footer HTML fragments
├── local_art/              # User-provided art directory
└── index.html              # Main landing page / SPA shell
```

## Architecture

### SPA Navigation
The app uses HTMX to swap content into `<div id="content">` without full page reloads. Each module (creator, print, askurza, etc.) has its own `index.html` loaded via `hx-get`.

### Canvas Rendering Pipeline
Card rendering uses multiple compositing canvases:
- `cardCanvas` - final composite output
- `frameCanvas` - frame graphics
- `frameMaskingCanvas` / `frameCompositingCanvas` - layer blending
- `textCanvas` / `paragraphCanvas` / `lineCanvas` - text layout
- `watermarkCanvas` - watermark compositing
- `bottomInfoCanvas` - collector info
- `guidelinesCanvas` - optional guides
- `prePTCanvas` - power/toughness pre-rendering

### Card Object (Global State)
```javascript
var card = {
  width: 2010, height: 2814,
  marginX: 0, marginY: 0,
  frames: [],              // Active frame layers
  artSource: '',           // Image URL / data URL
  artX: 0, artY: 0, artZoom: 1, artRotate: 0,
  setSymbolSource: '', setSymbolX/Y/Zoom: ...,
  watermarkSource: '', watermarkX/Y/Zoom/Left/Right/Opacity: ...,
  infoYear: <currentYear>,
  bottomInfoColor: 'white',
  manaSymbols: [],
  version: '',
  // ...many more properties added dynamically by frame configs
};
```

### Key Global Variables
- `availableFrames[]` - loaded frame templates
- `selectedFrame` / `selectedFrameIndex` - current frame selection
- `selectedMaskIndex` - active mask
- `canvasList[]` - all rendering canvas names
- `replacementMasks{}` - custom mask mappings
- `art`, `setSymbol`, `watermark` - Image objects for current card assets

### Frame System
Frames are organized hierarchically:
1. **Groups** (`groupStandard-3.js`, etc.) define categories shown in the UI
2. **Packs** (`packM15.js`, etc.) define specific frame sets within groups
3. Each pack provides frame options with image URLs, masks, and text configurations

Frame configs are loaded dynamically via `<script>` injection.

## Key Files in Detail

### `js/creator-23.js` (~5,500 lines)
The heart of the application. Key functions:
- `resetCardIrregularities()` - reset card dimensions/state
- `drawFrames()` - main rendering loop
- `loadFramePacks()` / `loadFrameVersion()` - frame template loading
- `frameOptionClicked()` - frame selection handler
- `cardFrameProperties()` - determine color/type from frame
- `exportCard()` - card export/save
- `loadManaSymbols()` - mana symbol rendering
- `scaleX()` / `scaleY()` / `scaleWidth()` / `scaleHeight()` - coordinate scaling
- `setBottomInfoStyle()` - collector information layout
- `artEdited()` / `setSymbolEdited()` / `watermarkEdited()` - image update handlers

### `js/main-1.js`
- `toggleMenu()` - hamburger menu
- `notify(message, seconds)` - toast notifications
- `uploadFiles()` - file upload / drag-drop handling
- `closeNotification()` - dismiss notifications

### `js/themes.js`
- `updateCSS()` - apply CSS variable theme
- `rainbowMode()` - animated color cycling
- Theme persisted in `localStorage['theme']`

### `creator/index.html`
The massive card creator UI with all form controls (tabs for frames, art, text, set symbols, watermarks, collector info, etc.). Uses inline `onclick` handlers extensively.

## Coding Conventions

### JavaScript
- **camelCase** for all functions and variables: `toggleMenu()`, `selectedFrameIndex`
- **Global scope** - most variables and functions are global (no modules/imports)
- **var** used throughout (not const/let for most variables)
- **Inline event handlers** - `onclick='functionName()'` in HTML
- **Image objects** created with `new Image()` and managed globally
- **Canvas operations** use direct 2D context API calls
- **Async functions** used for rendering pipeline (`async function resetCardIrregularities()`)
- **No semicolons** after function declarations (but used in statements)
- **Tab indentation** in most files

### CSS
- **kebab-case** for classes: `readable-background`, `creator-grid`, `frame-option`
- **CSS variables** for theming: `--site-background`, `--color-primary`
- Defined in `style-9.css`

### File Naming
- **Versioned filenames**: `creator-23.js`, `style-9.css`, `groupStandard-3.js`
- **Prefixed by type**: `group*.js`, `pack*.js`, `manaSymbols*.js`

### HTML
- **Functional IDs**: `previewCanvas`, `text-editor`, `selectFrameGroup`
- **HTMX attributes** for navigation: `hx-get`, `hx-target`, `hx-trigger`

## Important Notes

- `fixUri()` function wraps all image paths - originally redirected to CDN, now returns input unchanged (disabled for local version)
- Card dimensions default to 2010x2814 pixels (high-res standard)
- `baseWidth=1500`, `baseHeight=2100` are the original dimensions; `highResScale=1.34`
- LocalStorage keys: `theme` (JSON theme object), `high-res` (boolean string)
- The `?debug` URL parameter enables debug mode with alerts and visible debug elements
- Cross-origin is set on all Image objects (`image.crossOrigin = 'anonymous'`)
- No minification or bundling - all JS served as-is
- The `data/images/` directory is very large (~196MB) with card frame templates
- Frame config files in `js/frames/` are loaded on-demand, not all at once
