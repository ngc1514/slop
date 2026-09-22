# slop

slop skills

## Installation

Replace skill with the one you need
```sh
git clone https://github.com/ngc1514/slop.git ~/.slop && \
  mkdir -p ~/.claude/skills && \
  ln -sfn ~/.slop/<skillname> ~/.claude/skills/<skillname>
```

## Skills

| Name | Summary | Command |
|---|---|---|
| read-pp | Digests a privacy policy, ToS, or DPA into a short list of what the service reserves the right to do with your data. | `/read-pp <url \| file \| pasted text>` |
| mortar | Turns two map grid coordinates into a mortar firing solution: bearing, compass direction, range in meters. | `/mortar <my Y X> <target Y X>` |

### mortar

Grid coords in, firing solution out — one line, nothing else.

**Order matters: first pair is you (the mortar), second pair is the target.** Within each pair, unlabeled numbers are `Y X` (northing first).

Otherwise, type coords however is fastest:

- **No `x`/`y` needed** — `99.8 70.2` means `Y=99.8 X=70.2`.
- **No period needed** — a 4–5 digit number gets its last two digits as decimals: `9980` → `99.8`, `10418` → `104.18`, `0934` → `9.34`. 1–3 digit numbers are used as typed.
- **Commas optional** — `,` `/` `;` and spaces are all just separators.
- **Role labels override order** — tag pairs with `me`/`self`/`from`/`obs`/`gun` and `target`/`tgt`/`enemy`/`to` and you can give them in any order.
- **Axis labels still work** — `x70.2 y99.8` is honored even when X comes first.

Defaults to 1 grid unit = 100 m.

#### Example usage

```sh
# bare Y X pairs: me, then target
/mortar 99.8 70.2 104.18 68.58
340° NNW, 467 m

# no periods: 4–5 digit numbers get implied decimals
/mortar 9980 7020 10418 6858
340° NNW, 467 m

# role labels, target called out first
/mortar target 120 34.31 me 112.4 31.07
023° NNE, 826 m

# commas, slashes, whatever
/mortar 48.85,93.41 / 52.10,99.32
061° ENE, 674 m

# explicit axis labels, any order
/mortar x70.2 y99.8 -> x68.58 y104.18
340° NNW, 467 m
```

```sh
git clone https://github.com/ngc1514/slop.git ~/.slop 2>/dev/null; \
  mkdir -p ~/.claude/skills && ln -sfn ~/.slop/mortar ~/.claude/skills/mortar
```
