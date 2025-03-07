# Building DNG SDK with CMake

- target DNG SDK version: v1.7.1

## How to Build

```shell-session
$ cmake -S . -B build [-DCMAKE_BUILD_TYPE=<build type>] [-DCMAKE_INSTALL_PREFIX=<installation path>]
$ cmake --build build --config release # sudo may be necessary for installing libjpeg & libjxl
$ cmake --install build  # sudo may be necessary 
```

## Testing

```shell
$ ./build/dng_validate -v build/_deps/dngsdk-src/sample_files/01_jxl_linear_raw_integer.dng
```

***

## Status

| platform              | status             |
|:----------------------|:------------------:|
| macOS (Intel)         | :heavy_check_mark: |
| macOS (Apple Silicon) | :heavy_check_mark: |
| Windows               | .dll only          |
| Linux (Ubuntu 22.04)  | :heavy_check_mark: |
