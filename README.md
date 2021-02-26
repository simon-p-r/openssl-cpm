Simple script to build OpenSSL

Currently only builds x64/x86 both statically and dynamically on Windows

Valid options are 

    BUILD_INSTALL_PREFIX - to specify where to output OpenSSL prefix directory
    BUILD_SHARED_LIBS - to specify shared or static libraries
    WITH_APPS - to build apps target
    WITH_TEST - to enable testing


Todos

* Allow better options passing
* Use target triplets to build different configurations
* Add install target to improve install command and build better package

