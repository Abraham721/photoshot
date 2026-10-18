# Building a Free Photoshop Alternative, Part 1: Existing Code, Connected with AI

[All ten parts](README.md) · [Repository](../../README.md)

English edition · October 18, 2026 · Source excerpts pinned to their original revisions.

Image rendering and layer technology existed before AI. Building a free Photoshop alternative meant connecting those capabilities into one document and editing workflow. In Photoshot, React and the early PixiJS renderer supplied foundations; AI helped write and revise the code between them.

![Actual Photoshot editing workspace](images/01-workspace.webp)

*Actual Photoshot workspace. The example portrait is AI-generated; the editing interface and output are from Photoshot.*

## 1. The foundations already existed

Photoshop alternatives and public implementations predate AI. The change is less about making the idea possible for the first time and more about reducing the work of reading existing technology and adapting it to a product. Photoshot makes that distinction visible where libraries meet application code.

React provided UI and state management; PixiJS rendered image layers in the early version. The initial public commit imports Application, Sprite and Texture from pixi.js. This is evidence of an actual dependency, not just a repository mentioned during research.

Using GitHub-developed code here meant installing a package and calling its API. It did not mean copying an entire editor repository and relabeling the finished application.

[src/stage.tsx · eead3caec5407c14cdd8cdb698a12c633295f8ef · lines 1–5](https://github.com/Abraham721/photoshot/blob/eead3caec5407c14cdd8cdb698a12c633295f8ef/src/stage.tsx#L1-L5)

```tsx
'use client';
import { useEffect, useRef } from 'react';
import { Application, Sprite, Texture } from 'pixi.js';
import 'pixi.js/advanced-blend-modes';
import { layerSurface, type StudioDoc } from './document';
```

## 2. AI helped write the integration

PixiJS does not know where Photoshot stores position, rotation or opacity. The integration code creates a Sprite from a layer surface, then maps Photoshot’s layer fields onto PixiJS properties. In this excerpt, l is our layer data and s is the renderer’s object.

This was the practical role of AI-assisted coding: connect an established library to our application’s data and editing behavior.

This is a concrete role for AI: writing the code that gives an existing rendering library the rules of our document. Capable components do not automatically make a coherent editor. Their data and APIs still need to be connected.

[src/stage.tsx · eead3caec5407c14cdd8cdb698a12c633295f8ef · lines 27–33](https://github.com/Abraham721/photoshot/blob/eead3caec5407c14cdd8cdb698a12c633295f8ef/src/stage.tsx#L27-L33)

```tsx
const s = new Sprite(Texture.from(canvas));
s.anchor.set(0.5);
s.position.set(l.x + l.width / 2, l.y + l.height / 2);
s.rotation = (l.rotation * Math.PI) / 180;
s.alpha = l.opacity;
s.blendMode = l.blend;
a.stage.addChild(s);
```

## 3. A mask without erasing the source

Displaying layers was only the start. Painting a mask must preserve source pixels so disabling it can restore the image. Photopea’s mask documentation informed the expected behavior; Photoshot’s masking integration was written separately.

The current code applies destination-in to a temporary layer surface, then restores source-over. Keeping the source image and mask separate makes this reversible behavior possible.

[src/document.ts · v0.16.0 · lines 192–196](https://github.com/Abraham721/photoshot/blob/v0.16.0/src/document.ts#L192-L196)

```ts
if (layer.mask?.enabled) {
  ctx.globalCompositeOperation = 'destination-in';
  ctx.drawImage(maskSurface(layer.mask), 0, 0, c.width, c.height);
  ctx.globalCompositeOperation = 'source-over';
}
```

## 4. Separating the early and current renderers

Later development needed preview and export to agree. In v0.16.0, Stage calls the shared composite function rather than creating the early PixiJS Sprites. Preview size is bounded, and stale work can be cancelled when a newer edit arrives.

The accurate story therefore has two stages: actual PixiJS use initially, followed by a shared Canvas-based composition path. Reading different revisions matters when explaining where the software came from.

[src/stage.tsx · v0.16.0 · lines 31–35](https://github.com/Abraham721/photoshot/blob/v0.16.0/src/stage.tsx#L31-L35)

```tsx
const image = await composite(d, false, {
  maxEdge: 1600,
  cancelled: () => !mounted.current || r !== revision.current,
});
if (!mounted.current) break;
```

## 5. What the screenshots need to prove

The screenshots document the actual editor and implementation. Verification covered layer order, toggling masks, saving and reopening, and export. Mask-render tests also check that the source remains unchanged.

Fast code generation alone does not ensure that edits survive. Separating the library’s responsibilities from our document, persistence and composition code made failures easier to locate.

## 6. What we reused and what we built

We reused React and, initially, PixiJS. We consulted mask behavior documentation. With AI, we wrote and revised layer mapping, mask application, persistence and the later shared compositor. Building those connections turned available components into an editor.

The explanation is not that AI invented every editing feature. Established libraries supplied foundations, and integration code written with AI brought them into one editor. This record demonstrates the assembly of usable editing capabilities, rather than a complete reproduction of Photoshop.

Part 2 follows a more specific GitHub reference: Konva’s contrast filter, our own brightness/contrast implementation and the corrections required after comparison. Excerpts here come from the initial public commit and v0.16.0.

## Image comparison

![Before: AI-generated example portrait](images/01-original.webp)

*Before: AI-generated example portrait.*

![After: actual Photoshot output](images/01-result.webp)

*After: actual Photoshot output. Settings and verification limits are described in the article above.*

## Implementation and references

- [React](https://github.com/facebook/react)
- [PixiJS](https://github.com/pixijs/pixijs)
- [Photopea · Masks](https://www.photopea.com/learn/masks)
- [Photoshot · Initial renderer](https://github.com/Abraham721/photoshot/blob/eead3caec5407c14cdd8cdb698a12c633295f8ef/src/stage.tsx#L1-L5)
- [Photoshot · mask-render.mjs](https://github.com/Abraham721/photoshot/blob/v0.16.0/tests/mask-render.mjs)

---

[Part 2 →](02-brightness-contrast.md)
