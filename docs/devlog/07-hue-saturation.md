# 07 · Hue/saturation: circular distance, feathering, and revisioned response

[Full development article and photo examples →](https://slohero.com/photoshot/guides/hue-saturation/en/)

Reuse RGB/HSL conversion, but store color-band selection explicitly. Range selection and per-pixel response have separate rules and must survive saving the adjustment.

## Implementation

### Circular distance

Wrap hue difference into −180°…180° before taking its magnitude: 359° and 1° are 2° apart. Convert distance into a shared adjustment weight rather than a binary selection.

### Width and feather

For center 240°, width 30°, feather 30°: full strength extends 15° either side, weight is 0.5 at 30°, and zero at 45°. The picker rejects nearly gray pixels and changes the center of the nearest band.

### Revisioned response

In revision 2, positive saturation +50 uses 1/(1−0.5)=2 before clamping rather than 1.5. Revised lightness adjusts RGB before returning to HSL; legacy adjustments retain their older calculation.

## Core code

[src/adjustments.ts · v0.16.0 · L603–610](https://github.com/Abraham721/photoshot/blob/v0.16.0/src/adjustments.ts#L603-L610)

```ts
if (!v.colorize)
  for (const band of a.bands ?? []) {
    const d = Math.abs(((h * 360 - band.center + 540) % 360) - 180),
      w = clamp(1 - (d - band.width / 2) / band.feather);
    hue += band.hue * w;
    sat += band.saturation * w;
    light += band.lightness * w;
  }
```

[src/tone-tools.ts · v0.16.0 · L45–48](https://github.com/Abraham721/photoshot/blob/v0.16.0/src/tone-tools.ts#L45-L48)

```ts
const bands = structuredClone(a.bands ?? makeAdjustment('hsl').bands!);
const i = Math.round(hue / 60) % 6;
bands[i].center = hue;
return { ...a, bands };
```

[src/adjustments.ts · v0.16.0 · L619–626](https://github.com/Abraham721/photoshot/blob/v0.16.0/src/adjustments.ts#L619-L626)

```ts
const ns = v.colorize
    ? v.colorizeSaturation / 100
    : clamp(
        inputS *
          (a.revision === 2 && sat > 0
            ? 1 / Math.max(0.0001, 1 - sat / 100)
            : 1 + sat / 100),
      ),
```

## Validation and limits

Inspect wraparound at 0°/360°, the three feather boundaries and gray-pixel rejection. These are review cases implied by the code, not newly reported test runs. The saved blue-band example uses hue −25 and saturation +15, with global adjustments zero.

## Provenance

The session records consulting Konva HSL.ts, but not importing its range-selection code. The excerpts are Photoshot’s own implementation. A selected color range can affect multiple objects; use a mask for spatial selection.

[Full website article](https://slohero.com/photoshot/guides/hue-saturation/en/) · [All engineering notes](README.md)
