# 01 · Layer state, reversible masks, and a shared compositor

[Full development article and photo examples →](https://slohero.com/photoshot/guides/basic-tools/en/)

Keep document state independent of the renderer. React owns UI/state; the initial public revision used PixiJS Sprites, while v0.16.0 uses a shared Canvas compositor for preview and output.

## Implementation

### Integration boundary

The initial adapter maps position, rotation and opacity from a Photoshot layer onto a Sprite. PixiJS was an actual dependency in that revision; it is not evidence that the current renderer still uses the same path.

### Reversible masking

Apply destination-in to a temporary layer surface, then restore source-over. Keep the source pixels separate from the mask so disabling the mask can restore the image.

### Preview/output consistency

Stage delegates to the shared composite function, bounds preview size and supports cancellation of stale work. A new rendering path must still preserve layer order, project persistence and export behavior.

## Core code

[src/stage.tsx · eead3caec5407c14cdd8cdb698a12c633295f8ef · L27–33](https://github.com/Abraham721/photoshot/blob/eead3caec5407c14cdd8cdb698a12c633295f8ef/src/stage.tsx#L27-L33)

```ts
const s = new Sprite(Texture.from(canvas));
s.anchor.set(0.5);
s.position.set(l.x + l.width / 2, l.y + l.height / 2);
s.rotation = (l.rotation * Math.PI) / 180;
s.alpha = l.opacity;
s.blendMode = l.blend;
a.stage.addChild(s);
```

[src/document.ts · v0.16.0 · L192–196](https://github.com/Abraham721/photoshot/blob/v0.16.0/src/document.ts#L192-L196)

```ts
if (layer.mask?.enabled) {
  ctx.globalCompositeOperation = 'destination-in';
  ctx.drawImage(maskSurface(layer.mask), 0, 0, c.width, c.height);
  ctx.globalCompositeOperation = 'source-over';
}
```

[src/stage.tsx · v0.16.0 · L31–35](https://github.com/Abraham721/photoshot/blob/v0.16.0/src/stage.tsx#L31-L35)

```ts
const image = await composite(d, false, {
  maxEdge: 1600,
  cancelled: () => !mounted.current || r !== revision.current,
});
if (!mounted.current) break;
```

## Validation and limits

Inspect tests/mask-render.mjs for source immutability and mask behavior. The full article records save/reopen and export checks. Treat those historical checks separately from the repository CI run.

## Provenance

React and early PixiJS are used dependencies. Photopea mask documentation is a behavioral reference. Photoshot layer adapters, persistence and later composition code are application-specific integration.

- [React](https://github.com/facebook/react)
- [PixiJS](https://github.com/pixijs/pixijs)
- [Photopea · Masks](https://www.photopea.com/learn/masks)

[Full website article](https://slohero.com/photoshot/guides/basic-tools/en/) · [All engineering notes](README.md)
