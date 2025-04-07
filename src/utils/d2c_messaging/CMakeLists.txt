cmake_minimum_required (VERSION 3.5)

set (target_name d2c_messaging)
add_library (${target_name} STATIC src/d2c_messaging.c)
add_library (aduc::${target_name} ALIAS ${target_name})

target_include_directories (${target_name} PUBLIC ./inc ${ADUC_TYPES_INCLUDES}
                                                  ${ADUC_EXPORT_INCLUDES})

#
# Turn -fPIC on, in order to use this library in another shared library.
#
set_property (TARGET ${target_name} PROPERTY POSITION_INDEPENDENT_CODE ON)

find_package (OpenSSL REQUIRED)
target_link_libraries (${target_name} PRIVATE  OpenSSL::Crypto)

target_link_aziotsharedutil (${target_name} PRIVATE)

# Check if the architecture is AMD64
if(CMAKE_SYSTEM_PROCESSOR STREQUAL "x86_64")
    # Check if libssl.so.1.1 exists in both directories
    if(EXISTS "/usr/lib/x86_64-linux-gnu/libssl.so.1.1" AND EXISTS "/usr/local/lib/deviceupdate-openssl/lib/libssl.so.1.1")
        # Set the CMAKE_SKIP_RPATH variable to TRUE
        set(CMAKE_SKIP_RPATH TRUE)

        # Specify the directories where CMake should look for libraries
        link_directories(/usr/lib/x86_64-linux-gnu)

        # Set the LINK_DIRECTORIES property to exclude /usr/local/lib/deviceupdate-openssl/lib
        set_target_properties(AducIotAgent PROPERTIES LINK_DIRECTORIES "/usr/lib/x86_64-linux-gnu")
    endif()
endif()

target_link_libraries (
    ${target_name}
    PUBLIC aduc::adu_types
    PRIVATE aduc::communication_abstraction aduc::logging aduc::retry_utils)

if (ADUC_BUILD_UNIT_TESTS)
    if (${IS_ARM})
        message (STATUS "Skipping tests for utils/d2c_messaging for ARM processor...")
    else ()
        add_subdirectory (tests)
    endif ()
endif ()
