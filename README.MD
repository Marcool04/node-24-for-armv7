# Node.js for armv7

Since node v24+ is not officially supported anymore on armv7, I decided to try to make it myself.

Building requires two patches:
- `deps/v8/src/strings/string-hasher.cc` — fixes an x86_64-only SSE2 intrinsic that breaks 32-bit host compilation
- `out/deps/crates/node_crates.*.mk` — fixes incorrect shared output paths for host/target Rust crates in cross-compile builds

When building **v24.x** (the 32-bit host build compiles V8's int64-lowering reducer), an additional patch is required:
- `deps/v8/src/compiler/turboshaft/int64-lowering-reducer.h` — adds the missing `template` disambiguator to `__ Tuple<Word32, Word32>(...)` in `LowerClz`/`LowerCtz`. GCC parses the unqualified `Tuple<Word32, Word32>` as the value-type struct instead of the member-template operation call, failing with `invalid use of 'struct ...Tuple'`. v26.x already includes the equivalent upstream fix (renamed to `Tuple`→`MakeTuple` with the `template` keyword), so this patch is gated to branches that still contain the unfixed `__ Tuple<` pattern.

## Download
Go to [releases](https://github.com/4meters/node-for-armv7/releases)

Binaries are built with `--partly-static`, because many systems still use gcc/g++12 libs.

### Install with nvm

```bash
# Replace v26.7.0 with the version you downloaded
tar -xzf node-v26.7.0-linux-armv7l.tar.gz -C "$NVM_DIR/versions/node/"
nvm use v26.7.0
```

## How to build
Run:


```bash
chmod +x ./build-for-armv7.sh

# Build latest release on default branch (v26.x)
sudo ./build-for-armv7.sh

# Build a specific branch
sudo ./build-for-armv7.sh v24.x
```


It's fully automated script, required packages are installed automatically.

Output file is `node-vX.Y.Z-linux-armv7l.tar.gz` archive in the current directory

## Additional info
See [nodejs/node issue #61794](https://github.com/nodejs/node/issues/61794#issuecomment-5302695402)