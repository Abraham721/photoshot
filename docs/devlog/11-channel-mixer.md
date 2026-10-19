# 11 · Channel mixer: RGB weights, constants and monochrome

[Full development article and photo examples →](https://slohero.com/photoshot/guides/channel-mixer/en/)

Channel mixing connects an established RGB weighted sum to the existing adjustment layer. The integration work covers output controls, saved settings and shared compositing.

## Implementation

### Output contract

Each output channel has three input percentages and a constant: twelve numeric settings in total. A constant of +10 adds 0.1 before clamping. The total shown in the panel does not normalize the weights.

### Monochrome routing

With monochrome enabled, all three outputs read the red-output configuration. The same weighted value reaches R, G and B. This differs from the six hue-range controls in Black & White.

### Compositing boundary

The compiled adjustment produces transformed RGB. Layer opacity, mask and blend mode act in the shared compositor afterward; the constant is not a substitute for opacity.

## Core code

[src/adjustments.ts · v0.16.0 · L720–727](https://github.com/Abraham721/photoshot/blob/v0.16.0/src/adjustments.ts#L720-L727)

```ts
    if (t === 'mixer') {
      for (let c = 0; c < 3; c++) {
        const key = v.mono ? 'r' : 'rgb'[c];
        out[c] = clamp(
          (r * v[key + 'R'] + g * v[key + 'G'] + b * v[key + 'B'] + v[key + 'Constant']) / 100,
        );
      }
    }
```

[src/adjustments.ts · v0.16.0 · L166–184](https://github.com/Abraham721/photoshot/blob/v0.16.0/src/adjustments.ts#L166-L184)

```ts
  mixer: {
    name: '채널 혼합',
    description: '출력 채널을 원본 RGB 채널의 비율로 만듭니다.',
    controls: ['r', 'g', 'b']
      .flatMap((c) =>
        ['R', 'G', 'B', 'Constant'].map((v, i) =>
          control(
            c + v,
            ['빨강', '초록', '파랑', '상수'][i],
            -200,
            200,
            c === v.toLowerCase() ? 100 : 0,
            1,
            c,
          ),
        ),
      )
      .concat(control('mono', '단색', 0, 1)),
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

The recorded portrait uses monochrome on, R21/G72/B7, constant 0 and opacity 100%. The portrait is AI-generated; the workspace and output are actual Photoshot captures. Existing adjustment tests cover alpha, saved-project roundtrip and compositor parity, not complete Photoshop equivalence.

## Provenance

The RGB weighted sum is established mathematics. Adobe documentation informs the behavior of controls. The development record does not identify an imported external Channel Mixer implementation; AI-assisted integration uses Photoshot’s existing layer infrastructure.

[Full website article](https://slohero.com/photoshot/guides/channel-mixer/en/) · [All engineering notes](README.md)
