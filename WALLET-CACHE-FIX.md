# AirCard-iOS Wallet cache unlink patch

This copy ports the stale Wallet artwork invalidation fix from AirCard macOS
commit `7e8979b` (released as v1.2.4).

## Source changes

- `ios-app/AppViewModel.swift`
  - Removes the old `"corrupted"` cache-file overwrite.
  - Calls `al_exploit_remove_card_cache` for both `.cache` and `.pkcache`.

- `rust-core/src/exploit.rs`
  - Adds the real cache-removal AirTraffic flow:
    - stages the Airlift symlink against the protected cache directory
    - moves `FrontFace`, `Preview`, and `PlaceHolder` to generated `removed-N` paths
    - verifies the moved leaves
    - removes the generated relocation tree
    - restores `Books/Sync/Books.plist`

- `rust-core/src/lib.rs`
  - Exposes the new Rust operation through C FFI.

- `rust-core/include/airlift.h`
  - Adds the matching FFI declaration.

- `AirliftFFI.xcframework/*/Headers/airlift.h`
  - Header copies updated so the checked-in framework exposes the new symbol.
  - The static libraries themselves must be regenerated; the included GitHub Actions
    workflow does that on a macOS runner.

- `.github/workflows/build-patched.yml`
  - Builds the Rust iOS targets, regenerates `AirliftFFI.xcframework`, and builds
    an unsigned IPA as a GitHub Actions artifact.

## Important

The checked-in `libairlift_ffi.a` binaries are not modified in-place here. The
workflow must run `build-ios.sh` before `build-ipa.sh` so the new Rust symbol is
actually linked into the IPA.
