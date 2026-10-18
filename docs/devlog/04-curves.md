# 04 · Curves: revisioned interpolation and channel-table order

[Full development article and photo examples →](https://slohero.com/photoshot/guides/curves/en/)

Precompute curves into channel lookup tables. Keep the interpolation revision synchronized between the editor preview and pixel processing so saved control points keep their intended meaning.

## Implementation

### Interpolation choice

The initial curveTable used shape-preserving Hermite interpolation, including zero tangents at slope sign changes. The revised naturalCurveTable solves a natural cubic spline; the same control points can produce different intermediate values.

### Compatibility contract

Revision 2 selects the natural spline. Older adjustments retain Hermite, and the UI selects its displayed table with the same revision condition.

### Application order

Pixel processing reads the individual channel table before the shared RGB table. Fractional positions interpolate adjacent entries; the table alone does not describe this ordering.

## Core code

[src/adjustments.ts · v0.16.0 · L414–424](https://github.com/Abraham721/photoshot/blob/v0.16.0/src/adjustments.ts#L414-L424)

```ts
for (let i = 1; i < n - 1; i++) {
  const span = points[i + 1].x - points[i - 1].x,
    sig = (points[i].x - points[i - 1].x) / span,
    p = sig * y2[i - 1] + 2;
  y2[i] = (sig - 1) / p;
  const slope =
    (points[i + 1].y - points[i].y) / (points[i + 1].x - points[i].x) -
    (points[i].y - points[i - 1].y) / (points[i].x - points[i - 1].x);
  u[i] = ((6 * slope) / span - sig * u[i - 1]) / p;
}
for (let i = n - 2; i >= 0; i--) y2[i] = y2[i] * y2[i + 1] + u[i];
```

[src/curves-editor.tsx · v0.16.0 · L43–43](https://github.com/Abraham721/photoshot/blob/v0.16.0/src/curves-editor.tsx#L43-L43)

```ts
const table = (value.revision === 2 ? naturalCurveTable : curveTable)(points),
```

[src/adjustments.ts · v0.16.0 · L529–535](https://github.com/Abraham721/photoshot/blob/v0.16.0/src/adjustments.ts#L529-L535)

```ts
if (t === 'curves') {
  x = ct![channel as Channel][i];
  const f = clamp(x) * 255,
    lo = Math.floor(f),
    hi = Math.min(255, lo + 1);
  x = ct!.rgb[lo] + (ct!.rgb[hi] - ct!.rgb[lo]) * (f - lo);
}
```

## Validation and limits

For the historical chart with RGB points (0,0), (64,48), (192,208), (255,255), the revised path recorded zero mean and maximum difference. The portrait instead uses (64,80) and (192,176). Agreement on that chart does not establish agreement for all curves.

## Provenance

glfx.js curves.js supplied a concrete LUT design to inspect. Photoshot’s interpolation, revision handling and editor integration were written separately; the whole WebGL filter was not imported.

- [glfx.js · curves.js (reference)](https://github.com/evanw/glfx.js/blob/master/src/filters/adjust/curves.js)

[Full website article](https://slohero.com/photoshot/guides/curves/en/) · [All engineering notes](README.md)
