cmake_minimum_required(VERSION 3.10)
project(mtls_server C)

set(CMAKE_C_STANDARD 99)
set(CMAKE_C_STANDARD_REQUIRED ON)

find_package(wolfssl REQUIRED)
find_path(UTHASH_INCLUDE_DIRS "utarray.h")

set(CMAKE_EXE_LINKER_FLAGS "-static")

add_executable(mtls_server
    main.c
    client_mgr.h
    client_mgr.c
    stage.h
    stage.c
    message.h
    user_settings.h
    user_settings_dtls.h
    log.c
)

target_include_directories(mtls_server PRIVATE
    ${UTHASH_INCLUDE_DIRS}
)

target_compile_definitions(mtls_server PRIVATE
    LOG_USE_COLOR
)

if(WIN32)
    target_compile_definitions(mtls_server PRIVATE
        WOLFSSL_USER_SETTINGS
    )

    target_link_libraries(mtls_server
        wolfssl::wolfssl
        ws2_32)
else()
    target_link_libraries(mtls_server
        wolfssl::wolfssl
        pthread)
endif()

# set output directory
set_target_properties(mtls_server
    PROPERTIES
    RUNTIME_OUTPUT_DIRECTORY "${CMAKE_SOURCE_DIR}/target/"
    LIBRARY_OUTPUT_DIRECTORY "${CMAKE_SOURCE_DIR}/target/"
    ARCHIVE_OUTPUT_DIRECTORY "${CMAKE_SOURCE_DIR}/target/"
)

# copy assets to target directory
add_custom_command(
    TARGET mtls_server POST_BUILD
    COMMAND ${CMAKE_COMMAND} -E copy_directory
        ${CMAKE_SOURCE_DIR}/certs
        ${CMAKE_SOURCE_DIR}/target/certs
)