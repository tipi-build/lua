cmake_minimum_required(VERSION 3.0)

project(lua VERSION 5.3.2)


### Library ###

add_library(lua_lib
    src/lapi.c
    src/lcode.c
    src/lctype.c
    src/ldebug.c
    src/ldo.c
    src/ldump.c
    src/lfunc.c
    src/lgc.c
    src/llex.c
    src/lmem.c
    src/lobject.c
    src/lopcodes.c
    src/lparser.c
    src/lstate.c
    src/lstring.c
    src/ltable.c
    src/ltm.c
    src/lundump.c
    src/lvm.c
    src/lzio.c
    src/lauxlib.c
    src/lbaselib.c
    src/lbitlib.c
    src/lcorolib.c
    src/ldblib.c
    src/liolib.c
    src/lmathlib.c
    src/loslib.c
    src/lstrlib.c
    src/ltablib.c
    src/lutf8lib.c
    src/loadlib.c
    src/linit.c

    src/lopcodes.h
    src/lparser.h
    src/lprefix.h
    src/lzio.h
    src/lstate.h
    src/lmem.h
    src/lstring.h
    src/llimits.h
    src/llex.h
    src/ltable.h
    src/lgc.h
    src/lfunc.h
    src/ltm.h
    src/ldo.h
    src/lua.h
    src/lapi.h
    src/ldebug.h
    src/luaconf.h
    src/lualib.h
    src/lctype.h
    src/lundump.h
    src/lcode.h
    src/lauxlib.h
    src/lvm.h
    src/lobject.h
    src/lua.hpp
)

set_property(TARGET lua_lib PROPERTY VERSION ${lua_VERSION})

target_include_directories(lua_lib
    PRIVATE src
)

include(CheckLibraryExists) # check_library_exists


# Some platforms (e.g. Linux) need separate math library
check_library_exists(m pow "" LIB_M_REQUIRED)
if(LIB_M_REQUIRED)
  target_link_libraries(lua_lib PRIVATE m)
endif()


### Interpreter ###

add_executable(lua src/lua.c)

target_link_libraries(lua lua_lib)


### Compiler ###

add_executable(luac src/luac.c)

target_link_libraries(luac lua_lib)


### Installation ###

install(TARGETS lua_lib lua luac
    EXPORT LuaConfig
    LIBRARY DESTINATION bin
    RUNTIME DESTINATION bin
    ARCHIVE DESTINATION lib
    INCLUDES DESTINATION include
)

install(
    FILES
    src/lua.h
    src/lua.hpp
    src/lualib.h
    src/lauxlib.h
    src/luaconf.h

    DESTINATION
    include
)

include(CMakePackageConfigHelpers)

write_basic_package_version_file(
    "${CMAKE_CURRENT_BINARY_DIR}/Lua/LuaConfigVersion.cmake"
    COMPATIBILITY AnyNewerVersion
)

install(
    FILES
    "${CMAKE_CURRENT_BINARY_DIR}/Lua/LuaConfigVersion.cmake"

    DESTINATION
    cmake
)


install(
    EXPORT LuaConfig
    DESTINATION cmake
    NAMESPACE Lua::
)
