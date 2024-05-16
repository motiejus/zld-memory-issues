# Purpose

Demonstrate that on macos + apple silicon, `zig ld` uses prohibitive amounts of time and memory.
On larger projects, linking multiple objects in parallel with `zig ld` can easily OOM on a beefy M# with 64GB RAM + 64GB swap.


# Minimized zig repro

For a full repro, see "Initial bazel repro" below.
To minimize, I copied all the files in [objs](objs).

Zig repro: `xcrun ~/Downloads/zig-macos-aarch64-0.13.0-dev.75+5c9eb4081/zig c++ -o /tmp/foo objs/*`.

The activity monitor reports more than `14GB` for and time says:
```
real    0m12.740s
user    0m11.066s
sys     0m3.427s
```

By comparison, clang repro: `xcrun clang++ -o /tmp/foo objs/*`.
```
real    0m0.912s
user    0m3.993s
sys     0m0.806s
```


# Initial bazel repro

```
bazel build @llvm-project//llvm:opt --sandbox_debug -s
```

Copy the `cd` from last command printed by bazel, it resembles:
```
cd  /private/var/tmp/_bazel_${USER}/<hash>/execroot/_main
```

Then run:
```
zig c++ -target aarch64-macos-none -o /tmp/opt objs/*.a -pthread -ldl -Wl,-headerpad_max_install_names -Wl,-S -fno-lto
```
