# 15 · Threshold: inclusive comparison, rounding and histogram weights

[Full development article and photo examples →](https://slohero.com/photoshot/guides/threshold/en/)

Make the brightness formula and comparison order explicit when connecting binary thresholding to the shared adjustment layer.

## Implementation

### Decision contract

Revision 2 rounds 0.3R+0.59G+0.11B from eight-bit RGB, then tests greater than or equal to the threshold. At 128, gray 127 is black and 128 is white. Pure red, green and blue give decision values 77, 150 and 28.

### Histogram limitation

The shared histogram uses 0.2126, 0.7152 and 0.0722. Colored pixels may therefore occupy a histogram position different from the threshold decision value. This difference remains documented, not resolved by the presence of a graph.

### Alpha and compositing

Threshold emits black or white RGB but does not turn white into transparency. The shared path preserves original alpha; layer opacity and masks can mix the binary result with original colors.

## Core code

[src/adjustments.ts · v0.16.0 · L728–735](https://github.com/Abraham721/photoshot/blob/v0.16.0/src/adjustments.ts#L728-L735)

```ts
    if (t === 'threshold')
      out.fill(
        (a.revision === 2 ? Math.round(0.3 * R + 0.59 * G + 0.11 * B) : luma(r, g, b) * 255) >=
          v.threshold
          ? 1
          : 0,
      );
    if (t === 'gradient') {
```

[src/adjustments.ts · v0.16.0 · L190–200](https://github.com/Abraham721/photoshot/blob/v0.16.0/src/adjustments.ts#L190-L200)

```ts
  invert: { name: '반전', description: 'RGB 색상을 반전합니다.', controls: [] },
  posterize: {
    name: '포스터화',
    description: '각 채널의 명암 단계 수를 줄입니다.',
    controls: [control('steps', '단계', 2, 256, 6)],
  },
  threshold: {
    name: '한계값',
    description: '기준 밝기에 따라 검정과 흰색으로 나눕니다.',
    controls: [control('threshold', '한계값', 0, 255, 128)],
  },
```

[src/adjustments.ts · v0.16.0 · L836–853](https://github.com/Abraham721/photoshot/blob/v0.16.0/src/adjustments.ts#L836-L853)

```ts
    beforeRGB: RGB = [0, 0, 0],
    mixed: RGB = [0, 0, 0];
  for (let i = start; i < end; i += 4) {
    if (!data[i + 3]) continue;
    let amount = opacity * (mask ? mask[i + 3] / 255 : 1);
    if (blend === 'dissolve') amount = pixelNoise(i / 4) < amount ? 1 : 0;
    if (!amount) continue;
    compiled(data[i], data[i + 1], data[i + 2], out);
    for (let c = 0; c < 3; c++) beforeRGB[c] = data[i + c] / 255;
    if (a.values.dither && (a.type === 'lookup' || a.type === 'gradient'))
      for (let c = 0; c < 3; c++) out[c] = clamp(out[c] + (pixelNoise(i / 4) - 0.5) / 255);
    blendRGB(beforeRGB, out, blend, mixed);
    for (let c = 0; c < 3; c++)
      data[i + c] = Math.round(clamp(beforeRGB[c] + (mixed[c] - beforeRGB[c]) * amount) * 255);
  }
}
```

## Validation and limits

The saved portrait uses threshold 142 and opacity 100%. Existing tests cover alpha, project roundtrip and compositor parity. The histogram coefficient difference and complete Photoshop equivalence remain outside those claims.

## Provenance

Thresholding and weighted display-RGB brightness are established operations. AI-assisted integration connects their ordering, controls and saved settings; this is not a new binarization method or a physical linear-light luminance calculation.

[Full website article](https://slohero.com/photoshot/guides/threshold/en/) · [All engineering notes](README.md)
