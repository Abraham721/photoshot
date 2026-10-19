# 14 · Posterize: quantization boundaries and saved revisions

[Full development article and photo examples →](https://slohero.com/photoshot/guides/posterize/en/)

Quantize each RGB channel through existing lookup tables while preserving the calculation used by older documents.

## Implementation

### Revision 2 bins

For eight-bit input i, compute min(steps−1, floor(i×steps/256))/(steps−1). At four levels, inputs 0–63, 64–127, 128–191 and 192–255 map to 0, 85, 170 and 255.

### Legacy compatibility

Revision 1 rounds normalized x×(steps−1). Saved revision metadata selects the old calculation rather than silently moving boundaries in existing projects.

### Color versus channel levels

R, G and B are quantized independently. Four channel levels allow up to 4³=64 RGB combinations; two levels can still produce colored pixels. Mask and opacity blending can add intermediate values.

## Core code

[src/adjustments.ts · v0.16.0 · L541–547](https://github.com/Abraham721/photoshot/blob/v0.16.0/src/adjustments.ts#L541-L547)

```ts
        if (t === 'invert') x = 1 - x;
        if (t === 'posterize')
          x =
            a.revision === 2
              ? Math.min(v.steps - 1, Math.floor((i * v.steps) / 256)) / (v.steps - 1)
              : Math.round(x * (v.steps - 1)) / (v.steps - 1);
        return clamp(x);
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

The recorded photo uses five levels at opacity 100%, distinct from the four-level explanatory table. Existing compatibility tests check legacy pixel preservation and project roundtrips; the example is not proof of full Photoshop equivalence.

## Provenance

Quantization is an established technique. Adobe documentation informs channel-level semantics. AI-assisted work connects the calculation, allowed controls, revision metadata and existing compositing path.

[Full website article](https://slohero.com/photoshot/guides/posterize/en/) · [All engineering notes](README.md)
