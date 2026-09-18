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
| mortar | Turns two map grid coordinates into a mortar firing solution: bearing, compass direction, range in meters. | `/mortar <my X,Y> <target X,Y>` |

### mortar

Grid coords in, firing solution out — one line, nothing else.

```sh
/mortar y99.8 x70.2 -> y104.18 x68.58
340° NNW, 467 m
```

Takes either order (`x,y` pairs or labelled axes), defaults to 1 grid unit = 100 m.

```sh
git clone https://github.com/ngc1514/slop.git ~/.slop 2>/dev/null; \
  mkdir -p ~/.claude/skills && ln -sfn ~/.slop/mortar ~/.claude/skills/mortar
```
