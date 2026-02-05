# CLAUDE.md - AI Assistant Guide for LibreELEC/EmuELEC

**Last Updated:** 2026-02-05

## Project Overview

LibreELEC is a **"Just enough OS" Linux distribution build system** for the Kodi media center software. This repository contains:
- Complete cross-platform build system for embedded Linux distributions
- Support for multiple hardware platforms (ARM, ARM64, x86_64, Rockchip, Amlogic, RPi, etc.)
- Modular package management system with 1000+ packages
- Customizable distribution and device configurations
- Parallel multi-threaded build infrastructure

**License:** GPLv2  
**Upstream:** Forked from OpenELEC; uses GeeXboX/OpenBricks build system foundation

---

## 🏗️ Repository Structure

```
LibreELEC.tv-EmuELEC/
├── distributions/          # Distribution profiles (LibreELEC, LEIoT)
│   ├── LibreELEC/         # Main distribution config
│   └── LEIoT/             # IoT variant
├── projects/              # Hardware platform support
│   ├── ARM/               # Generic ARM (32-bit)
│   ├── Allwinner/         # Allwinner SoCs
│   ├── Amlogic/           # Amlogic SoCs (S905, S912, etc.)
│   ├── Generic/           # x86_64 generic
│   ├── NXP/               # NXP iMX platforms
│   ├── Qualcomm/          # Qualcomm Snapdragon
│   ├── RPi/               # Raspberry Pi
│   ├── Rockchip/          # Rockchip SoCs (RK3399, etc.)
│   └── Samsung/           # Samsung Exynos
├── packages/              # Package build recipes (1000+ packages)
│   ├── addons/            # Kodi addons
│   ├── audio/             # Audio libraries and tools
│   ├── compress/          # Compression utilities
│   ├── databases/         # Database engines
│   ├── debug/             # Debugging tools
│   ├── devel/             # Development libraries
│   ├── emulation/         # Emulation packages (RetroArch, cores)
│   ├── graphics/          # Graphics drivers and libraries
│   ├── lang/              # Language runtimes (Python, etc.)
│   ├── linux/             # Linux kernel and modules
│   ├── linux-firmware/    # Firmware blobs
│   ├── mediacenter/       # Kodi and related packages
│   ├── multimedia/        # Media libraries (ffmpeg, etc.)
│   ├── network/           # Networking tools
│   ├── python/            # Python packages
│   ├── rust/              # Rust packages
│   ├── security/          # Security libraries
│   ├── sysutils/          # System utilities
│   ├── textproc/          # Text processing tools
│   └── tools/             # Build and runtime tools
├── scripts/               # Build automation scripts
│   ├── build              # Single package build script
│   ├── build_mt           # Multi-threaded build orchestrator
│   ├── image              # Image generation script
│   ├── pkgbuild           # Per-package build executor
│   └── [many helpers]     # Extract, install, unpack, etc.
├── config/                # Build system configuration
│   ├── functions          # Core shell functions
│   ├── options            # Default build options
│   ├── arch.aarch64       # ARM64 architecture config
│   ├── arch.arm           # ARM32 architecture config
│   ├── arch.x86_64        # x86_64 architecture config
│   ├── multithread        # Parallel build configuration
│   ├── sources            # Source download URLs
│   └── [other configs]    # Graphics, optimization, etc.
├── tools/                 # Utility scripts
│   ├── docker/            # Docker build support
│   └── [various tools]    # Kernel config, package checkers
├── Makefile               # Top-level build targets
├── README.md              # Project documentation
├── CONTRIBUTING.md        # Contribution guidelines
└── CHANGELOG              # Version history
```

---

## 🔧 Build System Architecture

### Configuration Hierarchy

The build system loads configuration in **cascading order** (later configs override earlier):

1. **Distribution options** (`distributions/LibreELEC/options`)
   - Version, release type, branding
   - Default packages and features

2. **Project options** (`projects/ARM/options`)
   - Architecture (aarch64, arm, x86_64)
   - Bootloader, kernel target
   - Default graphic drivers

3. **Device options** (`projects/Rockchip/devices/RK3399/options`) ← **Most specific**
   - CPU tuning (cortex-a72.cortex-a53)
   - Firmware requirements (ATF, u-boot)
   - Kernel parameters, console settings

4. **Architecture config** (`config/arch.aarch64`)
   - Toolchain paths
   - CFLAGS, LDFLAGS
   - CPU-specific optimizations

### Build Targets (Makefile)

```bash
make system      # Full system build
make release     # Release build (default)
make image       # Create filesystem image
make noobs       # NOOBS-compatible image
make clean       # Clean build artifacts
make distclean   # Clean everything including sources
make src-pkg     # Package sources
```

### Build Scripts

| Script | Purpose |
|--------|---------|
| `scripts/image` | **Main entry point** - Validates configs, initiates build |
| `scripts/build` | Single-package build (handles dependencies) |
| `scripts/build_mt` | **Multi-threaded orchestrator** - Parallel compilation with worker pools |
| `scripts/pkgbuild` | Per-package build/install executor |
| `scripts/install` | Install package to target rootfs |
| `scripts/extract` | Extract and patch source tarballs |
| `scripts/unpack` | Unpack sources with git/svn/wget support |

### Build Environment Variables

```bash
PROJECT=Rockchip           # Hardware platform
DEVICE=RK3399              # Specific device
ARCH=aarch64               # Target architecture
DISTRO=LibreELEC           # Distribution name
BUILD_WITH_DEBUG=yes       # Include debug symbols
```

---

## 📦 Package Management

### Package Structure

Every package has a `package.mk` file following this template:

```bash
# SPDX-License-Identifier: GPL-2.0
# Copyright (C) 2018-present Team LibreELEC (https://libreelec.tv)

PKG_NAME="example"
PKG_VERSION="1.0.0"
PKG_SHA256="abc123..."                    # SHA256 checksum
PKG_ARCH="any"                            # Target architectures
PKG_LICENSE="GPLv2"
PKG_SITE="http://example.com"
PKG_URL="https://github.com/example/example/archive/$PKG_VERSION.tar.gz"
PKG_DEPENDS_TARGET="toolchain zlib openssl"  # Build dependencies
PKG_SECTION="multimedia"                  # Package category
PKG_SHORTDESC="Example package"
PKG_LONGDESC="Longer description..."
PKG_TOOLCHAIN="auto"                      # cmake, meson, autotools, manual

# Optional: CMake configuration
PKG_CMAKE_OPTS_TARGET="-DENABLE_FEATURE=ON \
                       -DBUILD_SHARED_LIBS=ON"

# Optional: Build hooks
pre_configure_target() {
  # Custom pre-configuration steps
}

post_makeinstall_target() {
  # Custom post-installation steps
}
```

### Key Package Variables

| Variable | Description |
|----------|-------------|
| `PKG_NAME` | Package identifier |
| `PKG_VERSION` | Version string or git hash |
| `PKG_SHA256` | Source file checksum |
| `PKG_DEPENDS_TARGET` | Build dependencies (space-separated) |
| `PKG_DEPENDS_HOST` | Host tool dependencies |
| `PKG_BUILD_FLAGS` | Optimization flags (`+lto`, `+speed`, `-parallel`) |
| `PKG_TOOLCHAIN` | Build system: `auto`, `cmake`, `meson`, `autotools`, `manual` |
| `PKG_CMAKE_OPTS_TARGET` | CMake configuration flags |
| `PKG_MESON_OPTS_TARGET` | Meson configuration flags |

### Build Hooks (Execution Order)

1. `pre_unpack_<target>()` - Before source extraction
2. `unpack_<target>()` - Custom extraction logic
3. `post_unpack_<target>()` - After extraction
4. `pre_configure_<target>()` - Before configuration
5. `configure_<target>()` - Custom configuration
6. `pre_make_<target>()` - Before compilation
7. `make_<target>()` - Custom build commands
8. `post_make_<target>()` - After compilation
9. `makeinstall_<target>()` - Custom install
10. `post_makeinstall_<target>()` - After installation

Replace `<target>` with: `host`, `target`, `init`, `bootstrap`

### Dependency Syntax

```bash
# Target dependencies (cross-compiled for device)
PKG_DEPENDS_TARGET="toolchain zlib openssl:target"

# Host dependencies (native tools)
PKG_DEPENDS_HOST="autoconf automake"

# Multiple targets
PKG_DEPENDS_TARGET="toolchain ffmpeg:target SDL2:host"
```

---

## 🎯 Common Development Workflows

### 1. Building a Complete System

```bash
# Set environment
export PROJECT=Rockchip
export DEVICE=RK3399
export ARCH=aarch64

# Full build
make release

# Output: target/LibreELEC-RK3399.aarch64-X.Y.Z.img.gz
```

### 2. Building a Single Package

```bash
# Build package with dependencies
scripts/build <package-name>

# Examples
scripts/build kodi
scripts/build ffmpeg:target
scripts/build gcc:host
```

### 3. Adding a New Package

1. **Create package directory:**
   ```bash
   mkdir -p packages/multimedia/newpkg
   ```

2. **Create `package.mk`:** (see template above)

3. **Add patches (if needed):**
   ```bash
   packages/multimedia/newpkg/patches/001-fix-issue.patch
   ```

4. **Test build:**
   ```bash
   scripts/build newpkg
   ```

### 4. Modifying an Existing Package

1. **Edit package.mk:** Update version, dependencies, or build flags
2. **Clean old build:**
   ```bash
   rm -rf build.LibreELEC-*/newpkg-*
   rm -f target/*/stamps/newpkg/build_target
   ```
3. **Rebuild:**
   ```bash
   scripts/build newpkg
   ```

### 5. Creating a New Device Configuration

1. **Create device directory:**
   ```bash
   mkdir -p projects/Rockchip/devices/RK3588
   ```

2. **Create `options` file:**
   ```bash
   # Target CPU
   TARGET_CPU="cortex-a76.cortex-a55"
   
   # Firmware
   UBOOT_FIRMWARE+=" atf"
   ATF_PLATFORM="rk3588"
   
   # Graphics
   GRAPHIC_DRIVERS="panfrost"
   
   # Kernel
   KERNEL_TARGET="Image"
   EXTRA_CMDLINE="console=uart8250,mmio32,0xfeb50000"
   ```

3. **Build:**
   ```bash
   PROJECT=Rockchip DEVICE=RK3588 ARCH=aarch64 make release
   ```

### 6. Testing Changes in Docker

```bash
# Build in Docker container
docker run --rm \
  -v $(pwd):/work \
  -e PROJECT=Generic \
  -e ARCH=x86_64 \
  ghcr.io/libreelec/libreelec-build:latest \
  make release
```

---

## 🔍 Important Conventions

### Coding Standards

1. **Shell Scripts:**
   - Use `#!/bin/bash` shebang
   - Include SPDX license header
   - Use `die()` for fatal errors
   - Use `print_color()` for colored output
   - Validate arguments early

2. **Package Files:**
   - Always include `PKG_SHA256` for reproducible builds
   - Use `PKG_DEPENDS_TARGET="toolchain ..."` to include toolchain
   - Prefer `PKG_TOOLCHAIN="auto"` when possible
   - Add meaningful `PKG_SHORTDESC` and `PKG_LONGDESC`

3. **Patches:**
   - Name patches with numeric prefix: `001-fix-foo.patch`
   - Include descriptive patch headers
   - Keep patches minimal and focused

### Git Workflow

1. **Branch naming:**
   - Feature: `feature/add-package-name`
   - Fix: `fix/package-name-issue`
   - Update: `update/package-name-version`

2. **Commit messages:**
   ```
   package-name: short description
   
   Longer description explaining why the change
   was made and what it addresses.
   ```

3. **Pull requests:**
   - One feature per PR
   - Squash commits before submitting
   - Create topic branches (not from master)

### Testing

1. **Before submitting changes:**
   ```bash
   # Clean build
   make clean
   
   # Test build
   scripts/build <changed-package>
   
   # Full system build (if critical)
   make release
   ```

2. **Verify checksums:**
   ```bash
   # Update SHA256 after version bump
   scripts/checksum <package-name>
   ```

---

## 🛠️ Helper Functions (config/functions)

Essential functions available in build scripts:

```bash
die "Error message"              # Abort with error
print_color CLR_ERROR "text"     # Colored output
listcontains "list" "item"       # Check list membership
listremoveitem "list" "item"     # Remove from list
setup_toolchain target           # Initialize cross-compiler
add_depends "pkg1 pkg2"          # Add dependencies dynamically
```

### Color Constants

```bash
CLR_ERROR    # Red
CLR_WARNING  # Yellow
CLR_INFO     # Cyan
CLR_SUCCESS  # Green
```

---

## 📋 Key Files to Reference

| File | Purpose |
|------|---------|
| `packages/readme.md` | **Comprehensive package documentation** |
| `packages/packages.mk.template` | Package template |
| `packages/packages.mk.addon_template` | Addon template |
| `config/functions` | Core shell functions |
| `config/options` | Default build options |
| `scripts/build` | Package build logic |
| `CONTRIBUTING.md` | Contribution guidelines |

---

## 🚨 Common Pitfalls for AI Assistants

### ❌ DON'T:

1. **Modify `config/functions` or core scripts** without deep understanding
   - These affect the entire build system
   - Changes can break all packages

2. **Skip `PKG_SHA256` checksums**
   - Required for reproducible builds
   - Source validation

3. **Use absolute paths in package.mk**
   - Use variables: `${SYSROOT_PREFIX}`, `${TARGET_PREFIX}`

4. **Forget to declare dependencies**
   - Missing `PKG_DEPENDS_TARGET` causes build failures

5. **Break existing device configurations**
   - Test across multiple devices when changing projects

6. **Add unnecessary packages**
   - Keep system minimal ("Just enough OS")

### ✅ DO:

1. **Follow existing patterns**
   - Look at similar packages for examples
   - Maintain consistent style

2. **Test incrementally**
   - Build individual packages first
   - Then test full system build

3. **Use parallel builds**
   - Build system supports multi-threading
   - Faster iteration

4. **Document changes**
   - Update `PKG_LONGDESC` for clarity
   - Add comments for complex logic

5. **Respect architecture limits**
   - Check `PKG_ARCH` for platform support
   - Use `!x86_64` to exclude architectures

---

## 🎓 Advanced Topics

### Build System Internals

1. **Stamp-based incremental builds:**
   - Stamps stored in `target/<build>/stamps/<pkg>/`
   - `build_target`, `install_target`, etc.
   - Delete stamps to force rebuild

2. **Multi-threaded compilation:**
   - Controlled by `config/multithread`
   - Slot-based job allocation
   - Parallel package builds

3. **Toolchain structure:**
   - Host tools: `build.LibreELEC-*/<pkg>.<host_arch>/`
   - Target packages: `build.LibreELEC-*/<pkg>-<version>/`
   - Sysroot: `build.LibreELEC-*/sysroot/`

### Cross-Compilation

```bash
# Target variables
TARGET_CC        # Cross-compiler
TARGET_CFLAGS    # Compilation flags
TARGET_LDFLAGS   # Linker flags
SYSROOT_PREFIX   # Target system root
```

### Optimization Flags

```bash
PKG_BUILD_FLAGS="+lto"      # Link-time optimization
PKG_BUILD_FLAGS="+speed"    # Optimize for speed
PKG_BUILD_FLAGS="-parallel" # Disable parallel build
```

---

## 📚 Additional Resources

- **Official Wiki:** https://wiki.libreelec.tv/
- **Forum:** https://forum.libreelec.tv/
- **Package Documentation:** `packages/readme.md`
- **IRC:** #libreelec on Libera.Chat
- **GitHub Issues:** Report bugs and feature requests

---

## 🤖 AI Assistant Checklist

When modifying this repository, always:

- [ ] Understand the package category (mediacenter, graphics, network, etc.)
- [ ] Check existing similar packages for patterns
- [ ] Verify `PKG_SHA256` checksums
- [ ] Include all build dependencies in `PKG_DEPENDS_TARGET`
- [ ] Use appropriate `PKG_TOOLCHAIN` (auto, cmake, meson, etc.)
- [ ] Test build with `scripts/build <package>`
- [ ] Follow naming conventions for patches
- [ ] Maintain minimal, focused changes
- [ ] Document changes in package descriptions
- [ ] Consider cross-platform compatibility

---

**Note:** This is a complex build system. When in doubt:
1. Reference existing packages in the same category
2. Read `packages/readme.md` for detailed documentation
3. Test changes incrementally
4. Keep modifications minimal and focused

*Generated for AI assistants working with the LibreELEC/EmuELEC codebase.*
