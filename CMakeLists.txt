cmake_minimum_required(VERSION 3.18)

project(
    OpenSSL-CMake
    LANGUAGES C
)

set(CPM_DOWNLOAD_VERSION 0.28.4)

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



set(OPENSSL_TARGET_VER "1.1.1j")
set(OPENSSL_URL "https://www.openssl.org/source/openssl-${OPENSSL_TARGET_VER}.tar.gz")
set(OPENSSL_SHA256 "aaf2fcb575cdf6491b98ab4829abf78a3dec8402b8b81efc8f23c00d443981bf")


find_program(MAKE_EXE NAMES gmake nmake make REQUIRED)
find_program(PERL_EXE perl REQUIRED)
find_program(NASM_EXE nasm REQUIRED)



# enable-capieng
if(CMAKE_SIZEOF_VOID_P EQUAL 8)
    set(TARGET_BUILD_ARCH "x64")
    if(MSVC)
        set(OPENSSL_CONFIG "VC-WIN64")
    endif()
elseif(CMAKE_SIZEOF_VOID_P EQUAL 4)
    set(TARGET_BUILD_ARCH "x86")
    if(MSVC)
        set(OPENSSL_CONFIG "VC-WIN32")
    endif()
endif()

if(CMAKE_BUILD_TYPE STREQUAL "Debug")
    if(MSVC)
        set(OPENSSL_CONFIG "debug-${OPENSSL_CONFIG}")
    endif()
endif()

if(BUILD_SHARED_LIBS)
    set(OPENSSL_SHARED_LIB shared)
else()
    set(OPENSSL_SHARED_LIB no-shared)
endif()

if(NOT OPENSSL_CONFIGURE_OPTS)
    set(OPENSSL_CONFIGURE_OPTS enable-capieng)
endif()


# message("MAKE_EXE ${MAKE_EXE}")
# message("NASM_EXE ${NASM_EXE}")
# message("PERL_EXE ${PERL_EXE}")
# message("OPENSSL_URL ${OPENSSL_URL}")
# message("CMAKE_INSTALL_PREFIX ${CMAKE_INSTALL_PREFIX}")
# message("OPENSSL_BUILD_DIR ${OPENSSL_BUILD_DIR}")
# message("OPENSSL_CONFIGURE ${OPENSSL_CONFIGURE}")


# unpack openssl
CPMAddPackage(
    NAME OpenSSL_Download
    VERSION 1.1.1j
    URL ${OPENSSL_URL}
    URL_HASH SHA256=${OPENSSL_SHA256}
)

if (OpenSSL_Download_ADDED)
    message("OpenSSL directory ${OpenSSL_Download_SOURCE_DIR}")
    set(OPENSSL_ROOT_DIR ${OpenSSL_Download_BINARY_DIR})
    set(OPENSSL_CONFIGURE_CMD 
        ${OPENSSL_CONFIG} ${OPENSSL_SHARED_LIB} ${OPENSSL_CONFIGURE_OPTS} --prefix=${OPENSSL_ROOT_DIR} --openssldir=${OPENSSL_ROOT_DIR}
    )
    include(ExternalProject)
    ExternalProject_Add(
        OpenSSL_Build
        SOURCE_DIR ${OpenSSL_Download_SOURCE_DIR}
        BUILD_IN_SOURCE 1
        CONFIGURE_COMMAND ${PERL_EXE} Configure ${OPENSSL_CONFIGURE_CMD}
        BUILD_COMMAND ${MAKE_EXE}
        INSTALL_COMMAND ${MAKE_EXE} install
        TEST_COMMAND ${MAKE_EXE} test
        TEST_BEFORE_INSTALL TRUE
    )

else()
    message(FATAL_ERROR "OpenSSL is a hard dependency of libssh2")
endif()


