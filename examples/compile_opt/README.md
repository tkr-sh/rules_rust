# Hello World with compiler optimization

Each binary target can have its own compiler options, and these can be customised differently for different optimisation levels.
This takes three steps:

1) In your root folder `BUILD.bazel`, add the following entry:

```Starlark
config_setting(
    name = "release",
    values = {
        "compilation_mode": "opt",
    },
)
```

2) Add config option in `.bazelrc` to enable LTO together with optimized build.

```Starlark
common:opt --compilation_mode=opt
common:opt --@rules_rust//rust/settings:lto=thin
```

3) In your binary target, add the optimization flags & strip settings prefixed with -C.
For a complete list of Rust compiler optimization flag, please read the
[official cargo documentation](https://doc.rust-lang.org/cargo/reference/profiles.html).

```Starlark
load("@rules_rust//rust:defs.bzl", "rust_binary")

rust_binary(
    name = "bin",
    srcs = ["src/main.rs"],
    deps = [],
    rustc_flags = select({
       "//:release": [
            "-Ccodegen-units=1",
            "-Cpanic=abort",
            "-Copt-level=3",
            "-Cstrip=symbols",
            ],
        "//conditions:default":
        [
           "-Copt-level=0",
        ],
    }),
    visibility = ["//visibility:public"],
)
```

Build with optimization:

`bazel build --config opt //...`

And run the optimized binary:

`bazel run --config opt //...`
