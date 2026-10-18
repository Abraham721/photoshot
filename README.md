# Photoshot

**A free browser photo editor, and the engineering record of connecting existing technology with AI.**

[Try Photoshot](https://slohero.com/photoshot/editor/) · [Read the English development series](docs/devlog/README.md) · [Release v0.16.1](https://github.com/Abraham721/photoshot/releases/tag/v0.16.1)

![Actual Photoshot workspace](docs/devlog/images/01-workspace.webp)

## What's new in v0.16.1

The English development series now includes Parts 1–10, with full code excerpts, pinned source links, actual workspace screenshots and before/after examples. This is a documentation and repository-metadata update to the public `v0.16.0` source snapshot. It does not claim new editing algorithms or parity with the current production service.

The original Photoshot implementation remains public source with **no open-source license granted**. See [COPYRIGHT.md](COPYRIGHT.md) and [third-party notices](THIRD_PARTY_NOTICES.md).

## English development series: Parts 1–10

| Part | Article |
| --- | --- |
| 01 | [Existing Code, Connected with AI](docs/devlog/01-basic-tools.md) |
| 02 | [Connecting Existing Contrast Code with AI](docs/devlog/02-brightness-contrast.md) |
| 03 | [Assembling Familiar Levels Math into a Tool](docs/devlog/03-levels-histogram.md) |
| 04 | [Connecting a GitHub Curves Idea with AI](docs/devlog/04-curves.md) |
| 05 | [Reusing the Foundation for Exposure](docs/devlog/05-exposure.md) |
| 06 | [From Existing Vibrance Code to Integration with AI](docs/devlog/06-vibrance.md) |
| 07 | [Connecting Color Selection to Existing Code with AI](docs/devlog/07-hue-saturation.md) |
| 08 | [Connecting Known Color Balance Behavior with AI](docs/devlog/08-color-balance.md) |
| 09 | [Extending Existing Color Code into Black & White](docs/devlog/09-black-white.md) |
| 10 | [Connecting Existing Technology into an Editor with AI](docs/devlog/10-photo-filter.md) |

These article numbers follow the current journal order. Historical source-release episode numbering differs; old commits and tags remain available. The prose is English. Verbatim code excerpts and the existing source editor retain their original UI strings.

## Run locally

Use Node.js 24 (minimum 22.13.0) and npm.

```sh
git clone https://github.com/Abraham721/photoshot.git
cd photoshot
npm ci
npm run dev
```

Open the local URL printed by Vite, normally `http://127.0.0.1:5173`. The standalone editor needs no backend or API key.

```sh
npm test
npm run build
npm run preview
```

The build is written to `dist/`. Use the development or preview server rather than opening `index.html` directly.

## What this source snapshot includes

- Image import, crop, text, brush/eraser, layer transforms and non-destructive masks.
- Project save/reopen, undo/redo and PNG/JPG export.
- Sixteen adjustment types, including brightness/contrast, levels, curves, exposure, vibrance, hue/saturation, color balance, black-and-white, photo filters and channel mixing.
- Layer opacity, clipping and 27 blend modes; CUBE/3DL/LOOK/CSP and limited ICC lookup support.
- Tests for masks, alpha, compositing, saved-project compatibility, adjustments, LUT/ICC parsing and transformations.

## Scope and limitations

This public snapshot is separate from the larger production application. It supports RGB 8-bit documents, a canvas dimension of up to 4096 pixels, up to 50 layers and image inputs up to 25 MB per file. PSD, RAW, CMYK, Camera Raw, AI layer separation and the production backend are outside this snapshot.

Edits are processed in the browser. Automatic recovery uses IndexedDB; keep exported `.layerstudio` project files because browser data can be cleared. The standalone snapshot uses its own `photoshot-adjustments` storage name. Fonts, browser behavior and blend modes can affect rendered results.

Photoshot is an independent project, not an Adobe product. A free editor is not a free distribution of Adobe Photoshop, and example comparisons do not establish full feature or pixel equivalence.

## Project layout

```text
src/                 Editor UI, document model and rendering/adjustment code
src/vendor/lcms/     Bundled ICC engine and its original license notices
tests/               Rendering, adjustment and compatibility checks
docs/devlog/         English development articles 1–10 and local images
docs/examples/       Small LUT examples
```

## Credits and rights

Created by [SLOHERO](https://slohero.com/). React, early PixiJS usage, later Canvas composition and third-party color components are described with their distinct roles in the articles. Referenced implementations are not presented as copied dependencies unless supported by the development record.

The demo portraits are AI-generated; editor screenshots and outputs are actual Photoshot results. Third-party libraries keep their own licenses. For use of original Photoshot code beyond rights granted by applicable law and GitHub's terms, contact the creator. Report reproducible issues without including private photos or credentials.
