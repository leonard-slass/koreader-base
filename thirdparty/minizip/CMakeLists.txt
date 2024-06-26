cmake_minimum_required(VERSION 3.5.1)
project(minizip LANGUAGES)

set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} "${CMAKE_CURRENT_LIST_DIR}/../cmake_modules")
include("koreader_thirdparty_common")
include("koreader_thirdparty_git")

assert_var_defined(CC)
assert_var_defined(AR)
assert_var_defined(LDFLAGS)
assert_var_defined(RANLIB)

ep_get_source_dir(SOURCE_DIR)

# NOTE: 53a657318af1fccc4bac7ed230729302b2391d1d is the tip of the 1.2 branch. The fcrypt API we need is gone in master.
# FIXME: Even then, something in said branch seems to upset MuPDF with our custom patch as-is, so keep using the old code...
# list(APPEND PATCH_CMD COMMAND ${KO_PATCH} ${CMAKE_CURRENT_SOURCE_DIR}/minizip-libaes-makefile.patch)

list(APPEND PATCH_CMD COMMAND ${ISED} "s|^CC=|#CC=|g" -e "s|^CFLAGS=|#CFLAGS=|g" aes/Makefile)

#fix build error due to implicit declarations of function being invalid in C99 under macOS/XCode 12 
set(CC "${CC} -Wno-error=implicit-function-declaration")
list(APPEND BUILD_CMD COMMAND ${KO_MAKE_RECURSIVE} -C aes "CC=${CC}" "LDFLAGS=${LDFLAGS}" "AR=${AR}" "RANLIB=${RANLIB}")

ko_write_gitclone_script(
    GIT_CLONE_SCRIPT_FILENAME
    https://github.com/nmoinvaz/minizip
    0b46a2b4ca317b80bc53594688883f7188ac4d08
    ${SOURCE_DIR}
)

include(ExternalProject)
ExternalProject_Add(
    ${PROJECT_NAME}
    DOWNLOAD_COMMAND ${CMAKE_COMMAND} -P ${GIT_CLONE_SCRIPT_FILENAME}
    BUILD_IN_SOURCE 1
    PATCH_COMMAND ${PATCH_CMD}
    CONFIGURE_COMMAND ""
    BUILD_COMMAND ${BUILD_CMD}
    # skip install
    INSTALL_COMMAND ""
)
