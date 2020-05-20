# lua-mac68k
...is Lua, patched so that the [Retro68 cross-compiler](https://github.com/autc04/Retro68) can compile a library for 68k Macintoshes.
To build, execute the following commands...

    mkdir build
    cd build
    cmake .. -DCMAKE_TOOLCHAIN_FILE=path/to/Retro68-build/toolchain/m68k-apple-macos/cmake/retro68.toolchain.cmake
    make
    
...where *DCMAKE_TOOLCHAIN_FILE* is set relative to your installation of Retro68.

## License.
Lua, redistributed here, is licensed under the MIT license.
The CMake files here are adapted from Retro68's samples, which is GPL-3 licensed.
