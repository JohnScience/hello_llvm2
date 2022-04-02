# How to emit fully-linked LLVM IR from Cargo

At the time of writing, there is an unstable Cargo feature [`build-std`], which currently also requires explicitly passing the compile target.

As far as the author figured from:

* ["LLVM Language Reference Manual. High Level Structure. Target Triple"];
* ["The Embedonomicon. Creating a custom target. Deciding on a target triple"];
* ["The Rust RFC Book. RFC #131. Detailed design"];
* ["Clang 15.0.0git documentation. CROSS-COMPILATION USING CLANG. Target Triple"];

LLVM, Clang, and rustc; all use the same definition of a "target" aka "target triple", where the latter is probably a misnomer that could be left as-is due to backward compatibility.

Among the sources, the author most liked the definition definition from ["Clang 15.0.0git documentation. CROSS-COMPILATION USING CLANG. Target Triple"].

## How to obtain the host compile target

### Method 1: Check LLVM

In [`hello_llvm`], the 4th line of output for both `debug` and `release` builds contains `target triple = "..."`:

![Output example](https://i.imgur.com/EKV0qU6.jpg)

Even though there's a chance that the host compile target of the author (`x86_64-pc-windows-msvc`) is the same as yours, it can also be different.

### Method 2: DIY (Do It Yourself)

One can use `rustc`'s [`--print` command with `cfg` argument](https://doc.rust-lang.org/rustc/command-line-arguments.html#--print-print-compiler-information) to obtain the "`cfg` values", from which one can get the host compile target,

```
rustc --print cfg
```

On machine of the author, the output is like this:

```
debug_assertions
panic="unwind"
target_abi=""
target_arch="x86_64"
target_endian="little"
target_env="msvc"
target_family="windows"
target_feature="fxsr"
target_feature="sse"
target_feature="sse2"
target_has_atomic="16"
target_has_atomic="32"
target_has_atomic="64"
target_has_atomic="8"
target_has_atomic="ptr"
target_has_atomic_equal_alignment="16"
target_has_atomic_equal_alignment="32"
target_has_atomic_equal_alignment="64"
target_has_atomic_equal_alignment="8"
target_has_atomic_equal_alignment="ptr"
target_has_atomic_load_store="16"
target_has_atomic_load_store="32"
target_has_atomic_load_store="64"
target_has_atomic_load_store="8"
target_has_atomic_load_store="ptr"
target_os="windows"
target_pointer_width="64"
target_thread_local
target_vendor="pc"
windows
```

By concatenating `target_arch="x86_64"`, `target_vendor="pc"`, `target_family="windows"`, and `target_env="msvc"` (in the author's case, **4**, not 3, items delimited with hyphens), one can obtain their host compile target. Notice that according to, for example, ["Clang 15.0.0git documentation. CROSS-COMPILATION USING CLANG. Target Triple"], `target_env` and, respectively, the hyphen before it are optional.

## How to obtain "fully-linked" LLVM-IR for the desired compile target

Here's the command:

```
cargo +nightly rustc --release --target x86_64-pc-windows-msvc -Z build-std=core,alloc -- --emit=llvm-ir -C lto -C embed-bitcode=yes
```

[`build-std`]: https://doc.rust-lang.org/cargo/reference/unstable.html#build-std
["LLVM Language Reference Manual. High Level Structure. Target Triple"]: https://llvm.org/docs/LangRef.html#target-triple
["Clang 15.0.0git documentation. CROSS-COMPILATION USING CLANG. Target Triple"]: https://clang.llvm.org/docs/CrossCompilation.html#target-triple
["The Rust RFC Book. RFC #131. Detailed design"]: https://rust-lang.github.io/rfcs/0131-target-specification.html#detailed-design
["The Embedonomicon. Creating a custom target. Deciding on a target triple"]: https://docs.rust-embedded.org/embedonomicon/custom-target.html#deciding-on-a-target-triple
[`hello_llvm`]: https://github.com/JohnScience/hello_llvm
