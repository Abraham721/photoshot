# 13 · Invert: channel tables and shared compositing

[Full development article and photo examples →](https://slohero.com/photoshot/guides/invert/en/)

The expression 1−x is short; a usable adjustment still needs controls, storage, table evaluation and compositing connected to the document model.

## Implementation

### Channel operation

The compiled per-channel path maps normalized x to 1−x and stores its result in channel lookup tables.

### Shared infrastructure

Reuse the same table evaluator, adjustment layer, masks, opacity and saved settings as other channel operations.

### Reversibility boundary

Two full inversions recover the channel value. That algebra does not establish reversibility for partially blended or masked results: the compositor mixes transformed and original pixels as a separate operation.

## Core code

[src/adjustments.ts · v0.16.0 · L539–547](https://github.com/Abraham721/photoshot/blob/v0.16.0/src/adjustments.ts#L539-L547)

```ts
              ? clamp(x ** 2.2 * 2 ** v.ev + v.offset) ** (1 / (2.2 * v.gamma))
              : srgb(clamp(linear(x) * 2 ** v.ev + v.offset) ** (1 / v.gamma));
        if (t === 'invert') x = 1 - x;
        if (t === 'posterize')
          x =
            a.revision === 2
              ? Math.min(v.steps - 1, Math.floor((i * v.steps) / 256)) / (v.steps - 1)
              : Math.round(x * (v.steps - 1)) / (v.steps - 1);
        return clamp(x);
```

[src/adjustments.ts · v0.16.0 · L550–554](https://github.com/Abraham721/photoshot/blob/v0.16.0/src/adjustments.ts#L550-L554)

```ts
    return (r, g, b, out) => {
      out[0] = tables[0][r];
      out[1] = tables[1][g];
      out[2] = tables[2][b];
    };
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

The AI-generated portrait was processed and exported in Photoshot. Existing tests cover alpha, serialization and compositor parity. They do not prove pixel equivalence with all Adobe modes or color spaces.

## Provenance

Inversion is established arithmetic. This note documents its connection to Photoshot’s existing table and document infrastructure, not a newly invented algorithm or an established import from a specific external filter.

[Full website article](https://slohero.com/photoshot/guides/invert/en/) · [All engineering notes](README.md)
