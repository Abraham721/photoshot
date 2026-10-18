# 05 · Exposure: EV scaling, conversion space, and saved revisions

[Full development article and photo examples →](https://slohero.com/photoshot/guides/exposure/en/)

Exposure reuses compileAdjustment and the existing 256-entry table path. The key implementation decisions are the conversion domain, operation order and compatibility branch.

## Implementation

### EV is a scale

2 ** v.ev multiplies the working value: +1 doubles it and −1 halves it before offset/gamma processing. Adding an EV number directly to encoded display bytes would describe a different operation.

### Two conversion paths

Older adjustments retain the linear/srgb conversion helpers. Revision 2 uses the 2.2-power path introduced after comparison. Do not describe these distinct paths as a single unchanged formula.

### Persistence boundary

The saved revision chooses the calculation. Improving a formula must not silently reinterpret existing project files; masks, table application and layer composition stay shared.

## Core code

[src/adjustments.ts · v0.16.0 · L482–483](https://github.com/Abraham721/photoshot/blob/v0.16.0/src/adjustments.ts#L482-L483)

```ts
const linear = (s: number) => (s <= 0.04045 ? s / 12.92 : ((s + 0.055) / 1.055) ** 2.4);
const srgb = (s: number) => (s <= 0.0031308 ? 12.92 * s : 1.055 * s ** (1 / 2.4) - 0.055);
```

[src/adjustments.ts · v0.16.0 · L536–540](https://github.com/Abraham721/photoshot/blob/v0.16.0/src/adjustments.ts#L536-L540)

```ts
if (t === 'exposure')
  x =
    a.revision === 2
      ? clamp(x ** 2.2 * 2 ** v.ev + v.offset) ** (1 / (2.2 * v.gamma))
      : srgb(clamp(linear(x) * 2 ** v.ev + v.offset) ** (1 / v.gamma));
```

## Validation and limits

Recorded Photoshop chart comparisons: EV/offset/gamma +1/0/1 gave mean difference about 0.015, maximum 2; −0.7/0.015/1.2 gave about 0.015, maximum 1. The portrait uses +0.35/−0.004/1.05. These records do not cover every HDR or color-management workflow.

## Provenance

This feature reuses Photoshot’s adjustment infrastructure and standard color-conversion mathematics. The record does not establish copying these conversion lines from a particular external repository.

[Full website article](https://slohero.com/photoshot/guides/exposure/en/) · [All engineering notes](README.md)
