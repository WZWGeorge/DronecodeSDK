find_package(Threads REQUIRED)

include_directories(
    ${DRONECORE_ZLIB_INCLUDE_DIRS}
    ${CURL_INCLUDE_DIRS}
    SYSTEM ${PROJECT_SOURCE_DIR}/third_party/mavlink/include
)

if(IOS OR MSVC OR APPLE)
    set(LIBRARY_TYPE "STATIC")
else()
    set(LIBRARY_TYPE "SHARED")
endif()


configure_file(version.h.in version.h)

add_library(dronecode_sdk ${LIBRARY_TYPE}
    call_every_handler.cpp
    connection.cpp
    curl_wrapper.cpp
    system.cpp
    system_impl.cpp
    dronecode_sdk.cpp
    dronecode_sdk_impl.cpp
    global_include.cpp
    http_loader.cpp
    mavlink_parameters.cpp
    mavlink_commands.cpp
    mavlink_channels.cpp
    mavlink_receiver.cpp
    plugin_impl_base.cpp
    serial_connection.cpp
    tcp_connection.cpp
    timeout_handler.cpp
    udp_connection.cpp
    log.cpp
    cli_arg.cpp
    thread_pool.cpp
)

target_link_libraries(dronecode_sdk
    ${CMAKE_THREAD_LIBS_INIT}
    ${CURL_LIBRARY}
    ${TINYXML2_LIBRARY}
    ${DRONECORE_ZLIB_LIBRARIES}
)

if (IOS)
    target_link_libraries(dronecode_sdk
        "-framework Foundation"
        "-framework Security"
    )
endif()

# Link to Windows networking lib.
if (MSVC)
    target_link_libraries(dronecode_sdk
        ws2_32
    )
endif()

if(ANDROID)
    target_link_libraries(dronecode_sdk
        log
    )
endif()

set_target_properties(dronecode_sdk
    PROPERTIES COMPILE_FLAGS ${warnings}
)

target_include_directories(dronecode_sdk PRIVATE ${CMAKE_CURRENT_BINARY_DIR})

install(TARGETS dronecode_sdk
    # TODO: not sure if this is needed
    #EXPORT dronecode_sdk-targets
    DESTINATION ${lib_path}
)

install(FILES
    include/dronecode_sdk/connection_result.h
    include/dronecode_sdk/system.h
    include/dronecode_sdk/dronecode_sdk.h
    include/dronecode_sdk/plugin_base.h
    DESTINATION "${dronecode_sdk_install_include_dir}"
)

target_include_directories(dronecode_sdk
PUBLIC
    ${CMAKE_CURRENT_SOURCE_DIR}/include
    ${CMAKE_CURRENT_SOURCE_DIR}/include/dronecode_sdk
PRIVATE
    ${CMAKE_CURRENT_SOURCE_DIR}
)

list(APPEND UNIT_TEST_SOURCES
    ${PROJECT_SOURCE_DIR}/core/global_include_test.cpp
    ${PROJECT_SOURCE_DIR}/core/mavlink_channels_test.cpp
    ${PROJECT_SOURCE_DIR}/core/unittests_main.cpp
    # TODO: add this again
    #${PROJECT_SOURCE_DIR}/core/http_loader_test.cpp
    ${PROJECT_SOURCE_DIR}/core/timeout_handler_test.cpp
    ${PROJECT_SOURCE_DIR}/core/call_every_handler_test.cpp
    ${PROJECT_SOURCE_DIR}/core/curl_test.cpp
    ${PROJECT_SOURCE_DIR}/core/any_test.cpp
    ${PROJECT_SOURCE_DIR}/core/cli_arg_test.cpp
    ${PROJECT_SOURCE_DIR}/core/locked_queue_test.cpp
    ${PROJECT_SOURCE_DIR}/core/thread_pool_test.cpp
)
set(UNIT_TEST_SOURCES ${UNIT_TEST_SOURCES} PARENT_SCOPE)
