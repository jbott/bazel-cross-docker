# Bazel Cross-compile Test Workspace

This demonstrates building a linux x86_64 image from multiple host
architectures. I've tested with M1 macOS, as well as Linux x86_64.

## Building and testing

```console
# Should succeed on all platforms
$ bazelisk build ...
...

# Executes test.py using the host hermetic toolchain
$ bazelisk run :test
INFO: Analyzed target //:test (0 packages loaded, 0 targets configured).
INFO: Found 1 target...
Target //:test up-to-date:
  bazel-bin/test
INFO: Elapsed time: 0.146s, Critical Path: 0.00s
INFO: 1 process: 1 internal.
INFO: Build completed successfully, 1 total action
INFO: Build completed successfully, 1 total action
Hello world!
3.10.6 (main, Aug  2 2022, 20:27:59) [Clang 14.0.3 ]

# Executes test.py using the linux/amd64 container image with the target hermetic toolchain
$ bazelisk run :test_image
INFO: Analyzed target //:test_image (0 packages loaded, 0 targets configured).
INFO: Found 1 target...
Target //:test_image up-to-date:
  bazel-out/darwin_arm64-fastbuild-ST-4a519fd6d3e4/bin/test_image-layer.tar
INFO: Elapsed time: 0.140s, Critical Path: 0.00s
INFO: 1 process: 1 internal.
INFO: Build completed successfully, 1 total action
INFO: Build completed successfully, 1 total action
Loaded image ID: sha256:c0ab344bcb8cd2f8a3361b3adc4ec4c17e3634f299b7e6ccb7dc7a16f8ff743f
Tagging c0ab344bcb8cd2f8a3361b3adc4ec4c17e3634f299b7e6ccb7dc7a16f8ff743f as bazel:test_image
WARNING: The requested image's platform (linux/amd64) does not match the detected host platform (linux/arm64/v8) and no specific platform was requested
Hello world!
3.10.6 (main, Aug  2 2022, 18:26:10) [Clang 14.0.3 ]

# Executes test.py using the linux/amd64 container image with the container toolchain
$ bazelisk run :test_image --extra_toolchains=@io_bazel_rules_docker//toolchains:container_py_toolchain
INFO: Build option --extra_toolchains has changed, discarding analysis cache.
INFO: Analyzed target //:test_image (0 packages loaded, 19931 targets configured).
INFO: Found 1 target...
Target //:test_image up-to-date:
  bazel-out/darwin_arm64-fastbuild-ST-4a519fd6d3e4/bin/test_image-layer.tar
INFO: Elapsed time: 2.066s, Critical Path: 1.74s
INFO: 10 processes: 5 internal, 5 darwin-sandbox.
INFO: Build completed successfully, 10 total actions
INFO: Build completed successfully, 10 total actions
32e5737d7d0f: Loading layer [==================================================>]  30.72kB/30.72kB
Loaded image ID: sha256:1d8e6f4f23bf5ce94ddc1fb6b9cbcb21bf58598364adcb250a4437c6555ec36b
Tagging 1d8e6f4f23bf5ce94ddc1fb6b9cbcb21bf58598364adcb250a4437c6555ec36b as bazel:test_image
WARNING: The requested image's platform (linux/amd64) does not match the detected host platform (linux/arm64/v8) and no specific platform was requested
Hello world!
3.9.2 (default, Feb 28 2021, 17:03:44)
[GCC 10.2.1 20210110]

```

## Known drawbacks

- The py3_image rule packages a python toolchain, and a second is added to the
  image layer via the rules_python hermetic toolchain. This can be changed by
  passing the container_py_toolchain via the `--extra_toolchains` flag, but I
  have not yet figured out how to select this toolchain automatically.

  ```
  $ bazelisk build :test_image --extra_toolchains=@io_bazel_rules_docker//toolchains:container_py_toolchain
  ...

  $ tar -tvf $(bazel cquery --output=files :test_image 2>/dev/null)
  drwxr-xr-x  0 0      0           0 Dec 31  1969 ./
  drwxr-xr-x  0 0      0           0 Dec 31  1969 ./app/
  drwxr-xr-x  0 0      0           0 Dec 31  1969 ./app/test_image.binary.runfiles/
  drwxr-xr-x  0 0      0           0 Dec 31  1969 ./app/test_image.binary.runfiles/__main__/
  -r-xr-xr-x  0 0      0          22 Dec 31  1969 ./app/test_image.binary.runfiles/__main__/test.py
  -r-xr-xr-x  0 0      0       14989 Dec 31  1969 ./app/test_image.binary.runfiles/__main__/test_image.binary
  dr-xr-xr-x  0 0      0           0 Dec 31  1969 ./app/__main__/
  drwxr-xr-x  0 0      0           0 Dec 31  1969 /app/
  lrwxr-xr-x  0 0      0           0 Dec 31  1969 /app/test_image.binary -> /app//test_image.binary.runfiles/__main__/test_image.binary
  drwxr-xr-x  0 0      0           0 Dec 31  1969 /app/test_image.binary.runfiles/
  drwxr-xr-x  0 0      0           0 Dec 31  1969 /app/test_image.binary.runfiles/__main__/
  lrwxr-xr-x  0 0      0           0 Dec 31  1969 /app/test_image.binary.runfiles/__main__/external -> /app//test_image.binary.runfiles

  ```
