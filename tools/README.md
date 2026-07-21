# TermuxVoid automation tools

Scripts that automate the techniques this repo uses to make normally
Termux-incompatible packages (like **bun**) installable and usable on
Termux. Two layers:

| Tool | Runs where | What it does |
|------|-----------|--------------|
| `termuxify` | inside Termux | Wraps a glibc Linux binary **right now** — no packaging involved |
| `new-package` | anywhere (dev machine or Termux) | Scaffolds a ready-to-build `packages/<name>/` `.deb` tree in this repo's layout |

## How the trick works (the bun recipe, generalized)

Prebuilt binaries such as bun are linked against **glibc**, but Android
ships **bionic**, so they won't start normally. The fix, distilled from
`packages/bun` and `packages/opencode`:

1. The official upstream binary is downloaded **unmodified** and stored as
   `$PREFIX/share/<name>/<name>.real`. No patchelf, no proot.
2. A small C launcher is compiled on-device to `$PREFIX/bin/<name>`. It
   execs the **Termux glibc dynamic loader**
   (`$PREFIX/glibc/lib/ld-linux-aarch64.so.1`, from the `glibc` package)
   with `--library-path $PREFIX/glibc/lib` followed by the real binary and
   the user's arguments.
3. *Optionally* (bun needs this, opencode doesn't): an `LD_PRELOAD` shim
   (`lib/shim.c`) is compiled and preloaded. Android returns EACCES for the
   exact paths `/`, `/data` and `/data/data`, which breaks programs that
   walk absolute paths. The shim intercepts `open`/`stat`/`access` and
   redirects those paths (and absolute paths under the Termux home) through
   a pre-opened directory fd, using raw AArch64 syscalls so it can be built
   with `-nostdlib` and work inside any glibc process.

Two more patterns cover non-binary cases: **pip-build** (compile a Python
package natively on-device, like `python-pandas`) and **jar-launcher**
(download a JAR and install a `java -jar` launcher, like `apkeditor`).

## `termuxify` — wrap a binary on-device

Prerequisites in Termux: `pkg install clang glibc curl unzip`

```sh
# bun, which needs the syscall shim:
tools/termuxify --name bun \
    --url "https://github.com/oven-sh/bun/releases/download/bun-v1.3.14/bun-linux-aarch64.zip" \
    --shim

# a typical Go/Zig/Rust static-ish glibc binary, no shim needed:
tools/termuxify --name opencode \
    --url "https://github.com/anomalyco/opencode/releases/download/v1.18.3/opencode-linux-arm64.tar.gz" \
    --setenv GODEBUG=netdns=cgo

# a local file:
tools/termuxify --name mytool --file ./mytool
```

Flags: `--archive zip|tar.gz|tar.xz|raw` (default: guessed from extension),
`--bin NAME` if the executable inside the archive isn't named like the
command, `--setenv K=V` (repeatable) to bake environment variables into the
launcher, `--shim` for the EACCES fix (aarch64 only).

**When do I need `--shim`?** Try without it first. If the binary dies with
`permission denied` / `EACCES` touching `/`, `/data`, `/data/data` or paths
under `$HOME`, re-run with `--shim`.

Removal is printed at the end (`rm` the launcher, shim and share dir) —
nothing else is touched.

## `new-package` — scaffold a .deb package

Generates `packages/<name>/DEBIAN/{control,preinst,postinst,postrm}` from
`templates/`. The result builds with the **existing** CI workflow
(`.github/workflows/build_repo.yml`) untouched, or locally with
`dpkg-deb -b -Zxz packages/<name>/ debs/`.

```sh
# Pattern A: glibc binary (bun-style). ${VERSION} and ${ARCH_TAG} in the
# URL are resolved at install time on the device:
tools/new-package --type glibc-binary --name bun --version 1.3.14 \
    --url 'https://github.com/oven-sh/bun/releases/download/bun-v${VERSION}/bun-${ARCH_TAG}.zip' \
    --shim --homepage https://bun.sh \
    --desc 'Incredibly fast JavaScript runtime'

# Pattern B: native pip build (python-pandas-style):
tools/new-package --type pip-build --name python-pandas --version 2.2.3 \
    --pip pandas --desc 'Powerful data analysis library' \
    --depends 'python, python-pip, build-essential, cmake, ninja, libopenblas, binutils-is-llvm'

# Pattern C: JAR launcher (apkeditor-style):
tools/new-package --type jar-launcher --name apkeditor --version 1.4.9 \
    --jar-url 'https://github.com/REAndroid/APKEditor/releases/download/V1.4.9/APKEditor-1.4.9.jar' \
    --desc 'APK editing tool'
```

glibc-binary specifics:
- `--tag-aarch64` / `--tag-x86_64` set what `${ARCH_TAG}` expands to per
  architecture (defaults `linux-aarch64` / `linux-x86_64`).
- The generated `postinst` embeds the shim and wrapper C sources and
  compiles them on-device at install time — exactly like the hand-written
  bun package. The shim is automatically disabled on x86_64 (it uses raw
  AArch64 syscalls).

Generated packages are a **starting point**: review `DEBIAN/postinst`, and
add app-specific environment (cache dirs, `HOME`-relative paths) with
`--setenv` or by editing the wrapper's `setenv` block.

## Limits — what can't be automated

- **Binaries with incompatible CPU instructions** (see
  `packages/antigravity-cli`, which rewrites specific pointer-auth and
  bitfield instructions with an inline Python patcher): instruction-level
  patching is inherently binary-specific and must be done by hand.
- Binaries needing many **shared libraries** beyond what the Termux `glibc`
  package ships may need extra libs placed in `$PREFIX/glibc/lib`.
- **x86_64 shim**: the syscall shim is aarch64-only; on x86_64 the wrapper
  runs without it (rarely a problem — the EACCES quirk is an Android one,
  and x86_64 Termux is mostly emulators).

## Files

```
tools/
├── termuxify                # on-device wrapper CLI (pattern A, immediate)
├── new-package              # .deb scaffolder (patterns A, B, C)
├── lib/
│   ├── shim.c               # reusable LD_PRELOAD EACCES shim (from bun)
│   └── wrapper.c.tmpl       # glibc-loader launcher template
└── templates/
    ├── glibc-binary/DEBIAN/ # control, preinst, postinst, postrm
    ├── pip-build/DEBIAN/    # control, postinst
    └── jar-launcher/DEBIAN/ # control, postinst, postrm
```
