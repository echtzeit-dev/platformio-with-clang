# PlatformIO with Clang

## Description

This is a test project for integration a LLVM/Clang toolchain with PlatformIO.

As a base project, a TeensyLC board and the Arduino framework is chosen.

## Toolchain download

For successfull compilation, you will need to download the PlatformIO-packaged toolchain package.
* Windows: https://drive.google.com/file/d/1HcVAsVbIxxidfXbI5yAAf4bxpkV4Nr8g/view?usp=sharing
* Linux: ToDo
* Mac: ToDo

Downlaod the package for your toolchain, extract it somewhere, and reference the filepath in the `file://<your path here>` part of the `platformio.ini`.

These were created by downloading https://github.com/ARM-software/LLVM-embedded-toolchain-for-Arm/releases/ and repackaging them with a `package.json` which declares the package name, version, etc. (PlatformIO needs this)

Why not the toolchain from https://github.com/llvm/llvm-project/releases/ you might ask? When attempting to use that `clang` with the `-t arm-none-eabi` flags, it failed to find any C standard library headers like `stdlib.h`. On the other hand, the ARM-software provided toolchain has a `lib/clang-runtime` folder for each ARMv6 and ARMv7 variant that includes these headers (and more), along with convenient configuration files (e.g. `armv6m_soft_nofp_nosys.cfg`), so it was used instead.

Feel free to tell my how to use the official LLVM release with a Cortex-M0+ target and the standard C library headers.

## Linking hangs on Windows

This is a peculiar bug in the provided toolchain verison, also reported [here](https://github.com/msys2/MINGW-packages/issues/5231) and [here](https://github.com/msys2/MINGW-packages/issues/61269). The compiled package seems to have some problems with thread notifications and so the linking step appears to hang. **The solution** is to wait a bit (~5 seconds), open the task manager, find the `clang++.exe` process that hangs, kill it, then press the build button in VSCode again, choose "Terminate task", then press the build button yet again. The second linking step should go through most of the time and produce the `firmware.elf`. 

## Compilation warnings

As of now, there are still bugs to be ironed out in regards to the Teensyduino core and its usage of `uint32_t` which is apparently different to GCC and are likely causing the firmware to not yet work. (Clang's `uint32_t` is `unsigned int` while GCC is `unsigned long int` or something). 

```
C:\Users\Max\.platformio\packages\framework-arduinoteensy\cores\teensy3/IntervalTimer.h:68:43: warning: implicit conversion from 'const uint32_t' (aka 'const unsigned int') to 'float' changes value from 178956970 to 178956976 [-Wimplicit-const-int-float-conversion]
                if (microseconds <= 0 || microseconds > MAX_PERIOD) return false;
                                                      ~ ^~~~~~~~~~
C:\Users\Max\.platformio\packages\framework-arduinoteensy\cores\teensy3/DMAChannel.h:800:72: warning: all paths through this function will call itself [-Winfinite-recursion]
        void destinationCircular(volatile unsigned int p[], unsigned int len) {
                                                                              ^
```

I am not yet sure how to solve these properly and retain compatibility to other libraries that may use `uint32_t`.