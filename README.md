# MetallibSupportPkg

A collection of utilities related to patching Metal Libraries (`.metallib`) in macOS, specifically with the goal of restoring support for legacy hardware (namely Metal 3802-based GPUs)

## Logic

MetallibSupportPkg houses the `metal_libraries` python library, which was developed to streamline Metal Library patching through the following:

1. Programmatically fetching the latest macOS Sequoia IPSW.
2. Extracts the system volume DMG from the IPSW.
3. If AEA encrypted, decrypt using [`aastuff`](https://github.com/dhinakg/aeota).
4. Mount the disk image, and extract all supported `.metallib` files
5. Patch the `.metallib` files to support Metal 3802 GPUs
6. Convert the directory into a macOS Distribution Package (PKG)

Notes regarding patching individual `.metallib` files:
1. Each `.metallib` is a collection of `.air` files
2. Certain `.metallib` files are actually FAT Mach-O files. Thus they need to be thinned manually (Apple's `lipo` utility does not support the AIR64 architecture we need)
  - [metallib/patch.py: `_thin_file()`](./metal_libraries/metallib/patch.py)
3. Each `.metallib` file is actually a collection of `.air` files. Need to extract them using [zhouwei's format](https://github.com/zhuowei/MetalShaderTool)
  - metallib/patch.py: `_unpack_metallib_to_air()`](./metal_libraries/metallib/patch.py)
4. `.air` files need to next be decompiled to `.ll` (LLVM IR) using Apple's `metal-objdump` utility
  - [metallib/patch.py: `_decompile_air_to_ll()`](./metal_libraries/metallib/patch.py)
5. With the LLVM IR, we can begin patching the LLVM version to v26 (compared to Sequoia's v27)
  - metallib/patch.py: `_patch_ll()`](./metal_libraries/metallib/patch.py)
6. To compile IR to .air, we use Apple's `metal` utility
  - [metallib/patch.py: `_recompile_ll_to_air()`](./metal_libraries/metallib/patch.py)
7. To pack each individual .air to a .metallib collection, we use Apple's `metallib` utility
  - [metallib/patch.py: `_pack_air_to_metallib()`](./metal_libraries/metallib/patch.py)

Once all this is finished, the resulting `.metallib` files should work with Metal 3802-based GPUs in macOS Sequoia.