cmake_minimum_required(VERSION 3.7)
project(libiconv LANGUAGES)

set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} "${CMAKE_CURRENT_LIST_DIR}/../cmake_modules")
include("koreader_thirdparty_common")

assert_var_defined(HOST)

ep_get_source_dir(SOURCE_DIR)
ep_get_binary_dir(BINARY_DIR)

set(CFG_ENV_VAR "CC=${CC}" "LD=${LD}" "AR=${AR}" "RANLIB=${RANLIB}")
set(CFG_OPTS --with-threads=none --prefix=${BINARY_DIR} --libdir=${BINARY_DIR}/lib --enable-shared=false --enable-static=true --host=${HOST})
set(CFG_CMD env ${CFG_ENV_VAR} ${SOURCE_DIR}/configure ${CFG_OPTS})

list(APPEND BUILD_CMD COMMAND ${KO_MAKE_RECURSIVE})

list(APPEND INSTALL_CMD COMMAND ${KO_MAKE_RECURSIVE} install)

set(GETTEXT_VER "1.15")
include(ExternalProject)
ExternalProject_Add(
    libiconv
    URL http://ftpmirror.gnu.org/libiconv/libiconv-${GETTEXT_VER}.tar.gz
    http://ftp.gnu.org/pub/gnu/libiconv/libiconv-${GETTEXT_VER}.tar.gz
    URL_MD5 ace8b5f2db42f7b3b3057585e80d9808
    DOWNLOAD_DIR ${KO_DOWNLOAD_DIR}
    CONFIGURE_COMMAND ${CFG_CMD}
    BUILD_COMMAND ${BUILD_CMD}
    INSTALL_COMMAND ${INSTALL_CMD}
)
