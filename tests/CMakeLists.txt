project(pfs_common_TESTS CXX C)
option(ENABLE_COVERAGE "Build tests with coverage support" OFF)

find_package(Threads REQUIRED)

if (ENABLE_COVERAGE)
    message(STATUS "Coverage feature enabled")
    if (NOT "${CMAKE_CXX_COMPILER_ID}" STREQUAL "GNU")
        message(FATAL_ERROR "Coverage feature requires GCC")
    endif()

    set(CMAKE_CXX_FLAGS "-g -O0 --coverage")
    set(CMAKE_C_FLAGS "-g -O0 --coverage")
endif()

# Copy test files to build folder
file(COPY ${CMAKE_CURRENT_SOURCE_DIR}/unicode/utf8/data DESTINATION unicode/utf8)

set(TEST_NAMES
    any
    calendar
    compare_op
    dynamic_library
    filesystem
    iterator_bidirectional
    iterator_forward
    iterator_input
    iterator_output
    iterator_random_access
    optional
    pointer_proxy_iterator
    rational
    ring_buffer
    ring_buffer_mt
    safeformat
    string_view
    variant
    utf8_decode
    utf8_encode
    utf16le_decode
#     utf16le_encode
    utf16be_decode
#     utf16be_encode
    )

set(utf8_resource_SOURCES
    unicode/utf8/cyrillic.c
    unicode/utf8/gothic.c
    unicode/utf8/greek.c
    unicode/utf8/mideng.c
    unicode/utf8/midger.c
    unicode/utf8/multilang.c
    unicode/utf8/rune.c
    unicode/utf8/vietnamese.c)

set(utf16le_resource_SOURCES
    unicode/utf16le/cyrillic.c
    unicode/utf16le/gothic.c
    unicode/utf16le/greek.c
    unicode/utf16le/mideng.c
    unicode/utf16le/midger.c
    unicode/utf16le/multilang.c
    unicode/utf16le/rune.c
    unicode/utf16le/vietnamese.c)

set(utf16be_resource_SOURCES
    unicode/utf16be/cyrillic.c
    unicode/utf16be/gothic.c
    unicode/utf16be/greek.c
    unicode/utf16be/mideng.c
    unicode/utf16be/midger.c
    unicode/utf16be/multilang.c
    unicode/utf16be/rune.c
    unicode/utf16be/vietnamese.c)

set(utf8_decode_SOURCES ${utf8_resource_SOURCES})
set(utf8_encode_SOURCES ${utf8_resource_SOURCES})

set(utf16le_decode_SOURCES ${utf16le_resource_SOURCES})
set(utf16be_decode_SOURCES ${utf16be_resource_SOURCES})

foreach (name ${TEST_NAMES})
    if (${name}_SOURCES)
        add_executable(${name} ${${name}_SOURCES} ${name}.cpp)
    else ()
        add_executable(${name} ${name}.cpp)
    endif()

    target_link_libraries(${name} PRIVATE pfs::common)
    add_test(NAME ${name} COMMAND ${name})
endforeach()

target_link_libraries(ring_buffer_mt PRIVATE Threads::Threads)

if (UNIX)
    target_link_libraries(dynamic_library PRIVATE dl)
endif()

# Helper library for test `dynamic_library`
add_library(shared_object SHARED shared_object.c)

# Copy test data files
add_custom_command(TARGET utf8_decode
    POST_BUILD
    COMMAND ${CMAKE_COMMAND} -E make_directory "$<TARGET_FILE_DIR:utf8_decode>/unicode"
    COMMAND ${CMAKE_COMMAND} -E make_directory "$<TARGET_FILE_DIR:utf8_decode>/unicode/utf8"
    COMMAND ${CMAKE_COMMAND} -E make_directory "$<TARGET_FILE_DIR:utf8_decode>/unicode/utf8/data"
    COMMAND ${CMAKE_COMMAND} -E copy_directory
        "${CMAKE_CURRENT_SOURCE_DIR}/unicode/utf8/data"
        "$<TARGET_FILE_DIR:utf8_decode>/unicode/utf8/data"
    VERBATIM)

add_custom_command(TARGET utf16le_decode
    POST_BUILD
    COMMAND ${CMAKE_COMMAND} -E make_directory "$<TARGET_FILE_DIR:utf16le_decode>/unicode"
    COMMAND ${CMAKE_COMMAND} -E make_directory "$<TARGET_FILE_DIR:utf16le_decode>/unicode/utf16le"
    COMMAND ${CMAKE_COMMAND} -E make_directory "$<TARGET_FILE_DIR:utf16le_decode>/unicode/utf16le/data"
    COMMAND ${CMAKE_COMMAND} -E copy_directory
        "${CMAKE_CURRENT_SOURCE_DIR}/unicode/utf16le/data"
        "$<TARGET_FILE_DIR:utf16le_decode>/unicode/utf16le/data"
    VERBATIM)

add_custom_command(TARGET utf16be_decode
    POST_BUILD
    COMMAND ${CMAKE_COMMAND} -E make_directory "$<TARGET_FILE_DIR:utf16be_decode>/unicode"
    COMMAND ${CMAKE_COMMAND} -E make_directory "$<TARGET_FILE_DIR:utf16be_decode>/unicode/utf16be"
    COMMAND ${CMAKE_COMMAND} -E make_directory "$<TARGET_FILE_DIR:utf16be_decode>/unicode/utf16be/data"
    COMMAND ${CMAKE_COMMAND} -E copy_directory
        "${CMAKE_CURRENT_SOURCE_DIR}/unicode/utf16be/data"
        "$<TARGET_FILE_DIR:utf16be_decode>/unicode/utf16be/data"
    VERBATIM)
