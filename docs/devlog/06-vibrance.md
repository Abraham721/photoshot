# 06 · Vibrance: HSL gain versus sampled RGB response

[Full development article and photo examples →](https://slohero.com/photoshot/guides/vibrance/en/)

The public implementation has multiple processing paths. Distinguish the saturation-weighted HSL calculation from the revision-2 protected path that interpolates measured RGB responses.

## Implementation

### HSL path

The (1−s) term reduces additional vibrance gain for already saturated colors. Saturation supplies a separate multiplier afterward; this expression does not describe every current vibrance mode.

### Sampled response path

The source identifies a 17×17×17 RGB grid measured with Photoshop 27.3.1. Vibrance response is applied first, then saturation response; slider positions between stored samples interpolate neighboring tables.

### Protection semantics

The function receives RGB and slider values, not face/object labels. Equal RGB values receive equal processing, whether they belong to skin or a wall. Local targeting still requires a mask.

## Core code

[src/adjustments.ts · v0.16.0 · L646–647](https://github.com/Abraham721/photoshot/blob/v0.16.0/src/adjustments.ts#L646-L647)

```ts
const gain = (v.vibrance / 100) * (1 - s) * (1 - 0.75 * skin);
fromHsl(h, clamp(s * (1 + gain) * (1 + v.saturation / 100)), l, out);
```

[src/adjustments.ts · v0.16.0 · L634–642](https://github.com/Abraham721/photoshot/blob/v0.16.0/src/adjustments.ts#L634-L642)

```ts
} else if (a.revision === 2 && v.skin) {
  scratch[0] = r;
  scratch[1] = g;
  scratch[2] = b;
  applyVibranceResponse(scratch, v.vibrance, 0, out);
  scratch[0] = out[0];
  scratch[1] = out[1];
  scratch[2] = out[2];
  applyVibranceResponse(scratch, v.saturation, 1, out);
```

[src/vibrance-response.ts · v0.16.0 · L60–67](https://github.com/Abraham721/photoshot/blob/v0.16.0/src/vibrance-response.ts#L60-L67)

```ts
const p = Math.max(0, Math.min(20, (value + 100) / 10)),
  lo = Math.floor(p),
  hi = Math.min(20, lo + 1),
  f = p - lo,
  a = data[mode * 21 + lo],
  b = data[mode * 21 + hi];
table = new Float32Array(a.length);
for (let i = 0; i < a.length; i++) table[i] = a[i] + (b[i] - a[i]) * f;
```

## Validation and limits

tests/vibrance.mjs covers neutral settings in both modes, alpha, extremes and invalid input. The saved portrait uses vibrance +30, saturation −5, protection on. A sampled/interpolated response is not a guarantee of identical output for every photo.

## Provenance

glfx.js vibrance.js was read as an external reference. Photoshot’s HSL calculation and measured-response integration are separate implementations; the measurement tables are not Adobe preset files or glfx.js tables.

- [glfx.js / vibrance.js — external implementation reference](https://github.com/evanw/glfx.js/blob/master/src/filters/adjust/vibrance.js)

[Full website article](https://slohero.com/photoshot/guides/vibrance/en/) · [All engineering notes](README.md)
