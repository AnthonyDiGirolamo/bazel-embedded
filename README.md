# bazel-embedded
**Caution:** This is alpha quality software at the moment and has not be tested in depth.
bazel-embedded is a set of tools that enable embedded development using bazel. 

At this point it is relatively easy to add support for new architectures, that have gcc based compilers. In future we will be adding clang support, so that we can make use of clangs static-analyzers. If you would like an architecture added to this repository let us know.

Current support is limited to Arm Cortex-M Devices:
- Cortex M0
- Cortex M1
- Cortex M3
- Cortex M4 (with/out fpu)
- Cortex M7 (with/out fpu)

## What is included
List of support;
- [x] Toolchains
- [ ] Static analysers 
- [X] A collection of BUILD file templates for common embedded libraries
- [x] Utilities for programming targets
- [x] Utilities for debugging targets
- [ ] Parralell execution for a test "farm" of embedded test devices

## Getting started
Add the following to your WORKSPACE file


```py
load("@bazel_tools//tools/build_defs/repo:git.bzl", "git_repository")

git_repository(
    name = "bazel_embedded",
    commit = "d0d4bfacb47bd2db67f558adc69149b4d5a915ab",
    remote = "https://github.com/silvergasp/bazel-embedded.git",
    shallow_since = "1585022166 +0800",
)

load("@bazel_embedded//:bazel_embedded_deps.bzl", "bazel_embedded_deps")

bazel_embedded_deps()

load("@bazel_embedded//platforms:execution_platforms.bzl", "register_platforms")

register_platforms()

load(
    "@bazel_embedded//toolchains/compilers/gcc_arm_none_eabi:gcc_arm_none_repository.bzl",
    "gcc_arm_none_compiler",
)

gcc_arm_none_compiler()

load("@bazel_embedded//toolchains/gcc_arm_none_eabi:gcc_arm_none_toolchain.bzl", "register_gcc_arm_none_toolchain")

register_gcc_arm_none_toolchain()
```

Enable incompatible/future support for bazel toolchain resolution by adding the following .bazelrc to your project. This won't be required in future after this becomes the default functionality in bazel. Alternatively it is possible to add this flag each time you build.
```
build --incompatible_enable_cc_toolchain_resolution
```

Cross Compile your target

```sh
bazel build //:your_target --platforms=@bazel_embedded//platforms:cortex_m0
bazel build //:your_target --platforms=@bazel_embedded//platforms:cortex_m1
bazel build //:your_target --platforms=@bazel_embedded//platforms:cortex_m3
bazel build //:your_target --platforms=@bazel_embedded//platforms:cortex_m4
bazel build //:your_target --platforms=@bazel_embedded//platforms:cortex_m7
bazel build //:your_target --platforms=@bazel_embedded//platforms:cortex_m4_fpu
bazel build //:your_target --platforms=@bazel_embedded//platforms:cortex_m7_fpu
```

Explore the examples for more in depth details...

## STM32cube support
Currently supported device library build files
- [X] stm32h7 examples can be found in examples/minimal_stm32h7_cubemx
    - Hal libraries
    - USB middlewares
    - Network libraries
    - Fatfs
    - Mbed TLS
- [ ] stm32l4 WIP 
- [ ] stm32f4 TODO

## Caveats
If your repository contains platform independant you will not be able to automatically exclude platform dependant code. For example;
package/BUILD
```py
cc_library(
    name = "can_run_on_microcontroller_only"
    ...
)
cc_library(
    name = "can_run_on_anything"
    ...
)
```
You may compile for your host;
```sh
bazel build //package:can_run_on_anything
```
You may cross compile for your microcontroller
```sh
bazel build //package/... --platforms=@bazel_embedded//platforms:cortex_m7_fpu
```
But automated skipping of targets based on compatibility is not supported. So bazel will happily attempt to compile the //package:can_run_on_microcontroller_only using your host compiler, which in almost all cases will fail.
```sh
bazel build //package/... 
```
