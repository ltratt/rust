# Codegen options

All of these options are passed to `rustc` via the `-C` flag, short for "codegen." You can see
a version of this list for your exact compiler by running `rustc -C help`.

## ar

This option is deprecated and does nothing.

## code-model

This option lets you choose which code model to use.

To find the valid options for this flag, run `rustc --print code-models`.

## codegen-units

This flag controls how many code generation units the crate is split into. It
takes an integer greater than 0.

When a crate is split into multiple codegen units, LLVM is able to process
them in parallel. Increasing parallelism may speed up compile times, but may
also produce slower code. Setting this to 1 may improve the performance of
generated code, but may be slower to compile.

The default value, if not specified, is 16 for non-incremental builds. For
incremental builds the default is 256 which allows caching to be more granular.

## debug-assertions

This flag lets you turn `cfg(debug_assertions)` [conditional
compilation](../../reference/conditional-compilation.md#debug_assertions) on
or off. It takes one of the following values:

* `y`, `yes`, `on`, or no value: enable debug-assertions.
* `n`, `no`, or `off`: disable debug-assertions.

If not specified, debug assertions are automatically enabled only if the
[opt-level](#opt-level) is 0.

## debuginfo

This flag controls the generation of debug information. It takes one of the
following values:

* `0`: no debug info at all (the default).
* `1`: line tables only.
* `2`: full debug info.

Note: The [`-g` flag][option-g-debug] is an alias for `-C debuginfo=2`.

## default-linker-libraries

This flag controls whether or not the linker includes its default libraries.
It takes one of the following values:

* `y`, `yes`, `on`, or no value: include default libraries (the default).
* `n`, `no`, or `off`: exclude default libraries.

For example, for gcc flavor linkers, this issues the `-nodefaultlibs` flag to
the linker.

## embed-bitcode

This flag controls whether or not the compiler puts LLVM bitcode into generated
rlibs. It takes one of the following values:

* `y`, `yes`, `on`, or no value: put bitcode in rlibs (the default).
* `n`, `no`, or `off`: omit bitcode from rlibs.

LLVM bitcode is only needed when link-time optimization (LTO) is being
performed, but it is enabled by default for backwards compatibility reasons.

The use of `-C embed-bitcode=no` can significantly improve compile times and
reduce generated file sizes. For these reasons, Cargo uses `-C
embed-bitcode=no` whenever possible. Likewise, if you are building directly
with `rustc` we recommend using `-C embed-bitcode=no` whenever you are not
using LTO.

If combined with `-C lto`, `-C embed-bitcode=no` will cause `rustc` to abort
at start-up, because the combination is invalid.

## extra-filename

This option allows you to put extra data in each output filename. It takes a
string to add as a suffix to the filename. See the [`--emit`
flag][option-emit] for more information.

## force-frame-pointers

This flag forces the use of frame pointers. It takes one of the following
values:

* `y`, `yes`, `on`, or no value: force-enable frame pointers.
* `n`, `no`, or `off`: do not force-enable frame pointers. This does
  not necessarily mean frame pointers will be removed.

The default behaviour, if frame pointers are not force-enabled, depends on the
target.

## incremental

This flag allows you to enable incremental compilation, which allows `rustc`
to save information after compiling a crate to be reused when recompiling the
crate, improving re-compile times. This takes a path to a directory where
incremental files will be stored.

## inline-threshold

This option lets you set the default threshold for inlining a function. It
takes an unsigned integer as a value. Inlining is based on a cost model, where
a higher threshold will allow more inlining.

The default depends on the [opt-level](#opt-level):

| opt-level | Threshold |
|-----------|-----------|
| 0         | N/A, only inlines always-inline functions |
| 1         | N/A, only inlines always-inline functions and LLVM lifetime intrinsics |
| 2         | 225 |
| 3         | 275 |
| s         | 75 |
| z         | 25 |

## link-arg

This flag lets you append a single extra argument to the linker invocation.

"Append" is significant; you can pass this flag multiple times to add multiple arguments.

## link-args

This flag lets you append multiple extra arguments to the linker invocation. The
options should be separated by spaces.

## link-dead-code

This flag controls whether the linker will keep dead code. It takes one of
the following values:

* `y`, `yes`, `on`, or no value: keep dead code.
* `n`, `no`, or `off`: remove dead code (the default).

An example of when this flag might be useful is when trying to construct code coverage
metrics.

## linker

This flag controls which linker `rustc` invokes to link your code. It takes a
path to the linker executable. If this flag is not specified, the linker will
be inferred based on the target. See also the [linker-flavor](#linker-flavor)
flag for another way to specify the linker.

## linker-flavor

This flag controls the linker flavor used by `rustc`. If a linker is given with
the [`-C linker` flag](#linker), then the linker flavor is inferred from the
value provided. If no linker is given then the linker flavor is used to
determine the linker to use. Every `rustc` target defaults to some linker
flavor. Valid options are:

* `em`: use [Emscripten `emcc`](https://emscripten.org/docs/tools_reference/emcc.html).
* `gcc`: use the `cc` executable, which is typically gcc or clang on many systems.
* `ld`: use the `ld` executable.
* `msvc`: use the `link.exe` executable from Microsoft Visual Studio MSVC.
* `ptx-linker`: use
  [`rust-ptx-linker`](https://github.com/denzp/rust-ptx-linker) for Nvidia
  NVPTX GPGPU support.
* `wasm-ld`: use the [`wasm-ld`](https://lld.llvm.org/WebAssembly.html)
  executable, a port of LLVM `lld` for WebAssembly.
* `ld64.lld`: use the LLVM `lld` executable with the [`-flavor darwin`
  flag][lld-flavor] for Apple's `ld`.
* `ld.lld`: use the LLVM `lld` executable with the [`-flavor gnu`
  flag][lld-flavor] for GNU binutils' `ld`.
* `lld-link`: use the LLVM `lld` executable with the [`-flavor link`
  flag][lld-flavor] for Microsoft's `link.exe`.

[lld-flavor]: https://lld.llvm.org/Driver.html

## linker-plugin-lto

This flag defers LTO optimizations to the linker. See
[linker-plugin-LTO](../linker-plugin-lto.md) for more details. It takes one of
the following values:

* `y`, `yes`, `on`, or no value: enable linker plugin LTO.
* `n`, `no`, or `off`: disable linker plugin LTO (the default).
* A path to the linker plugin.

## llvm-args

This flag can be used to pass a list of arguments directly to LLVM.

The list must be separated by spaces.

Pass `--help` to see a list of options.

## lto

This flag controls whether LLVM uses [link time
optimizations](https://llvm.org/docs/LinkTimeOptimization.html) to produce
better optimized code, using whole-program analysis, at the cost of longer
linking time. It takes one of the following values:

* `y`, `yes`, `on`, `fat`, or no value: perform "fat" LTO which attempts to
  perform optimizations across all crates within the dependency graph.
* `n`, `no`, `off`: disables LTO.
* `thin`: perform ["thin"
  LTO](http://blog.llvm.org/2016/06/thinlto-scalable-and-incremental-lto.html).
  This is similar to "fat", but takes substantially less time to run while
  still achieving performance gains similar to "fat".

If `-C lto` is not specified, then the compiler will attempt to perform "thin
local LTO" which performs "thin" LTO on the local crate only across its
[codegen units](#codegen-units). When `-C lto` is not specified, LTO is
disabled if codegen units is 1 or optimizations are disabled ([`-C
opt-level=0`](#opt-level)). That is:

* When `-C lto` is not specified:
  * `codegen-units=1`: disable LTO.
  * `opt-level=0`: disable LTO.
* When `-C lto=true`:
  * `lto=true`: 16 codegen units, perform fat LTO across crates.
  * `codegen-units=1` + `lto=true`: 1 codegen unit, fat LTO across crates.

See also [linker-plugin-lto](#linker-plugin-lto) for cross-language LTO.

## metadata

This option allows you to control the metadata used for symbol mangling. This
takes a space-separated list of strings. Mangled symbols will incorporate a
hash of the metadata. This may be used, for example, to differentiate symbols
between two different versions of the same crate being linked.

## no-prepopulate-passes

This flag tells the pass manager to use an empty list of passes, instead of the
usual pre-populated list of passes.

## no-redzone

This flag allows you to disable [the
red zone](https://en.wikipedia.org/wiki/Red_zone_\(computing\)). It takes one
of the following values:

* `y`, `yes`, `on`, or no value: disable the red zone.
* `n`, `no`, or `off`: enable the red zone.

The default behaviour, if the flag is not specified, depends on the target.

## no-stack-check

This option is deprecated and does nothing.

## no-vectorize-loops

This flag disables [loop
vectorization](https://llvm.org/docs/Vectorizers.html#the-loop-vectorizer).

## no-vectorize-slp

This flag disables vectorization using
[superword-level
parallelism](https://llvm.org/docs/Vectorizers.html#the-slp-vectorizer).

## opt-level

This flag controls the optimization level.

* `0`: no optimizations, also turns on
  [`cfg(debug_assertions)`](#debug-assertions) (the default).
* `1`: basic optimizations.
* `2`: some optimizations.
* `3`: all optimizations.
* `s`: optimize for binary size.
* `z`: optimize for binary size, but also turn off loop vectorization.

Note: The [`-O` flag][option-o-optimize] is an alias for `-C opt-level=2`.

The default is `0`.

## overflow-checks

This flag allows you to control the behavior of [runtime integer
overflow](../../reference/expressions/operator-expr.md#overflow). When
overflow-checks are enabled, a panic will occur on overflow. This flag takes
one of the following values:

* `y`, `yes`, `on`, or no value: enable overflow checks.
* `n`, `no`, or `off`: disable overflow checks.

If not specified, overflow checks are enabled if
[debug-assertions](#debug-assertions) are enabled, disabled otherwise.

## panic

This option lets you control what happens when the code panics.

* `abort`: terminate the process upon panic
* `unwind`: unwind the stack upon panic

If not specified, the default depends on the target.

## passes

This flag can be used to add extra [LLVM
passes](http://llvm.org/docs/Passes.html) to the compilation.

The list must be separated by spaces.

See also the [`no-prepopulate-passes`](#no-prepopulate-passes) flag.

## prefer-dynamic

By default, `rustc` prefers to statically link dependencies. This option will
indicate that dynamic linking should be used if possible if both a static and
dynamic versions of a library are available. There is an internal algorithm
for determining whether or not it is possible to statically or dynamically
link with a dependency. For example, `cdylib` crate types may only use static
linkage. This flag takes one of the following values:

* `y`, `yes`, `on`, or no value: use dynamic linking.
* `n`, `no`, or `off`: use static linking (the default).

## profile-generate

This flag allows for creating instrumented binaries that will collect
profiling data for use with profile-guided optimization (PGO). The flag takes
an optional argument which is the path to a directory into which the
instrumented binary will emit the collected data. See the chapter on
[profile-guided optimization] for more information.

## profile-use

This flag specifies the profiling data file to be used for profile-guided
optimization (PGO). The flag takes a mandatory argument which is the path
to a valid `.profdata` file. See the chapter on
[profile-guided optimization] for more information.

## relocation-model

This option controls generation of
[position-independent code (PIC)](https://en.wikipedia.org/wiki/Position-independent_code).

Supported values for this option are:

#### Primary relocation models

- `static` - non-relocatable code, machine instructions may use absolute addressing modes.

- `pic` - fully relocatable position independent code,
machine instructions need to use relative addressing modes.  \
Equivalent to the "uppercase" `-fPIC` or `-fPIE` options in other compilers,
depending on the produced crate types.  \
This is the default model for majority of supported targets.

#### Special relocation models

- `dynamic-no-pic` - relocatable external references, non-relocatable code.  \
Only makes sense on Darwin and is rarely used.  \
If StackOverflow tells you to use this as an opt-out of PIC or PIE, don't believe it,
use `-C relocation-model=static` instead.
- `ropi`, `rwpi` and `ropi-rwpi` - relocatable code and read-only data, relocatable read-write data,
and combination of both, respectively.  \
Only makes sense for certain embedded ARM targets.
- `default` - relocation model default to the current target.  \
Only makes sense as an override for some other explicitly specified relocation model
previously set on the command line.

Supported values can also be discovered by running `rustc --print relocation-models`.

#### Linking effects

In addition to codegen effects, `relocation-model` has effects during linking.

If the relocation model is `pic` and the current target supports position-independent executables
(PIE), the linker will be instructed (`-pie`) to produce one.  \
If the target doesn't support both position-independent and statically linked executables,
then `-C target-feature=+crt-static` "wins" over `-C relocation-model=pic`,
and the linker is instructed (`-static`) to produce a statically linked
but not position-independent executable.

## remark

This flag lets you print remarks for optimization passes.

The list of passes should be separated by spaces.

`all` will remark on every pass.

## rpath

This flag controls whether [`rpath`](https://en.wikipedia.org/wiki/Rpath) is
enabled. It takes one of the following values:

* `y`, `yes`, `on`, or no value: enable rpath.
* `n`, `no`, or `off`: disable rpath (the default).

## save-temps

This flag controls whether temporary files generated during compilation are
deleted once compilation finishes. It takes one of the following values:

* `y`, `yes`, `on`, or no value: save temporary files.
* `n`, `no`, or `off`: delete temporary files (the default).

## soft-float

This option controls whether `rustc` generates code that emulates floating
point instructions in software. It takes one of the following values:

* `y`, `yes`, `on`, or no value: use soft floats.
* `n`, `no`, or `off`: use hardware floats (the default).

## target-cpu

This instructs `rustc` to generate code specifically for a particular processor.

You can run `rustc --print target-cpus` to see the valid options to pass
here. Additionally, `native` can be passed to use the processor of the host
machine. Each target has a default base CPU.

## target-feature

Individual targets will support different features; this flag lets you control
enabling or disabling a feature. Each feature should be prefixed with a `+` to
enable it or `-` to disable it. Separate multiple features with commas.

To see the valid options and an example of use, run `rustc --print
target-features`.

Using this flag is unsafe and might result in [undefined runtime
behavior](../targets/known-issues.md).

See also the [`target_feature`
attribute](../../reference/attributes/codegen.md#the-target_feature-attribute)
for controlling features per-function.

This also supports the feature `+crt-static` and `-crt-static` to control
[static C runtime linkage](../../reference/linkage.html#static-and-dynamic-c-runtimes).

Each target and [`target-cpu`](#target-cpu) has a default set of enabled
features.

[option-emit]: ../command-line-arguments.md#option-emit
[option-o-optimize]: ../command-line-arguments.md#option-o-optimize
[profile-guided optimization]: ../profile-guided-optimization.md
[option-g-debug]: ../command-line-arguments.md#option-g-debug
