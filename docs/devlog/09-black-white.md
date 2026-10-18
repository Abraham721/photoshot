# 09 · Black & white: six hue weights, neutral invariance, and tint

[Full development article and photo examples →](https://slohero.com/photoshot/guides/black-white/en/)

Convert original color to an editable gray value by interpolating six hue controls. This is color-dependent mixing, not facial recognition or a uniform desaturation operation.

## Implementation

### Hue interpolation

h×6 identifies adjacent red/yellow/green/cyan/blue/magenta controls. Its fractional part interpolates their weights; modulo arithmetic wraps magenta back to red.

### Neutral invariant

gray = min(RGB) + (max(RGB)−min(RGB))×weight. When all channels are equal, the second term is zero, so color-slider changes leave neutral gray unchanged with tint off.

### Separate tint and composition

out.fill(gray) creates equal RGB channels. Optional tint is applied afterward; layer masks and opacity are handled by the shared compositor. Similar source hues can respond together even on different objects.

## Core code

[src/adjustments.ts · v0.16.0 · L692–699](https://github.com/Abraham721/photoshot/blob/v0.16.0/src/adjustments.ts#L692-L699)

```ts
if (t === 'blackwhite') {
  const [h] = hsl(r, g, b, scratch),
    pos = h * 6,
    i = Math.floor(pos) % 6,
    f = pos - Math.floor(pos),
    weight = (v['color' + i] * (1 - f) + v['color' + ((i + 1) % 6)] * f) / 100;
  const gray = clamp(Math.min(r, g, b) + (Math.max(r, g, b) - Math.min(r, g, b)) * weight);
  out.fill(gray);
```

[src/adjustments.ts · v0.16.0 · L699–700](https://github.com/Abraham721/photoshot/blob/v0.16.0/src/adjustments.ts#L699-L700)

```ts
out.fill(gray);
if (v.tint) fromHsl(v.tintHue / 360, v.tintSaturation / 100, gray, out);
```

## Validation and limits

From the formula, pure red with Red 55 yields normalized gray 0.55 (about 140/255); pure yellow with Yellow 60 yields 153/255. These are single-color derivations, not whole-image test metrics. The portrait settings are 55/60/40/60/5/80, tint off.

## Provenance

No specific external black-and-white filter import was found in the record. The reusable foundation is Photoshot’s HSL conversion, stored adjustments and compositor; the six-control gray calculation is application code.

[Full website article](https://slohero.com/photoshot/guides/black-white/en/) · [All engineering notes](README.md)
