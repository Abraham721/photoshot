# 10 · Photo filter: compiled color matrix versus layer compositing

[Full development article and photo examples →](https://slohero.com/photoshot/guides/photo-filter/en/)

Keep filter density inside the color transform and layer opacity inside compositing. The revision-2 matrix path differs from the legacy direct blend with the filter color.

## Implementation

### Compile the transform

Combine filter-color gains with RGB↔XYZ matrices to prepare photoMatrix once per adjustment configuration. Per-pixel processing linearizes RGB, evaluates row dot products, clamps and re-encodes to sRGB.

### Preservation and compatibility

The revised preservation branch applies color blending after filtering. Older projects keep the original source×(1−amount)+filterColor×amount path and its preservation behavior.

### Composition boundary

adjustPixels receives the adjustment, layer opacity, mask and blend separately. Density 22% with opacity 50% cannot generally be replaced by density 11%: nonlinear conversion, preservation and composition intervene.

## Core code

[src/adjustments.ts · v0.16.0 · L581–586](https://github.com/Abraham721/photoshot/blob/v0.16.0/src/adjustments.ts#L581-L586)

```ts
const photoMatrix =
  t === 'photo'
    ? inv.map((row) =>
        [0, 1, 2].map((c) => row.reduce((sum, x, i) => sum + x * gain[i] * xyz[i][c], 0)),
      )
    : [];
```

[src/adjustments.ts · v0.16.0 · L707–718](https://github.com/Abraham721/photoshot/blob/v0.16.0/src/adjustments.ts#L707-L718)

```ts
if (t === 'photo' && a.revision === 2) {
  scratch[0] = linear(r);
  scratch[1] = linear(g);
  scratch[2] = linear(b);
  for (let c = 0; c < 3; c++)
    out[c] = srgb(clamp(photoMatrix[c].reduce((sum, x, i) => sum + x * scratch[i], 0)));
  if (v.preserve) {
    scratch[0] = r;
    scratch[1] = g;
    scratch[2] = b;
    blendRGB(scratch, out, 'color', out);
  }
```

[src/document.ts · v0.16.0 · L284–293](https://github.com/Abraham721/photoshot/blob/v0.16.0/src/document.ts#L284-L293)

```ts
adjustPixels(
  pixels.data,
  layer.adjustment,
  layer.opacity,
  mask,
  layer.blend,
  start,
  Math.min(start + chunk, pixels.data.length),
  compiled,
);
```

## Validation and limits

The saved demonstration uses Cooling (80), color #006dff, density 22%, preservation on. Review the transform and compositing stages independently. The example does not establish exact luminance conservation or full Adobe equivalence.

## Provenance

Adobe documentation informed the controls. No import from a particular external GitHub filter was established. Photoshot reuses its own document/adjustment infrastructure and separately written matrix integration.

- [Adobe / Photo Filter controls — behavior reference](https://helpx.adobe.com/photoshop/using/applying-color-balance-adjustment.html)

[Full website article](https://slohero.com/photoshot/guides/photo-filter/en/) · [All engineering notes](README.md)
