## Barretenberg, an optimized elliptic curve library for the bn128 curve, and PLONK SNARK prover

**this code is highly experimental, use at your own risk!**

The structured reference string contains monomials up to x^{2^20}. This SRS was generated locally and is for testing and development purposes only!

### Dependencies

cmake version 3.16 or later
clang 9 or gcc 9 or later

### Installing Dependencies (Linux)

(you may need to `sudo` to move clang into `/usr/local`. An alternative is to update PATH to point to a different directory)

```
apt-get update && apt-get install -y \
  xz-utils \
  build-essential \
  curl \
  && curl -SL http://releases.llvm.org/9.0.0/clang+llvm-9.0.0-x86_64-linux-gnu-ubuntu-18.04.tar.xz | tar -xJC . \
  && mv clang+llvm-9.0.0-x86_64-linux-gnu-ubuntu-18.04 /usr/local/clang_9.0.0

export PATH="/usr/local/clang_9.0.0/bin:$PATH"
export LD_LIBRARY_PATH="/usr/local/clang_9.0.0/lib:$LD_LIBRARY_PATH"
```

### Installing Dependencies (Mac)

```
brew install cmake
brew install llvm
```

### Getting started (Linux)

```
git clone https://github.com/AztecProtocol/barretenberg
cd barretenberg/srs_db
./srs_db/download_ignition.sh 1
cd ..
mkdir build && cd build
cmake -DLINUX_CLANG=ON ..
make [optional target name]
```

### Getting started (Mac)

```
git clone https://github.com/AztecProtocol/barretenberg
cd barretenberg/barretenberg
./srs_db/download_ignition.sh 1
cd ..
mkdir build && cd build
cmake ..
make [optional target name]
```

### Parallelise the build
Add the number of jobs you want to your `make` command. e.g. `make -j16`.


### Formatting

You should format the codebase before committing changes. Ensure `clang-format` is installed.
To format the entire codebase:

```
./format.sh
```

This will reorder `#includes` *which can potentially break your build*. This means you have not correctly included
the minimal required set of headers in some files and you should fix. Always re-test your build after running this command.

To format only staged changes:

```
./format.sh staged
```

This will *not* reorder `#includes`, and so will never break your build. It maybe desirable to install the later as a pre-commit hook.

### Tests

Each module has its own tests. To build and run, for example `ecc` tests:

```
make ecc_tests
./src/aztec/ecc/ecc_tests
```

Running the entire suite of tests using `ctest`:

```
make test
```

To compile without tests and benchmarks, use `cmake .. -DTESTING=OFF -DBENCHMARKS=OFF`

To select a test, run `<path_to_module_tests> --gtest_filter=<test_filter>*`

### Benchmarks

Some modules have benchmarks. The build targets are named `<module_name>_bench`. To build and run, for example `ecc` benchmarks.

```
make ecc_bench
./src/aztec/ecc/ecc_bench
```

A shorthand for the above is:

```
make run_ecc_bench
```

### Debug build

```
mkdir build && cd build
cmake -DCMAKE_BUILD_TYPE="Debug" ..
make
```

### Build without x64 assembly:

```
mkdir build && cd build
cmake -DDISABLE_ASM=ON ..
make
```

### WASM build

We need to install `wasi-sdk` runtime to build the WASM version. We then hook a part of the runtime so we can build
the tests, which use `gtest`.

#### OS X

```
cd ./src
curl -s -L https://github.com/CraneStation/wasi-sdk/releases/download/wasi-sdk-8/wasi-sdk-8.0-macos.tar.gz | tar zxfv -
sed -e '213i#include "../../../../wasi/stdlib-hook.h"' -i ./wasi-sdk-8.0/share/wasi-sysroot/include/stdlib.h
```

#### Linux

```
cd ./src
curl -s -L https://github.com/CraneStation/wasi-sdk/releases/download/wasi-sdk-8/wasi-sdk-8.0-linux.tar.gz | tar zxfv -
sed -e $'213i\\\n#include "../../../../wasi/stdlib-hook.h"' -i ./wasi-sdk-8.0/share/wasi-sysroot/include/stdlib.h
```

To build:

```
mkdir build-wasm && cd build-wasm
cmake -DWASM=ON ..
make -j16 barretenberg.wasm
```

There will be a binary at `./src/aztec/barretenberg.wasm` that can be copied to `barretenberg.js` for use in node and the browser.

#### Testing

To run the tests, you'll need to install `wasmtime`.

```
curl https://wasmtime.dev/install.sh -sSf | bash
```

The WASM build does not currently support `ctest` meaning you must run each modules tests individually.
Tests can be run like:

```
wasmtime --dir=.. ./src/aztec/ecc/ecc_tests
```

### Modules

The following is the tree of module targets. e.g. to build `keccak` just `make keccack` (ignore the path).

```
crypto
  |_keccak
    blake2s
    pedersen
    sha256
    schnorr
ecc
noir
  |_noir_cli
    noir
numeric
plonk
  |_composer
    pippenger_bench
    proof_system
    reference_string
    transcript
polynomials
rollup
srs
stdlib
  encryption
    |_schnorr
  merkle_tree
  hash
    |_keccak
      blake2s
      pedersen
      sha256
  primitives
    |_stdlib_bit_array
      stdlib_bool
      stdlib_byte_array
      stdlib_field
      stdlib_uint
```

Their test targets are at `<module_name>_tests`.