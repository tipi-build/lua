cmake_minimum_required(VERSION 3.0)

project(Lua VERSION 5.1.5)

### Library ###

add_library(lua_lib
    src/lapi.c
    src/lapi.h
    src/lauxlib.c
    src/lauxlib.h
    src/lbaselib.c
    src/lcode.c
    src/lcode.h
    src/ldblib.c
    src/ldebug.c
    src/ldebug.h
    src/ldo.c
    src/ldo.h
    src/ldump.c
    src/lfunc.c
    src/lfunc.h
    src/lgc.c
    src/lgc.h
    src/linit.c
    src/liolib.c
    src/llex.c
    src/llex.h
    src/llimits.h
    src/lmathlib.c
    src/lmem.c
    src/lmem.h
    src/loadlib.c
    src/lobject.c
    src/lobject.h
    src/lopcodes.c
    src/lopcodes.h
    src/loslib.c
    src/lparser.c
    src/lparser.h
    src/lstate.c
    src/lstate.h
    src/lstring.c
    src/lstring.h
    src/lstrlib.c
    src/ltable.c
    src/ltable.h
    src/ltablib.c
    src/ltm.c
    src/ltm.h
    src/lua.h
    src/luaconf.h
    src/lualib.h
    src/lundump.c
    src/lundump.h
    src/lvm.c
    src/lvm.h
    src/lzio.c
    src/lzio.h
    src/print.c
)

set_property(TARGET lua_lib PROPERTY VERSION ${Lua_VERSION})

target_include_directories(lua_lib
    PUBLIC
    "$<BUILD_INTERFACE:${CMAKE_CURRENT_LIST_DIR}/src>"
)

include(CheckLibraryExists) # check_library_exists


# Some platforms (e.g. Linux) need separate math library
check_library_exists(m pow "" LIB_M_REQUIRED)
if(LIB_M_REQUIRED)
  target_link_libraries(lua_lib PRIVATE m)
endif()

### Installation (https://github.com/forexample/package-example) ###

set(config_install_dir "lib/cmake/${PROJECT_NAME}")

set(generated_dir "${CMAKE_CURRENT_BINARY_DIR}/generated")

set(version_config "${generated_dir}/${PROJECT_NAME}ConfigVersion.cmake")
set(project_config "${generated_dir}/${PROJECT_NAME}Config.cmake")
set(targets_export_name "${PROJECT_NAME}Targets")
set(namespace "${PROJECT_NAME}::")

install(TARGETS lua_lib
    EXPORT "${targets_export_name}"
    LIBRARY DESTINATION lib
    RUNTIME DESTINATION bin
    ARCHIVE DESTINATION lib
    INCLUDES DESTINATION include
)

install(
    FILES
    src/lua.h
    src/lualib.h
    src/lauxlib.h
    src/luaconf.h

    DESTINATION
    include
)

include(CMakePackageConfigHelpers)

write_basic_package_version_file(
    "${version_config}" COMPATIBILITY AnyNewerVersion
)

# Use 'targets_export_name'
configure_package_config_file(
    "cmake/Config.cmake.in"
    "${project_config}"
    INSTALL_DESTINATION "${config_install_dir}"
)

install(
    FILES "${project_config}" "${version_config}"
    DESTINATION "${config_install_dir}"
)

install(
    EXPORT "${targets_export_name}"
    NAMESPACE "${namespace}"
    DESTINATION "${config_install_dir}"
)

if(NOT CMAKE_CROSSCOMPILING)
  ### Interpreter ###
  add_executable(lua src/lua.c)
  target_link_libraries(lua lua_lib)

  ### Compiler ###
  add_executable(luac src/luac.c)
  target_link_libraries(luac lua_lib)

  string(COMPARE EQUAL "${CMAKE_SYSTEM_NAME}" "Linux" is_linux)
  if(is_linux)
    set_target_properties(
        lua luac
        PROPERTIES
        INSTALL_RPATH "\$ORIGIN/../lib"
    )
  elseif(APPLE)
    set_target_properties(
        lua luac
        PROPERTIES
        INSTALL_RPATH "@executable_path/../lib"
    )
  endif()

  install(TARGETS lua luac
      EXPORT "${targets_export_name}"
      LIBRARY DESTINATION lib
      RUNTIME DESTINATION bin
      ARCHIVE DESTINATION lib
      INCLUDES DESTINATION include
  )
endif()
