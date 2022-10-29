# Bazel Cross-compile Test Workspace

This demonstrates building a linux x86_64 image from multiple host
architectures. I've tested with M1 macOS, as well as Linux x86_64.

## Building and testing

```
# Should succeed on all platforms
bazelisk build ...

# Executes test.py using the host hermetic toolchain
bazelisk run :test

# Executes test.py using the linux/amd64 container image
bazelisk run :test_image
```

## Known downsides

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
