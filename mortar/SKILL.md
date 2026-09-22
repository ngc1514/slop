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

`/mortar <coord> <coord>`. Accept whatever shape it arrives in — do not ask the
user to reformat:

| Input shape | Reading |
|---|---|
| `99.8,70.2 104.18,68.58` | first pair = self, second = target, `Y,X` order |
| `99.8 70.2 104.18 68.58` | commas optional — four bare numbers are the same two pairs |
| `target 120 34.31, me 183.1 03` | role labels win over position: self is `Y=183.1 X=03` |
| `x70.2 y99.8 -> x68.58 y104.18` | axis labels win over position |
| `y99.8, x70.2` / `y104.18 x68.58` | axis labels win: X=70.2, Y=99.8 |
| two bare pairs on separate lines | first = self, second = target |
| `me 4885 9341 target 10234 3243` | implied decimals: self `Y=48.85 X=93.41`, target `Y=102.34 X=32.43` |
| `me 9 123 target 10 125` | short bare integers are taken as written: `Y=9 X=123` |

Rules:

- **Implied decimals.** Game coordinates always carry two decimal places, and
  combat coordinates normally sit in the 10s–100s. So a bare integer of **4 or 5
  digits** with no `.` has its last two digits as decimals: `4885` → `48.85`,
  `10234` → `102.34`, `0934` → `9.34`. Apply this per number, before stripping
  leading zeros. Always do it silently — no note in the output.
- **Short integers are taken as written.** A bare integer of 1–3 digits (`9`,
  `123`, `03`) is ambiguous; don't guess at a decimal, just compute with it as-is.
- Numbers that already contain a `.` are never rescaled.

- **Unlabeled numbers are `Y,X`** (northing first) — first number of every pair is
  Y, second is X. This is the default whenever the user does not write out the axes.
- **Axis labels always override order.** If `x` and `y` are written out, use them,
  even when X is printed first.
- **Role labels always override order.** `me`/`self`/`from`/`obs`/`gun` is the
  firing position, `target`/`tgt`/`enemy`/`to` is the target, wherever they appear
  in the line. With no role labels, first pair fires, second is the target.
- Commas are optional. Treat `,` `/` `;` and whitespace all as plain separators —
  a lone `,` is never a decimal point. Decimals are written with `.` (`34.31`).
- Strip stray `()` `[]` `:` `=` `m` `km` and leading zeros (`03` = 3).
- Exactly four numbers → two pairs, in order. Only one pair, or an odd count of
  numbers with no labels → ask one short question. Never guess which is which.

## 2. Compute

Always run the math, never eyeball it:

```bash
python3 -c "
import math
sy,sx,ty,tx = 99.8,70.2,104.18,68.58   # self Y, self X, target Y, target X
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
This is what the two-decimal `y104.18 / x68.58` form means on most tactical maps.

Override the default when:

- the user states a scale (`1km grids`, `units are meters`) — use theirs;
- the numbers are big and integral (6+ digits, `123456 / 654321`) — treat as
  meters (4–5 digit integers are implied decimals, see §1, not meters);
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
- ambiguous input that you resolved rather than asked about — `read as Y,X`

If the user asks for the working, *then* show dx/dy and the two formulas.

## 5. Repeat fire

If the user follows up with only a new target, reuse the previous firing position.
If they give only a new self position, reuse the previous target. Same one-line
output each time — never re-explain the method.
