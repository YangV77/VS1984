cmake_minimum_required(VERSION 3.27)
project(xbc_demo)

set(LIBRSX_INCLUDE_DIR "${CMAKE_SOURCE_DIR}/sdk")
set(LIBRSX_LIB_DIR "${CMAKE_SOURCE_DIR}/sdk")
include_directories(${LIBRSX_INCLUDE_DIR})
link_directories(${LIBRSX_LIB_DIR})

add_executable(demo
        src/main.c
)

set(SOURCE_DIR "${CMAKE_SOURCE_DIR}/res")
set(DEST_DIR "${CMAKE_BINARY_DIR}/cnf")
add_custom_command(
        OUTPUT ${DEST_DIR}
        COMMAND ${CMAKE_COMMAND} -E make_directory ${DEST_DIR}
)

add_custom_command(
        OUTPUT ${DEST_DIR}/routes.json ${DEST_DIR}/config.xbc
        COMMAND ${CMAKE_COMMAND} -E copy_if_different ${SOURCE_DIR}/config.xbc ${DEST_DIR}/config.xbc
        COMMAND ${CMAKE_COMMAND} -E copy_if_different ${SOURCE_DIR}/routes.json ${DEST_DIR}/routes.json
        COMMAND ${CMAKE_COMMAND} -E copy_if_different ${SOURCE_DIR}/vs1984-btd ${CMAKE_BINARY_DIR}/vs1984-btd
        DEPENDS ${SOURCE_DIR}/config.xbc ${SOURCE_DIR}/routes.json
        COMMENT "prepare files to build directory"
)

add_custom_target(
        PREPARE_COMPILE ALL
        DEPENDS ${DEST_DIR}/routes.json ${DEST_DIR}/config.xbc
)

target_link_directories(demo PRIVATE
        ${CMAKE_SOURCE_DIR}/libs
)

set_target_properties(demo PROPERTIES
        BUILD_RPATH "$ORIGIN/../libs"
        INSTALL_RPATH "$ORIGIN/../libs;$ORIGIN"
)

target_link_libraries(demo PRIVATE xbc
        microhttpd
        pulse
        asound
        jack
        pthread dl
        m z png stdc++)

add_dependencies(demo PREPARE_COMPILE)