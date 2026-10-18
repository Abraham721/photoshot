# 02 · Brightness/contrast: ordered response sampling and LUT compilation

[Full development article and photo examples →](https://slohero.com/photoshot/guides/brightness-contrast/en/)

Compile a 256-entry channel lookup table once per adjustment configuration. The relevant change is the revisioned transfer function and operation order, not a new document pipeline.

## Implementation

### Legacy compatibility

The older path applies brightness before midpoint-based contrast. It remains available for saved adjustments that use the earlier calculation.

### Measured-response path

The revised normal mode samples brightness and then contrast responses. Swapping the order changes the result because these operations do not generally commute.

### Shared execution

compileAdjustment prepares tables used by the pixel path; masks, layer opacity and saved parameters remain responsibilities of the existing adjustment-layer machinery.

## Core code

[src/adjustments.ts · v0.16.0 · L506–510](https://github.com/Abraham721/photoshot/blob/v0.16.0/src/adjustments.ts#L506-L510)

```ts
if (t === 'brightness' && a.revision !== 2) {
  x = v.brightness < 0 ? x * (1 + v.brightness / 100) : x + ((1 - x) * v.brightness) / 100;
  const contrast = v.contrast < 0 ? 1 + v.contrast / 100 : 1 / (1 - v.contrast / 101);
  x = (x - 0.5) * contrast + 0.5;
}
```

[src/adjustments.ts · v0.16.0 · L517–518](https://github.com/Abraham721/photoshot/blob/v0.16.0/src/adjustments.ts#L517-L518)

```ts
x = sampleResponse(toneResponse.brightness, v.brightness, -150, x);
x = sampleResponse(toneResponse.contrast, v.contrast, -50, x);
```

[src/adjustments.ts · v0.16.0 · L503–505](https://github.com/Abraham721/photoshot/blob/v0.16.0/src/adjustments.ts#L503-L505)

```ts
const tables = ['r', 'g', 'b'].map((channel) =>
  Float64Array.from({ length: 256 }, (_, i) => {
    let x = i / 255;
```

## Validation and limits

The historical +30 brightness/+20 contrast chart improved from mean channel error 11.505 (maximum 25) for the tested intermediate implementation to 0.241 (maximum 1). Reversing operation order gave 1.150 and 4. This is one recorded chart, not a general Photoshop-equivalence claim; the portrait uses +18/+12.

## Provenance

Konva Contrast.ts was consulted as an external implementation. Photoshot uses separately written expressions and response-data integration; this is not a claim that Konva’s filter was installed or copied verbatim.

- [Konva · Contrast.ts (reference)](https://github.com/konvajs/konva/blob/master/src/filters/Contrast.ts)

[Full website article](https://slohero.com/photoshot/guides/brightness-contrast/en/) · [All engineering notes](README.md)
