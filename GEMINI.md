# GEMINI.md

This file serves as a comprehensive reference guide, containing architectural blueprints, development standards, and instructional context for the Wardrobe project.

---

## 1. Project Overview

**Wardrobe** is a local-first, AI-assisted digital wardrobe gallery. It enables users to digitize their clothes by extracting transparent garment cutouts from everyday photos using OpenAI's Response/Vision/Image APIs, refining them through local chroma-key background removal, and generating realistic modeled editorial previews.

### Core Architectural Features
- **Local-First Database:** All user metadata, jobs, and imported garments are stored locally inside the ignored `data/` directory. The main database is a JSON file at `data/library.json`.
- **Integrated Vite Server:** The development server extends both static asset delivery and Node APIs by integrating custom Vite server middlewares as plugins (see `vite.config.mjs`).
- **Responsive Image Pipeline:** High-performance, on-demand image optimization and resizing are handled on the server side via standard `_ipx` routes powered by `ipx` and `sharp`. On the frontend, image optimization is wrapped in the `<OptimizedImage>` component.
- **AI-Powered Skills:** The workspace bundles custom Codex agent skills inside the `.agents/skills/` directory for automated batch operations (clothing import and outfit styling).

---

## 2. Architecture & File Directory

### Repository Structure
```text
wardrobe/
├── .agents/                    # Codex skill assets
│   └── skills/
│       ├── import-clothes/     # Agent instructions for batch importing clothes
│       └── generate-outfits/   # Agent instructions for styling and generating looks
├── docs/                       # Project screenshots
├── public/                     # Static client files (icons, service worker, web manifest)
├── scripts/                    # Server-side plugins & API handlers loaded by Vite
│   ├── import-job-api.mjs      # /api/import/... routes (job management, database & file ops)
│   └── responsive-image-api.mjs # /_ipx/... image resizing and optimization server (IPX)
├── src/                        # React Frontend
│   ├── App.jsx                 # Gallery UI, local edits state, and palette extractors
│   ├── import-flow.jsx         # Multi-stage interactive upload & correction wizard
│   ├── import-flow.css         # Styling for import workflow wizard
│   ├── OptimizedImage.jsx      # @unpic/react component wrapping image rendering through IPX
│   └── styles.css              # Main gallery layout and design token rules
├── index.html                  # Client entry point
├── vite.config.mjs             # Vite configuration injecting Custom APIs & Dev settings
└── package.json                # Project configuration, scripts, and dependencies
```

### Key API Routes (Vite Dev Server)
- `/api/import/jobs`: Manage temporary import jobs.
- `/api/import/assets`: Server-side temp file storage during the active import stage.
- `/api/import/library`: Read and serve imported files residing in `data/imported/`.
- `/api/import/config`: Read status configuration (API keys configured, presence of model reference).
- `/_ipx`: Server-side on-the-fly image-processing pipeline using IPX.

### Database Schema (`data/library.json`)
The local wardrobe database stores an array of imported garment records structured as follows:
```json
[
  {
    "id": "import-b97c234a-93f5-46f9-bf53-7313837b2d5d",
    "name": "Navy Fair Isle Cardigan",
    "part": "wholebody_up",
    "color": "#172033",
    "secondaryColor": "#f2efe6",
    "palette": ["#172033", "#f2efe6"],
    "tags": ["knit", "fair isle", "zip"],
    "image": "/api/import/library/import-b97c234a-93f5-46f9-bf53-7313837b2d5d-garment.png",
    "thumbnail": "/api/import/library/import-b97c234a-93f5-46f9-bf53-7313837b2d5d-garment.png",
    "modeledImage": "/api/import/library/import-b97c234a-93f5-46f9-bf53-7313837b2d5d-modeled.png",
    "importJobId": "b97c234a-93f5-46f9-bf53-7313837b2d5d"
  }
]
```
*Note: Acceptable `part` categories are strictly `upperbody`, `wholebody_up`, `lowerbody`, `accessories_up`, and `shoes`.*

---

## 3. Building and Running

Ensure Node.js 22 or higher is installed.

### Commands

| Command | Purpose |
|---|---|
| `npm install` | Install dependencies. |
| `cp .env.example .env` | Initialize local environment configuration. |
| `npm run dev` | Start Vite dev server with integrated API middleware on `localhost:5173`. |
| `npm run build` | Compile client-side assets for production. |
| `npm run preview` | Serve and preview local production builds on port 4173. |
| `npm run check` | Runs build validation (equivalent to `npm run build` in CI). |

### Prerequisites for Importer Feature
To enable the AI importer in the Web UI:
1. Provide a valid `OPENAI_API_KEY` in `.env`.
2. Add a PNG reference photo of yourself to `data/model-reference.png` (or the custom path specified in `WARDROBE_MODEL_REFERENCE`).

---

## 4. Development Conventions

- **Module Format:** The codebase utilizes native ECMAScript Modules (`"type": "module"` in `package.json`). All file extensions must be explicitly imported (e.g., `import { App } from "./App.jsx"`).
- **Styling:** Maintain visual polish using Vanilla CSS. Avoid installing or using TailwindCSS. Leverage the theme variables and layout designs configured in `src/styles.css` and `src/import-flow.css`.
- **Image Optimization:** Always render items using the `<OptimizedImage>` component from `src/OptimizedImage.jsx` rather than standard `<img>` tags, allowing the IPX resizing pipeline to deliver properly sized responsive assets.
- **Validation Guidelines:** The project has no formal test framework configured. Code correctness is verified by running a successful compiler build with `npm run check`. Always run `npm run check` before submitting pull requests or preparing commits.
- **Git Safety Protocol:** Under **no circumstances** should local user files inside `data/` (such as `library.json`, custom modeled images, or user-supplied reference photos) or private keys/secrets in `.env` be committed or staged to Git. These are ignored via `.gitignore` and must stay private and local.

---

## 5. Built-in Agent Skills

For automated curation, ingest, or styling, future agent sessions can activate these specialized skills:
1. **`import-clothes`**: Handles batch import of outfits or model photos, clean transparent cutout extraction, modeled editorial preview generation, and direct JSON database persistence in `data/library.json`.
2. **`generate-outfits`**: Curates outfits from `data/library.json`, styles them based on seasonal/color guidelines, generates 1:1 square modeled looks, and persists them under `data/outfits.json` and `data/outfit-images/`.
