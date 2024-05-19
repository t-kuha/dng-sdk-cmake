# Building DNG SDK with CMake

- target DNG SDK version: v1.7.1

## How to Build:

```shell-session
$ cmake -S . -B build [-DCMAKE_BUILD_TYPE=<build type>] [-DCMAKE_INSTALL_PREFIX=<installation path>]
$ cmake --build build    # sudo may be necessary to install libjpeg & libjxl
$ cmake --install build  # sudo may be necessary 
```

***

## Status

| platform              | status |
|-----------------------|--------|
| macOS (Intel)         | :heavy_check_mark: |
| macOS (Apple Silicon) | TODO |
| Windows               | TODO |
| Linux                 | TODO |
