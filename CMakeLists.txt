# To use this example as a standalone project using CMake:
# mkdir build
# cd build
# cmake .. -DCMAKE_TOOLCHAIN_FILE=path/to/Retro68-build/toolchain/m68k-apple-macos/cmake/retro68.toolchain.cmake
# make

# Step 0: Patch lua/luaconf.h
find_package(Patch)
execute_process(COMMAND ${Patch_EXECUTABLE} /lua/luaconf.h ./luaconf.diff)
message("@@ Patched lua/luaconf.h")

# Step 1: Tell CMake to compile the library.
# (CMake writes the output to an XCOFF file named libLua.so)
file(GLOB LUA_SOURCES lua/*.c lua/*.h)
	# Exclude lua/onelua.c from compilation, since it's just too big
list(FILTER LUA_SOURCES EXCLUDE REGEX "onelua")
add_library(Lua SHARED ${LUA_SOURCES})
message("@@ Lua's sources are: ${LUA_SOURCES}")

if(TARGET retrocrt)
    # Hack: if we are building as part of the Retro68 source tree,
    #       make sure the run-time library is already compiled
    # (not needed for standalone projects)
    add_dependencies(Lua retrocrt)
endif(TARGET retrocrt)

# Step 2: Tell the linker to export all symbols.
target_link_options(Lua PUBLIC -lm)

# Note: Step 1 & 2 are equivalent to the command:
#  powerpc-apple-macos-gcc -shared -Wl,-bE:lua.exp library.c -o libLua.so

# Step 3: Convert the library to PEF format:

add_custom_command(
    OUTPUT Lua.pef
    COMMAND ${MAKE_PEF} "libLua.so" -o "Lua.pef"
    DEPENDS Lua)

# Step 4: Combine the PEF data fork with a resource fork containing a cfrg resource:

add_custom_command(
    OUTPUT Lua.bin Lua Lua.dsk Lua.ad "%Lua.ad"
    COMMAND ${REZ} 
            ${REZ_FLAGS}
            ${CMAKE_CURRENT_SOURCE_DIR}/Lua.r
            -I${REZ_INCLUDE_PATH}
            -o "Lua.bin" --cc "Lua.dsk" --cc "Lua"
            --cc "%Lua.ad"
            -t "shlb" -c "????"
            --data Lua.pef
            
    DEPENDS Lua.pef ${rsrc_files})

# Step 5: Create a CMake custom target so CMake knows that Steps 3 and 4 need to be done
add_custom_target(Lua_APPL ALL DEPENDS Lua.bin)

# Step 6: Create the Lua demo
# (the add_application macro works similarly to the commands above, but for applications.)

add_application(Demo demo.c)

# Step 7: Link the application to the library target (the XCOFF version of the library!)
target_link_libraries(Demo Lua)
