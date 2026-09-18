---
name: mortar
description: Compute a mortar firing solution from two map grid coordinates — bearing in degrees, compass direction, and range in meters. Use when the user runs /mortar or gives a self/observer position and a target position and wants azimuth + distance (War Dogs, Arma, Squad, Foxhole, Hell Let Loose, any grid map).
allowed-tools: Bash
---

# mortar — Grid Firing Solution

Turn "I'm here, target is there" into one line: **bearing + compass direction, range in meters.**

Output is the answer and nothing else. No working, no table, no caveats, no
restating the inputs. The user is mid-firefight.

## 1. Parse the input

`/mortar <my coord> <target coord>`. Accept whatever shape it arrives in — do not
ask the user to reformat:

| Input shape | Reading |
|---|---|
| `70.2,99.8 68.58,104.18` | first pair = self, second = target, `X,Y` order |
| `x70.2 y99.8 -> x68.58 y104.18` | labels win over position |
| `y99.8, x70.2` / `y104.18 x68.58` | labels win: this is still X=70.2, Y=99.8 |
| two bare pairs on separate lines | first = self, second = target |
| `me: ... target: ...` | labels win |

Rules:

- **Axis labels always override order.** If `x` and `y` are written out, use them,
  even when Y is printed first.
- **Unlabeled pairs are `X,Y`** (easting first) — the map-grid convention.
- Strip stray `()` `[]` `:` `=` `m` `km` and commas used as separators.
- Decimal commas (`70,2`) only when it is unambiguous; otherwise treat `,` as a
  separator.
- Whoever is first is the firing position, second is the target. If the user says
  "from"/"to", "me"/"enemy", "obs"/"tgt", honor that instead.
- Only one coordinate given, or three+ numbers with no labels → ask one short
  question. Never guess which is which.

## 2. Compute

Always run the math, never eyeball it:

```bash
python3 -c "
import math
sx,sy,tx,ty = 70.2,99.8,68.58,104.18   # self X, self Y, target X, target Y
scale = 100                             # meters per grid unit
dx,dy = tx-sx, ty-sy
rng = math.hypot(dx,dy)*scale
brg = math.degrees(math.atan2(dx,dy)) % 360
print(round(brg), round(rng))
"
```

- `dx = targetX - selfX`, `dy = targetY - selfY`
- **Bearing** = `atan2(dx, dy)` in degrees, normalized to `0–360`. Note the
  argument order: `dx` first. 0° = grid north, 90° = east, clockwise.
- **Range** = `hypot(dx, dy) × scale`

### Scale

Default **1 grid unit = 100 m** (a coordinate like `70.2` is 70 keys + 20 m).
This is what the two-decimal `x70.2 / y104.18` form means on most tactical maps.

Override the default when:

- the user states a scale (`1km grids`, `units are meters`) — use theirs;
- the numbers are big and integral (6-figure `123456 / 654321`) — treat as meters;
- the computed range lands absurdly far outside mortar envelope (< 5 m or
  > 20,000 m) — the scale is wrong; adjust by the obvious factor of 10/100/1000 and
  append a single trailing note naming the assumed scale.

### North convention

Y increasing = grid north. If the user says their map's Y grows downward/southward,
negate `dy` before the atan2 and recompute.

## 3. Compass direction

16-point rose from the bearing, each sector 22.5° wide:

`N` 348.75–11.25 · `NNE` 11.25–33.75 · `NE` 33.75–56.25 · `ENE` 56.25–78.75 ·
`E` 78.75–101.25 · `ESE` 101.25–123.75 · `SE` 123.75–146.25 · `SSE` 146.25–168.75 ·
`S` 168.75–191.25 · `SSW` 191.25–213.75 · `SW` 213.75–236.25 · `WSW` 236.25–258.75 ·
`W` 258.75–281.25 · `WNW` 281.25–303.75 · `NW` 303.75–326.25 · `NNW` 326.25–348.75

The letters must agree with the number. A bearing of 340 is **NNW**, never NE —
if they disagree, the sign of `dx`/`dy` got flipped.

## 4. Output

Exactly one line:

```
340° NNW, 467 m
```

- Bearing rounded to a whole degree, zero-padded to 3 digits (`007° N`).
- Range rounded to a whole meter under 1000 m, to the nearest 10 m above that.
- Nothing else on the line. No "the target is", no bullet, no bold.

Additions are allowed **only** in these cases, each one short line under the answer:

- an assumed non-default scale — `assuming 1 unit = 1 km`
- range outside a mortar's reach — `out of range for 60mm (max ~2000 m)`
- ambiguous input that you resolved rather than asked about — `read as X,Y`

If the user asks for the working, *then* show dx/dy and the two formulas.

## 5. Repeat fire

If the user follows up with only a new target, reuse the previous firing position.
If they give only a new self position, reuse the previous target. Same one-line
output each time — never re-explain the method.
