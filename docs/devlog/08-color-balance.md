# 08 · Color balance: revision branches and sequential tonal responses

[Full development article and photo examples →](https://slohero.com/photoshot/guides/color-balance/en/)

Shadows, midtones and highlights share settings storage and compositing, but the math differs between the legacy branch and the two revised preservation modes.

## Implementation

### Legacy weighted blend

With l=(max RGB+min RGB)/2, the shadow/midtone/highlight weights are (1−l)², 2l(1−l), l². They sum to one; at l=0.5 the contributions are 0.25/0.5/0.25, not disjoint ranges.

### Revision 2 with preservation

Construct per-channel black, white and gamma values. Shadow and highlight settings define input references; midtone settings affect gamma. Normalize the interval before applying gamma.

### Revision 2 without preservation

balance-response.ts is applied sequentially from zone 2 down to 0: highlights → midtones → shadows. Each stage receives the previous output, unlike the initial additive weighted blend.

## Core code

[src/adjustments.ts · v0.16.0 · L650–663](https://github.com/Abraham721/photoshot/blob/v0.16.0/src/adjustments.ts#L650-L663)

```ts
if (t === 'balance' && a.revision !== 2) {
  const l = (Math.max(r, g, b) + Math.min(r, g, b)) / 2,
    sh = (1 - l) ** 2,
    hi = l * l,
    mid = 2 * l * (1 - l);
  for (let c = 0; c < 3; c++)
    out[c] = clamp(
      out[c] +
        (v['shadows' + 'RGB'[c]] * sh +
          v['midtones' + 'RGB'[c]] * mid +
          v['highlights' + 'RGB'[c]] * hi) /
          100,
    );
  if (v.preserve) preserveLuma(out, luma(r, g, b));
```

[src/adjustments.ts · v0.16.0 · L674–679](https://github.com/Abraham721/photoshot/blob/v0.16.0/src/adjustments.ts#L674-L679)

```ts
  const key = 'RGB'[c],
    black = shMax - v['shadows' + key],
    white = 255 - v['highlights' + key] + hiMin,
    gamma = 2 ** ((v['midtones' + key] - midCenter) / 100);
  out[c] = clamp((out[c] * 255 - black) / Math.max(1, white - black)) ** (1 / gamma);
}
```

[src/adjustments.ts · v0.16.0 · L683–689](https://github.com/Abraham721/photoshot/blob/v0.16.0/src/adjustments.ts#L683-L689)

```ts
for (let zone = 2; zone >= 0; zone--)
  out[c] = sampleResponse(
    balanceResponse[zone],
    v[['shadows', 'midtones', 'highlights'][zone] + key],
    -100,
    out[c],
  );
```

## Validation and limits

The portrait records midtone Red +12, Blue −10, Green 0, other ranges 0, preservation on. Preserve is an option name, not proof that a particular luminance formula is invariant for every input. Check branch selection before comparing results.

## Provenance

GIMP Color Balance documentation informed the controls and tonal-range terminology. No import of GIMP calculation code is established; these excerpts show Photoshot’s evolving implementation.

- [GIMP / Color Balance — behavior reference](https://docs.gimp.org/3.0/en/gimp-tool-color-balance.html)

[Full website article](https://slohero.com/photoshot/guides/color-balance/en/) · [All engineering notes](README.md)
