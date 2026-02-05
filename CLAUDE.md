# CLAUDE.md - AI Assistant Guide for LibreELEC/EmuELEC

## Project Overview

LibreELEC is a **"Just enough OS" Linux distribution build system** for the Kodi media center software. This fork includes EmuELEC emulation packages.

- Cross-platform build system for embedded Linux distributions
- Support for 9 hardware platform families with 28 device configurations
- 464 packages organized in 33 categories
- Two distribution profiles: LibreELEC (Kodi media center) and LEIoT (IoT/container)
- Multi-threaded parallel build infrastructure with stamp-based incremental builds

**License:** GPLv2
**Current Version:** OS 13.0 (development), Addon 12.80.5
**Upstream:** Forked from LibreELEC/LibreELEC.tv, itself descended from OpenELEC

---

## Repository Structure

```
LibreELEC.tv-EmuELEC/
├── distributions/          # Distribution profiles
│   ├── LibreELEC/         # Kodi media center distro (version, options, splash images)
│   └── LEIoT/             # IoT variant (no Kodi, Docker-focused)
├── projects/              # Hardware platform support (9 projects, 28 devices)
│   ├── Allwinner/         # Allwinner SoCs (A64, H2-plus, H3, H5, H6, R40)
│   ├── Amlogic/           # Amlogic SoCs (AMLGX)
│   ├── ARM/               # Generic ARM (ARMv7, ARMv8)
│   ├── Generic/           # x86_64 (Generic, Generic-legacy, gbm, wayland, x11)
│   ├── NXP/               # NXP iMX (iMX6, iMX8)
│   ├── Qualcomm/          # Qualcomm (Dragonboard)
│   ├── Rockchip/          # Rockchip (RK3288, RK3328, RK3399, RK356X, RK3576, RK3588)
│   ├── RPi/               # Raspberry Pi (RPi, RPi2, RPi4, RPi5)
│   └── Samsung/           # Samsung (Exynos)
├── packages/              # Package build recipes (464 packages in 33 categories)
│   ├── addons/            # Kodi addons (7)
│   ├── audio/             # Audio libraries (32)
│   ├── compress/          # Compression (9)
│   ├── databases/         # Databases (2)
│   ├── debug/             # Debug tools (8)
│   ├── devel/             # Development libs (76)
│   ├── emulation/         # Emulators/RetroArch cores (79)
│   ├── graphics/          # Graphics drivers/libs (34)
│   ├── lang/              # Language runtimes (10)
│   ├── linux/             # Kernel (3)
│   ├── linux-driver-addons/  # Driver addons (1)
│   ├── linux-firmware/    # Firmware blobs (12)
│   ├── mediacenter/       # Kodi packages (8)
│   ├── multimedia/        # Media libs/ffmpeg (22)
│   ├── network/           # Networking (30)
│   ├── print/             # Printing (1)
│   ├── python/            # Python packages (4)
│   ├── rust/              # Rust packages (6)
│   ├── security/          # Security libs (9)
│   ├── sysutils/          # System utilities (33)
│   ├── textproc/          # Text processing (12)
│   ├── tools/             # Build/runtime tools (29)
│   ├── virtual/           # Virtual/meta packages (16)
│   ├── wayland/           # Wayland packages (9)
│   ├── web/               # Web packages (3)
│   └── x11/               # X11 packages (9)
├── scripts/               # Build automation (24 shell + 2 Python scripts)
├── config/                # Build system configuration (15 files/dirs)
├── tools/                 # Utility scripts (19+ tools)
├── licenses/              # License files (51)
├── .github/               # Issue templates, funding config
├── Makefile               # Top-level build targets
├── CONTRIBUTING.md        # Contribution guidelines
├── README.md              # Project documentation
└── CHANGELOG              # References GitHub commit history
```

---

## Build System Architecture

### Configuration Loading Order

The build system sources configuration files in this exact order (from `config/options`):

1. **`config/functions`** - Core shell helper functions
2. **`distributions/<DISTRO>/version`** - Version numbers (OS_VERSION, ADDON_VERSION)
3. **`distributions/<DISTRO>/options`** - Distribution features (Kodi, PulseAudio, Samba, etc.)
4. **`projects/<PROJECT>/options`** - Project-level settings (architecture, bootloader, kernel)
5. **`projects/<PROJECT>/devices/<DEVICE>/options`** - Device-specific tuning (CPU, firmware, console)
6. **`config/arch.<TARGET_ARCH>`** - Architecture compiler flags and toolchain
7. **`config/graphic`** - Graphics driver configuration
8. **`config/path`** - Directory structure definitions
9. **`$ROOT/.libreelec/options`** - Local persistent overrides (optional)
10. **`$HOME/.libreelec/options`** - Global persistent overrides (optional)

Later files override earlier ones. Device options are the most specific.

### Environment Variables

```bash
PROJECT=Rockchip           # Hardware platform (default: Generic)
DEVICE=RK3399              # Specific device (default: Generic for Generic project)
ARCH=aarch64               # Target architecture (default: x86_64)
DISTRO=LibreELEC           # Distribution (default: LibreELEC)
CONCURRENCY_MAKE_LEVEL=N   # Parallel jobs (default: nproc)
VERBOSE=yes                # Verbose compilation (default: yes)
```

### Makefile Targets

```bash
make release     # Default target - release build via scripts/image
make system      # Full system build via scripts/image
make image       # Create filesystem image (mkimage)
make noobs       # NOOBS-compatible image
make clean       # Remove build artifacts
make distclean   # Clean everything including sources
make src-pkg     # Package sources into sources.tar.xz
```

### Build Scripts

| Script | Purpose |
|--------|---------|
| `scripts/image` | Main entry point - validates configs, initiates full build |
| `scripts/build` | Single-package build with dependency resolution and stamp caching |
| `scripts/build_mt` | Multi-threaded orchestrator - parallel compilation with worker pools |
| `scripts/pkgbuild` | Per-package build/install executor |
| `scripts/install` | Install package to target rootfs |
| `scripts/extract` | Extract and apply patches to source tarballs |
| `scripts/unpack` | Unpack sources (git, svn, wget, archive) |
| `scripts/mkimage` | Create final filesystem image |
| `scripts/get` | Source download dispatcher |
| `scripts/get_archive` | Download archive sources |
| `scripts/get_git` | Clone git sources |
| `scripts/get_file` | Download file sources |
| `scripts/create_addon` | Generate addon package structure |
| `scripts/checkdeps` | Verify host build dependencies |
| `scripts/autoreconf` | Run autoconf/automake |
| `scripts/pkgjson` | Generate package dependency JSON |
| `scripts/genbuildplan.py` | Python: generate parallel build plan from dependencies |
| `scripts/pkgbuilder.py` | Python: package builder |
| `scripts/uboot_helper` | U-Boot configuration helper |
| `scripts/ccache_stats` | Display ccache statistics |
| `scripts/makefile_helper` | Support for Makefile clean/distclean targets |

### Build Constraints

From `config/options`:
- **Cannot build as root** - exits with error
- **No spaces in paths** - exits with error
- Requires `gcc` and `g++` installed on host
- Uses `ccache` if available (10G default cache)
- Uses `/bin/dash` as config shell if available

---

## Package System

### Package Structure

Every package is a directory containing a `package.mk` file:

```bash
# SPDX-License-Identifier: GPL-2.0
# Copyright (C) 2018-present Team LibreELEC (https://libreelec.tv)

PKG_NAME="example"
PKG_VERSION="1.0.0"
PKG_SHA256="abc123..."
PKG_LICENSE="GPLv2"
PKG_SITE="https://example.com"
PKG_URL="https://github.com/example/$PKG_NAME/archive/$PKG_VERSION.tar.gz"
PKG_DEPENDS_TARGET="toolchain zlib openssl"
PKG_LONGDESC="Description of what this package does."
PKG_TOOLCHAIN="cmake"

PKG_CMAKE_OPTS_TARGET="-DENABLE_FEATURE=ON"

post_makeinstall_target() {
  # Custom post-installation steps
}
```

### Key Package Variables

| Variable | Description |
|----------|-------------|
| `PKG_NAME` | Package identifier (must match directory name) |
| `PKG_VERSION` | Version string or git commit hash |
| `PKG_SHA256` | Source file SHA256 checksum (required) |
| `PKG_ARCH` | Target architectures (`any`, `x86_64`, `!arm`, etc.) |
| `PKG_DEPENDS_TARGET` | Space-separated build dependencies |
| `PKG_DEPENDS_HOST` | Host tool dependencies |
| `PKG_TOOLCHAIN` | Build system: `auto`, `cmake`, `meson`, `autotools`, `manual` |
| `PKG_BUILD_FLAGS` | Optimization: `+lto`, `+speed`, `-parallel` |
| `PKG_CMAKE_OPTS_TARGET` | CMake configuration flags |
| `PKG_MESON_OPTS_TARGET` | Meson configuration flags |
| `PKG_URL` | Source download URL |
| `PKG_SITE` | Project homepage |
| `PKG_LICENSE` | License identifier |
| `PKG_LONGDESC` | Package description |
| `PKG_SECTION` | Package category |

### Build Hooks (Execution Order)

1. `pre_unpack_<target>()` - Before source extraction
2. `unpack_<target>()` - Custom extraction
3. `post_unpack_<target>()` - After extraction
4. `pre_configure_<target>()` - Before configuration
5. `configure_<target>()` - Custom configuration
6. `pre_make_<target>()` - Before compilation
7. `make_<target>()` - Custom build
8. `post_make_<target>()` - After compilation
9. `makeinstall_<target>()` - Custom install
10. `post_makeinstall_<target>()` - After install

Where `<target>` is one of: `host`, `target`, `init`, `bootstrap`

### Dependency Syntax

```bash
PKG_DEPENDS_TARGET="toolchain zlib openssl"      # Target dependencies
PKG_DEPENDS_HOST="autoconf automake"              # Host tool dependencies
PKG_DEPENDS_TARGET="toolchain ffmpeg:target"      # Explicit target qualifier
```

### Patches

Patches go in `packages/<category>/<name>/patches/` with numeric prefix:
```
packages/multimedia/ffmpeg/patches/001-fix-build.patch
```

---

## Distributions

### LibreELEC (main)
- Full Kodi media center
- PulseAudio, Bluetooth, BluRay, DVD support
- Samba server/client, NFS, OpenVPN, WireGuard
- SSH, nano editor, cron, installer
- Joystick, CEC, IR remote support

### LEIoT (IoT variant)
- No Kodi (`MEDIACENTER="no"`)
- No PulseAudio
- Docker container support
- Minimal appliance OS

---

## Hardware Platforms

| Project | Devices | Architecture |
|---------|---------|-------------|
| Allwinner | A64, H2-plus, H3, H5, H6, R40 | arm / aarch64 |
| Amlogic | AMLGX | aarch64 |
| ARM | ARMv7, ARMv8 | arm / aarch64 |
| Generic | Generic, Generic-legacy, gbm, wayland, x11 | x86_64 |
| NXP | iMX6, iMX8 | arm / aarch64 |
| Qualcomm | Dragonboard | aarch64 |
| Rockchip | RK3288, RK3328, RK3399, RK356X, RK3576, RK3588 | arm / aarch64 |
| RPi | RPi, RPi2, RPi4, RPi5 | arm / aarch64 |
| Samsung | Exynos | aarch64 |

### Architecture Support

- **aarch64** - CPUs: cortex-a35, a53, a55, a57, a72, a73, a76; Variants: armv8-a, armv8.2-a
- **arm** - CPUs: cortex-a5, a7, a8, a9, a15, a17, ARM1176JZF-S; FPU: NEON, VFP
- **x86_64** - Variants: x86-64, x86-64-v2, x86-64-v3

---

## Development Workflows

### Building a Complete System

```bash
PROJECT=Rockchip DEVICE=RK3399 ARCH=aarch64 make release
# Output: target/LibreELEC-RK3399.aarch64-13.0-devel.img.gz
```

### Building a Single Package

```bash
scripts/build <package-name>
scripts/build kodi
scripts/build ffmpeg:target
scripts/build gcc:host
```

### Adding a New Package

1. Create directory: `mkdir -p packages/<category>/<name>`
2. Create `package.mk` with required variables (see template above)
3. Add patches in `patches/` subdirectory if needed
4. Test: `scripts/build <name>`

### Modifying an Existing Package

1. Edit `package.mk`
2. Clean stamps: `rm -rf build.LibreELEC-*/<name>-*` and `rm -f target/*/stamps/<name>/build_target`
3. Rebuild: `scripts/build <name>`

### Adding a New Device

1. Create: `mkdir -p projects/<project>/devices/<device>`
2. Add `options` file with TARGET_CPU, firmware, graphics, kernel settings
3. Build: `PROJECT=<project> DEVICE=<device> ARCH=<arch> make release`

### Docker Builds

Dockerfiles available for: Debian Bookworm, Debian Trixie, Ubuntu Jammy (22.04), Noble (24.04), Questing (25.10), Resolute (26.04).

```bash
# See tools/docker/ for Dockerfiles and tools/docker/README.md for instructions
```

---

## Tools

| Tool | Purpose |
|------|---------|
| `tools/adjust_kernel_config` | Modify kernel configuration |
| `tools/check_kernel_config` | Validate kernel config |
| `tools/change_addon_version` | Update addon versions |
| `tools/dashboard` | Build monitoring dashboard |
| `tools/distro-tool` | Distribution management |
| `tools/download-tool` | Source download manager |
| `tools/download-cleaner` | Clean unused downloads |
| `tools/mkpkg/` | Package creation utilities |
| `tools/packages-checker` | Validate package metadata |
| `tools/pkgcheck` | Package verification |
| `tools/pkginfo` | Package information display |
| `tools/update-pkg` | Update package versions |
| `tools/update-scan` | Scan for package updates |
| `tools/update-functions` | Shared update utilities |
| `tools/viewconfig` | View build configuration |
| `tools/viewplan` | View build plan |
| `tools/mtstats.py` | Multi-thread build statistics |
| `tools/fixlecode.py` | Fix LE code formatting |
| `tools/repo-tool` | Repository management |

---

## Key Conventions

### Shell Scripts
- `#!/bin/bash` shebang
- SPDX license header: `# SPDX-License-Identifier: GPL-2.0`
- Use `die()` for fatal errors
- Use `print_color()` for colored output

### Package Files
- Always include `PKG_SHA256` for reproducible builds
- First dependency should be `toolchain`
- Use build system variables (`${SYSROOT_PREFIX}`, `${TARGET_PREFIX}`), never absolute paths
- Keep `PKG_LONGDESC` descriptive

### Commit Messages
```
package-name: short description

Longer description explaining why the change was made.
```

### Pull Requests
- One feature per PR
- Squash commits
- Use topic branches

---

## Important Pitfalls

**Do not:**
- Modify `config/functions` or `config/options` without deep understanding of cascading effects
- Skip `PKG_SHA256` checksums
- Use absolute paths in `package.mk` files
- Omit dependencies from `PKG_DEPENDS_TARGET`
- Add unnecessary packages (the philosophy is "Just enough OS")
- Build as root
- Use paths with spaces

**Do:**
- Follow patterns from existing similar packages
- Test with `scripts/build <package>` before full builds
- Include all dependencies
- Use appropriate `PKG_TOOLCHAIN` value
- Check `PKG_ARCH` for platform compatibility
- Clean stamps when modifying packages

---

## Build System Internals

### Stamp-based Incremental Builds
Stamps in `target/<build>/stamps/<pkg>/` track build phases (`build_target`, `install_target`). Delete stamps to force rebuild of a specific phase.

### Multi-threaded Compilation
Controlled by `config/multithread`. Uses slot-based job allocation with per-worker MTJOBID. Build plan generated by `scripts/genbuildplan.py` from dependency graph.

### Toolchain Layout
- Host tools: `build.LibreELEC-*/<pkg>.<host_arch>/`
- Target packages: `build.LibreELEC-*/<pkg>-<version>/`
- Sysroot: `build.LibreELEC-*/sysroot/`

### Cross-Compilation Variables
```bash
TARGET_CC          # Cross-compiler
TARGET_CFLAGS      # Compilation flags
TARGET_LDFLAGS     # Linker flags
SYSROOT_PREFIX     # Target system root
```

---

## Key Files Reference

| File | Purpose |
|------|---------|
| `config/options` | Core build configuration loading and defaults |
| `config/functions` | Shell helper functions (die, listcontains, setup_toolchain, etc.) |
| `config/path` | Build directory structure definitions |
| `config/graphic` | Graphics driver configuration (gallium, xorg, vulkan) |
| `config/optimize` | Compiler optimization flags (LTO, debug, linker) |
| `config/multithread` | Parallel build orchestration |
| `config/arch.aarch64` | ARM64 compiler tuning |
| `config/arch.arm` | ARM32 compiler tuning |
| `config/arch.x86_64` | x86_64 compiler tuning |
| `config/sources` | Source download mirror URLs |
| `distributions/LibreELEC/options` | Full distribution feature flags |
| `distributions/LibreELEC/version` | Version numbers (OS 13.0, Addon 12.80.5) |
| `CONTRIBUTING.md` | Contribution guidelines and PR process |
