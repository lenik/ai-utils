# ai-utils

Encode or decode data in a repeated sequence of a pre-defined ASCII art picture.

## Tools

- **ai** — Encode files into an ASCII-art image format.
- **unai** — Decode AI-format images back to files.
- **air** — Run AI-encoded scripts (decode and execute).
- **loadimage** — Bash loadable builtin to source decoded AI images in scripts.

## Build

### Requirements

- [Meson](https://mesonbuild.com/) (≥ 0.60)
- Ninja
- C compiler (C11)
- **libglib-2.0** (development)
- **libbas-c** (development) — from this repo: `lib/libbas-c`

### Build and install

```bash
# Install libbas-c first if building from the same repo:
cd lib/libbas-c && meson setup build && ninja -C build && meson install -C build

# Then build ai-utils:
cd crypt/ai-utils
meson setup build
ninja -C build
meson install -C build   # or: sudo ninja -C build install
```

### Debian

```bash
dpkg-buildpackage -b -us -uc
```

## Usage

Encode a file:

```bash
ai -t grid myfile.txt
# -> myfile.txt.img
```

Decode:

```bash
unai myfile.txt.img
# -> myfile.txt
```

Use a template (e.g. `bridge`, `grid`): templates live under `$prefix/share/ai-utils/template/`.

## Install layout

| Path | Content |
|------|---------|
| `bin/` | `ai`, `unai`, `air` |
| `lib/` | `loadimage` (shared library) |
| `lib/shlib.d/` | `ai` (loadimage enable script) |
| `lib/systemd/system/` | `aifmt.service` (binfmt_misc; use `systemctl start aifmt`) |
| `share/ai-utils/` | `aifmt` (helper), `template/` |
| `share/bash-completion/completions/` | `ai_utils`, symlinks `ai`, `unai` |
| `share/setup/ai-utils/` | `postinst`, `prerm` (enable/disable aifmt via systemctl) |
| `share/man/man1/` | `ai.1`, `air.1`, `unai.1` |

## License

AGPL-3.0-or-later (GNU Affero General Public License v3 or later). See [debian/copyright](debian/copyright) for the full license and **Anti-AI terms**: you may not use this software (or its output) to train or operate AI/ML systems without prior written permission from the copyright holder. Use over a network (e.g. SaaS) requires offering the corresponding source to users.
