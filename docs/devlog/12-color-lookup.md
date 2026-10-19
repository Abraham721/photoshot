# 12 · Color lookup: validated grids, interpolation and limited ICC conversion

[Full development article and photo examples →](https://slohero.com/photoshot/guides/color-lookup/en/)

Connect multiple LUT formats and supported ICC profiles to one Cube representation, then use the existing adjustment and compositing path.

## Implementation

### Input boundary

CUBE, 3DL, LOOK and CSP inputs are parsed into size, input range and RGB data, with optional shaper curves. Inputs are limited to 32 MiB and a maximum grid edge of 65; malformed numeric values, ranges and data lengths are rejected.

### Evaluation order

Apply shaper curves, normalize against the input range, then interpolate. Trilinear interpolation uses eight surrounding points; tetrahedral interpolation selects four.

### ICC engine boundary

The bundled lcms-wasm/LittleCMS components handle RGB-to-RGB Device Link and Lab/XYZ Abstract transforms. The engine is sampled into a 65³ sRGB input grid. Monitor profiles and CMYK links are rejected; grid interpolation is distinct from running the original transform per pixel.

## Core code

[src/lut.ts · v0.16.0 · L19–38](https://github.com/Abraham721/photoshot/blob/v0.16.0/src/lut.ts#L19-L38)

```ts
export function validateCube(c: Cube) {
  if (
    !c ||
    !Number.isInteger(c.size) ||
    c.size < 2 ||
    c.size > MAX_SIZE ||
    typeof c.name !== 'string' ||
    c.name.length > 200 ||
    !Array.isArray(c.min) ||
    !Array.isArray(c.max) ||
    c.min.length !== 3 ||
    c.max.length !== 3 ||
    c.min.some((x, i) => !num(x) || !num(c.max[i]) || x >= c.max[i]) ||
    !Array.isArray(c.data) ||
    c.data.length !== c.size ** 3 * 3 ||
    c.data.some((x) => !num(x) || Math.abs(x) > 65536)
  )
    fail();
  if (c.only1d !== undefined && typeof c.only1d !== 'boolean') fail();
  if (c.shaper) {
```

[src/adjustments.ts · v0.16.0 · L777–790](https://github.com/Abraham721/photoshot/blob/v0.16.0/src/adjustments.ts#L777-L790)

```ts
    if (t === 'lookup' && cube) {
      const cr = shaperValue(cube, 0, r),
        cg = shaperValue(cube, 1, g),
        cb = shaperValue(cube, 2, b);
      const size = cube.size,
        rx = clamp((cr - cube.min[0]) / (cube.max[0] - cube.min[0])) * (size - 1),
        gy = clamp((cg - cube.min[1]) / (cube.max[1] - cube.min[1])) * (size - 1),
        bz = clamp((cb - cube.min[2]) / (cube.max[2] - cube.min[2])) * (size - 1),
        lr = Math.floor(rx),
        lg = Math.floor(gy),
        lb = Math.floor(bz),
        fr = rx - lr,
        fg = gy - lg,
        fb = bz - lb;
```

[src/icc.ts · v0.16.0 · L32–37](https://github.com/Abraham721/photoshot/blob/v0.16.0/src/icc.ts#L32-L37)

```ts
  if (kind === 'link' && (input !== 'RGB ' || output !== 'RGB '))
    throw Error('현재 RGB → RGB Device Link만 지원합니다.');
  if (kind === 'abst' && (!['Lab ', 'XYZ '].includes(input) || !['Lab ', 'XYZ '].includes(output)))
    throw Error('Lab/XYZ Abstract 프로파일만 지원합니다.');
  engine ??= instantiate(
    wasmBinary
```

[src/icc.ts · v0.16.0 · L65–77](https://github.com/Abraham721/photoshot/blob/v0.16.0/src/icc.ts#L65-L77)

```ts
      transform(p, input === 'Lab ' ? LAB : XYZ, 0, output === 'Lab ' ? LAB : XYZ);
      transform(last, output === 'Lab ' ? LAB : XYZ, srgb, RGB);
    }
    const size = 65,
      total = size ** 3,
      data: number[] = [];
    for (let start = 0; start < total; start += 4096) {
      const count = Math.min(4096, total - start);
      let values = new Float32Array(count * 3);
      for (let i = 0; i < count; i++) {
        const index = start + i;
        values[i * 3] = (index % size) / (size - 1);
        values[i * 3 + 1] = (Math.floor(index / size) % size) / (size - 1);
```

## Validation and limits

The recorded portrait uses the built-in Soft Film preset, tetrahedral interpolation, dithering off and opacity 100%. It does not demonstrate ICC import. Existing tests check LUT ordering/interpolation, supported identity/invert profiles and malformed ICC tag bounds; they do not establish universal format compatibility.

## Provenance

OpenColorIO was a format/structure reference. The ICC path actually bundles lcms-wasm 1.0.5 and LittleCMS 2.16 components, whose notices remain in THIRD_PARTY_NOTICES.md and src/vendor/lcms/. Distinguish these dependencies from separately written parser and UI integration.

[Full website article](https://slohero.com/photoshot/guides/color-lookup/en/) · [All engineering notes](README.md)
