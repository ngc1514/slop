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
