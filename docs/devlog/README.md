# Photoshot engineering notes — Parts 1–12
Concise implementation notes for the public editor snapshot. Each entry links to the full website article for development context and photo examples.
| Part | Technical focus |
|---|---|
| 01 | [Layer state, reversible masks, and a shared compositor](01-basic-tools.md) |
| 02 | [Brightness/contrast: ordered response sampling and LUT compilation](02-brightness-contrast.md) |
| 03 | [Levels: transfer function, inverse handle mapping, and alpha-weighted histograms](03-levels-histogram.md) |
| 04 | [Curves: revisioned interpolation and channel-table order](04-curves.md) |
| 05 | [Exposure: EV scaling, conversion space, and saved revisions](05-exposure.md) |
| 06 | [Vibrance: HSL gain versus sampled RGB response](06-vibrance.md) |
| 07 | [Hue/saturation: circular distance, feathering, and revisioned response](07-hue-saturation.md) |
| 08 | [Color balance: revision branches and sequential tonal responses](08-color-balance.md) |
| 09 | [Black & white: six hue weights, neutral invariance, and tint](09-black-white.md) |
| 10 | [Photo filter: compiled color matrix versus layer compositing](10-photo-filter.md) |
| 11 | [Channel mixer: RGB weights, constants and monochrome](11-channel-mixer.md) |
| 12 | [Color lookup: validated grids, interpolation and limited ICC conversion](12-color-lookup.md) |

These notes cover implementation contracts, selected code excerpts, compatibility decisions and validation scope. They do not duplicate the full narrative. Source links pin existing commits or v0.16.0; editorial updates do not change the editor implementation.
[Website journal](https://slohero.com/photoshot/journal/?lang=en) · [Repository setup](../../README.md)
