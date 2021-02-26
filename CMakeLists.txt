cmake_minimum_required(VERSION 3.18)

project(
    OpenSSL-CMake
    LANGUAGES C
)

set(OPENSSL_TARGET_VER "1.1.1j")
set(OPENSSL_URL "https://www.openssl.org/source/openssl-${OPENSSL_TARGET_VER}.tar.gz")
set(OPENSSL_SHA256 "aaf2fcb575cdf6491b98ab4829abf78a3dec8402b8b81efc8f23c00d443981bf")

option(WITH_APPS "Build OpenSSL with apps" OFF)
option(WITH_TEST "Build OpenSSL with testing" OFF)

include(ExternalProject)

set(CPM_DOWNLOAD_VERSION 0.31.1)

if(CPM_SOURCE_CACHE)
  set(CPM_DOWNLOAD_LOCATION "${CPM_SOURCE_CACHE}/cpm/CPM_${CPM_DOWNLOAD_VERSION}.cmake")
elseif(DEFINED ENV{CPM_SOURCE_CACHE})
  set(CPM_DOWNLOAD_LOCATION "$ENV{CPM_SOURCE_CACHE}/cpm/CPM_${CPM_DOWNLOAD_VERSION}.cmake")
else()
  set(CPM_DOWNLOAD_LOCATION "${CMAKE_BINARY_DIR}/cmake/CPM_${CPM_DOWNLOAD_VERSION}.cmake")
endif()

if(NOT (EXISTS ${CPM_DOWNLOAD_LOCATION}))
  message(STATUS "Downloading CPM.cmake to ${CPM_DOWNLOAD_LOCATION}")
  file(DOWNLOAD
       https://github.com/TheLartians/CPM.cmake/releases/download/v${CPM_DOWNLOAD_VERSION}/CPM.cmake
       ${CPM_DOWNLOAD_LOCATION}
  )
endif()

include(${CPM_DOWNLOAD_LOCATION})



find_program(MAKE_EXE NAMES gmake nmake make REQUIRED)
find_program(PERL_EXE perl REQUIRED)
find_program(NASM_EXE nasm REQUIRED)


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



# unpack openssl
CPMAddPackage(
    NAME OPENSSL_DOWNLOAD
    VERSION ${OPENSSL_TARGET_VER}
    URL ${OPENSSL_URL}
    URL_HASH SHA256=${OPENSSL_SHA256}
)


if (NOT DEFINED OPENSSL_DOWNLOAD_ADDED)
    message(FATAL_ERROR "OpenSSL is a hard dependency of libssh2")
endif()

message("OpenSSL directory ${OPENSSL_DOWNLOAD_SOURCE_DIR}")
set(OPENSSL_PREFIX ${OPENSSL_DOWNLOAD_BINARY_DIR})
if(DEFINED CMAKE_INSTALL_PREFIX)
    set(OPENSSL_PREFIX ${CMAKE_INSTALL_PREFIX})
endif()

set(CONFIGURE_CMD Configure ${OPENSSL_TARGET_ARCH} ${CONFIGURE_OPTIONS} --prefix=${OPENSSL_PREFIX} --openssldir=${OPENSSL_PREFIX})

if(WITH_TEST)
    ExternalProject_Add(
        OpenSSL
        SOURCE_DIR ${OPENSSL_DOWNLOAD_SOURCE_DIR}
        BUILD_IN_SOURCE 1
        CONFIGURE_COMMAND ${PERL_EXE} ${CONFIGURE_CMD}
        BUILD_COMMAND ${MAKE_EXE}
        INSTALL_COMMAND ${INSTALL_CMD}
        TEST_COMMAND ${MAKE_EXE} test
        TEST_BEFORE_INSTALL TRUE
    )
else()
    ExternalProject_Add(
        OpenSSL
        SOURCE_DIR ${OPENSSL_DOWNLOAD_SOURCE_DIR}
        BUILD_IN_SOURCE 1
        CONFIGURE_COMMAND ${PERL_EXE} ${CONFIGURE_CMD}
        BUILD_COMMAND ${MAKE_EXE}
        INSTALL_COMMAND ${INSTALL_CMD}
    )

endif()




