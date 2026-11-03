# WindowsOS

### 1. [C/C++ on WindowsOS](#ccpp-on-windows-os)
The quickest way to get a C/C++ runtime and all the includes is to install ```MSYS2``` package manager. <br/>
Install the UCRT (universal C runtime) toolchain which is basically the modern Windows C runtime (```ucrtbase.dll```).
```cli
$ pacman -S --needed base-devel mingw-w64-ucrt-x86_64-toolchain
$ g++ --version
```

## References Used
- https://github.com/msys2/msys2-installer/releases/
- https://code.visualstudio.com/docs/cpp/config-mingw