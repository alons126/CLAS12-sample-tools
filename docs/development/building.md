# Developer build reference

Use direct CMake builds for source development. Normal ifarm operation uses [`run.csh`](../../run.csh), which reads [`config/run.json`](../../config/run.json) and builds LUND applications automatically.

## Fresh Debug build

```bash
cmake \
    -S . \
    -B build/debug \
    -G "Unix Makefiles" \
    -DCMAKE_BUILD_TYPE=Debug
cmake \
    --build build/debug \
    --parallel 4
```

`-S .` tells CMake to read the source from your current directory. `-B build/debug` puts generated build files and compiled code in that separate directory. CMake saves its selected compiler, libraries, and options there in a cache. Use different build directories for different compilers, ROOT installations, or build types so an old cache does not select the wrong settings.

The code adopts the C++ standard published by ROOT and requires at least C++17. It does not force a different standard that might be incompatible with the selected ROOT build.

## After moving or renaming a checkout

For example, renaming a checkout from `CLAS12-sample-generator` to `CLAS12-sample-tools` leaves the old absolute paths in its CMake cache. The cache is the saved build configuration. CMake then reports that its cache directory or source directory does not match, and the application never starts.

When building, the [LUND build launcher](../../src/launcher/workflow.py) compares the source and build paths recorded in the cache with the current paths. If either differs, it prints the old and current paths, deletes only `<build-dir>/CMakeCache.txt` and `<build-dir>/CMakeFiles/` with its contents, and configures again. Here, `<build-dir>` is the selected build directory. Other files in that directory remain. The launcher checks both deletion targets before changing either one and rejects symbolic links or unexpected file types. A read or deletion failure stops the workflow before configuration or LUND creation.

This removes custom options saved only in the cache. The new configuration uses the launcher settings and current environment. Matching caches remain in place. `--build false` skips the check and deletion. The check does not detect a compiler or ROOT change at the same paths; select a fresh build directory for those changes.

With code that does not yet perform this recovery, or when building directly with CMake, select an unused build directory. For the physical LUND converter, add `--build-dir build/release-new` to your existing creation command. Keep its sample options and use your actual writable output parent. This leaves the previous build directory untouched. The [ifarm checkout cleanup](../submit-simulation/ifarm-environment.md#disposable-checkout) preserves custom build directories only when they are inside `build/`.

## Select ROOT explicitly

If CMake cannot find the intended ROOT installation, configure a fresh tree with its prefix:

```bash
cmake \
    -S . \
    -B build/debug-with-root \
    -DCMAKE_BUILD_TYPE=Debug \
    -DCMAKE_PREFIX_PATH="$(root-config --prefix)"
cmake \
    --build build/debug-with-root \
    --parallel 4
```

Confirm that `root-config` names the intended installation before configuring.

## Build one LUND source

Both applications are enabled by default. A focused build can omit the unused application and its ROOT components:

```bash
cmake \
    -S . \
    -B build/uniform-only \
    -DCMAKE_BUILD_TYPE=Debug \
    -DBUILD_GENIE=OFF
cmake \
    --build build/uniform-only \
    --parallel 4
```

```bash
cmake \
    -S . \
    -B build/physical-only \
    -DCMAKE_BUILD_TYPE=Debug \
    -DBUILD_UNIFORM=OFF
cmake \
    --build build/physical-only \
    --parallel 4
```

`BUILD_GENIE=OFF` omits the physical LUND converter. `BUILD_UNIFORM=OFF` omits the uniform LUND creator and monitoring code.

## Check the built interface

```bash
build/debug/apps/uniform-lund-creator --help
build/debug/apps/event-generator-to-lund-converter --help
```

Then run a small sample using fixed nonzero seeds through the code you changed. Inspect its files, event counts, LUND creation log, and printed messages. Build commands alone never create LUND files or submit simulation.
