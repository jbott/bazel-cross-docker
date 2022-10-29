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
  image layer via the rules_python hermetic toolchain.
