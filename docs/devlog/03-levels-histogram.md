# 03 · Levels: transfer function, inverse handle mapping, and alpha-weighted histograms

[Full development article and photo examples →](https://slohero.com/photoshot/guides/levels-histogram/en/)

Levels has three coupled responsibilities: pixel mapping, gamma-handle coordinates and histogram analysis. Keeping these in separate files makes their contracts inspectable.

## Implementation

### Transfer function

Normalize the input interval, apply inverse gamma, then map to output endpoints. The existing table builder applies shared RGB settings followed by individual-channel settings.

### Bidirectional controls

gammaPosition maps the gamma value to the input position that becomes 50% after correction. gammaAtPosition performs the inverse with logarithms; typed values and dragged handles must agree.

### Histogram semantics

Skip fully transparent pixels and weight other samples by alpha. Accumulate luminance and RGB distributions separately so transparent canvas does not create a false black peak.

## Core code

[src/adjustments.ts · v0.16.0 · L521–528](https://github.com/Abraham721/photoshot/blob/v0.16.0/src/adjustments.ts#L521-L528)

```ts
if (t === 'levels')
  for (const c of ['rgb', channel])
    x =
      (v[c + 'OutBlack'] +
        (v[c + 'OutWhite'] - v[c + 'OutBlack']) *
          clamp((x * 255 - v[c + 'Black']) / (v[c + 'White'] - v[c + 'Black'])) **
            (1 / v[c + 'Gamma'])) /
      255;
```

[src/levels-controls.ts · v0.16.0 · L2–13](https://github.com/Abraham721/photoshot/blob/v0.16.0/src/levels-controls.ts#L2-L13)

```ts
// The middle handle marks the input that becomes 50% after inverse gamma.
export function gammaPosition(black: number, white: number, gamma: number) {
  return black + (white - black) * Math.pow(0.5, clamp(gamma, 0.1, 10));
}
export function gammaAtPosition(black: number, white: number, position: number) {
  const t = clamp(
    (position - black) / Math.max(1, white - black),
    Math.pow(0.5, 10),
    Math.pow(0.5, 0.1),
  );
  return Math.round(clamp(Math.log(t) / Math.log(0.5), 0.1, 10) * 100) / 100;
}
```

[src/tone-analysis.ts · v0.16.0 · L22–28](https://github.com/Abraham721/photoshot/blob/v0.16.0/src/tone-analysis.ts#L22-L28)

```ts
const weight = data[i + 3] / 255;
if (!weight) continue;
bins[Math.round(0.2126 * data[i] + 0.7152 * data[i + 1] + 0.0722 * data[i + 2])] += weight;
channelBins.r[data[i]] += weight;
channelBins.g[data[i + 1]] += weight;
channelBins.b[data[i + 2]] += weight;
count += weight;
```

## Validation and limits

The recorded comparison uses input 18/1.12/235 and output 0/255: mean channel difference about 0.229, maximum 1. The portrait uses 12/1.08/245 instead. See tests/levels-histogram.mjs for the implementation checks; do not conflate those two inputs.

## Provenance

GIMP Levels documentation informed behavior. No GIMP source port is established; the transfer function, controls and histogram code shown here belong to Photoshot.

- [GIMP · Levels documentation](https://docs.gimp.org/3.0/en/gimp-tool-levels.html)

[Full website article](https://slohero.com/photoshot/guides/levels-histogram/en/) · [All engineering notes](README.md)
