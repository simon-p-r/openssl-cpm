cmake_minimum_required(VERSION 3.18)

project(
    OpenSSL-CMake
    LANGUAGES C
)


include(ExternalProject)
include(BuildOpenSSL)

unset(OPENSSL_INCLUDE_DIR CACHE)
unset(OPENSSL_CRYPTO_LIBRARY CACHE)
unset(OPENSSL_SSL_LIBRARY CACHE)



set(OPENSSL_PREFIX ${CMAKE_CURRENT_BINARY_DIR})
if(CMAKE_INSTALL_PREFIX)
    set(OPENSSL_PREFIX ${CMAKE_INSTALL_PREFIX})
endif()

set(OPENSSL_TARGET_VER "1.1.1j")
set(OPENSSL_URL "https://www.openssl.org/source/openssl-${OPENSSL_TARGET_VER}.tar.gz")
set(OPENSSL_SHA256 "aaf2fcb575cdf6491b98ab4829abf78a3dec8402b8b81efc8f23c00d443981bf")

option(WITH_APPS "Build OpenSSL with apps" OFF)
option(WITH_TEST "Build OpenSSL with testing" OFF)


find_program(MAKE_EXE NAMES gmake nmake make REQUIRED)
find_program(PERL_EXE perl REQUIRED)

if(WIN32)
    find_program(NASM_EXE nasm REQUIRED)
endif()


set(OPENSSL_SHARED no-shared)
if(BUILD_SHARED_LIBS)
    set(OPENSSL_SHARED shared)
endif()

if(MSVC)
    set(OPENSSL_TARGET_ARCH "VC-WIN32")
    if(CMAKE_BUILD_TYPE STREQUAL "Debug")
        set(OPENSSL_TARGET_ARCH "debug-VC-WIN32")
    endif()
    if(CMAKE_SIZEOF_VOID_P EQUAL 8)
        set(OPENSSL_TARGET_ARCH "VC-WIN64A")
        if(CMAKE_BUILD_TYPE STREQUAL "Debug")
            set(OPENSSL_TARGET_ARCH "debug-VC-WIN64A")
        endif()
    endif()
    set(CONFIGURE_OPTIONS 
        enable-static-engine
        enable-capieng
        no-ssl2
        no-tests
        -utf-8
        ${OPENSSL_SHARED}
    )
endif()


set(INSTALL_CMD ${MAKE_EXE} install_sw)
if(WITH_APPS)
    set(INSTALL_CMD ${MAKE_EXE} install)
endif()


set(CONFIGURE_CMD Configure ${OPENSSL_TARGET_ARCH} ${CONFIGURE_OPTIONS})

if(WITH_TEST)
    ExternalProject_Add(
        OpenSSL
        URL ${OPENSSL_URL}
        URL_HASH SHA256=${OPENSSL_SHA256}
        CONFIGURE_COMMAND ${PERL_EXE} ${CONFIGURE_CMD} --prefix=<INSTALL_DIR> --openssldir=<INSTALL_DIR>
        BUILD_COMMAND ${MAKE_EXE}
        BUILD_IN_SOURCE 1
        INSTALL_COMMAND ${INSTALL_CMD}
        TEST_COMMAND ${MAKE_EXE} test
        TEST_BEFORE_INSTALL 1
    )
else()
    ExternalProject_Add(
        OpenSSL
        URL ${OPENSSL_URL}
        URL_HASH SHA256=${OPENSSL_SHA256}
        CONFIGURE_COMMAND ${PERL_EXE} ${CONFIGURE_CMD} --prefix=<INSTALL_DIR> --openssldir=<INSTALL_DIR>
        BUILD_COMMAND ${MAKE_EXE}
        BUILD_IN_SOURCE 1
        INSTALL_COMMAND ${INSTALL_CMD}
    )
endif()


ExternalProject_Get_Property(OpenSSL INSTALL_DIR)
find_path(OPENSSL_INCLUDE_DIR
    NAMES
        openssl/ssl.h
    PATHS
        "${INSTALL_DIR}"
    PATH_SUFFIXES
        include
    NO_DEFAULT_PATH
)
find_library(OPENSSL_CRYPTO_LIBRARY
    NAMES
        crypto
        libcrypto
    PATHS
        "${INSTALL_DIR}"
    PATH_SUFFIXES
        lib
    NO_DEFAULT_PATH
)
find_library(OPENSSL_SSL_LIBRARY
    NAMES
        ssl
        libssl
    PATHS
        "${INSTALL_DIR}"
    PATH_SUFFIXES
        lib
    NO_DEFAULT_PATH
)

mark_as_advanced(OPENSSL_INCLUDE_DIR OPENSSL_CRYPTO_LIBRARY OPENSSL_SSL_LIBRARY)

add_library(OpenSSL::Crypto UNKNOWN IMPORTED)
set_target_properties(OpenSSL::Crypto PROPERTIES
    INTERFACE_INCLUDE_DIRECTORIES "${OPENSSL_INCLUDE_DIR}"
    IMPORTED_LINK_INTERFACE_LANGUAGES "C"
    IMPORTED_LOCATION "${OPENSSL_CRYPTO_LIBRARY}"
)

add_library(OpenSSL::SSL UNKNOWN IMPORTED)
set_target_properties(OpenSSL::SSL PROPERTIES
    INTERFACE_INCLUDE_DIRECTORIES "${OPENSSL_INCLUDE_DIR}"
    IMPORTED_LINK_INTERFACE_LANGUAGES "C"
    IMPORTED_LOCATION "${OPENSSL_SSL_LIBRARY}"
    INTERFACE_LINK_LIBRARIES OpenSSL::Crypto
)

add_dependencies(OpenSSL::Crypto openssl)
add_dependencies(OpenSSL::SSL openssl)

set(OPENSSL_ROOT_DIR ${INSTALL_DIR} PARENT_SCOPE)


