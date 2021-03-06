cmake_minimum_required(VERSION 3.9)

project(libmodbus C)

set(src ${CMAKE_CURRENT_SOURCE_DIR}/src)

# Configure files containing the version number (Copied from configure.ac)
find_package(GIT REQUIRED)
if(GIT_FOUND)
    execute_process(
        COMMAND ${GIT_EXECUTABLE} rev-list --count HEAD
        WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}
        OUTPUT_VARIABLE LIBMODBUS_VERSION_BUILD
        ERROR_QUIET
        OUTPUT_STRIP_TRAILING_WHITESPACE)
else()
    set(LIBMODBUS_VERSION_BUILD 0)
endif()

set(LIBMODBUS_VERSION_MAJOR 3)
set(LIBMODBUS_VERSION_MINOR 1)
set(LIBMODBUS_VERSION_MICRO 4)
set(LIBMODBUS_VERSION ${LIBMODBUS_VERSION_MAJOR}.${LIBMODBUS_VERSION_MINOR}.${LIBMODBUS_VERSION_MICRO})

configure_file(${src}/modbus-version.h.in ${CMAKE_BINARY_DIR}/modbus-version.h @ONLY)
if(WIN32)
    configure_file(${src}/win32/config.h.win32 ${CMAKE_BINARY_DIR}/config.h @ONLY)
endif()

set(vcpkg ${CMAKE_CURRENT_SOURCE_DIR}/vcpkg)
configure_file(${vcpkg}/CONTROL.in ${CMAKE_BINARY_DIR}/ports/libmodbus/CONTROL @ONLY)
configure_file(${vcpkg}/port.cmake ${CMAKE_BINARY_DIR}/ports/libmodbus/portfile.cmake COPYONLY)
configure_file(${vcpkg}/pack.bat.in ${CMAKE_BINARY_DIR}/pack.bat @ONLY)


# Define sources and (public/private) headers for the packaging
set(LIBMODBUS_SOURCES
    ${src}/modbus.c
    ${src}/modbus-data.c
    ${src}/modbus-rtu.c
    ${src}/modbus-tcp.c
)

set(LIBMODBUS_PUBLIC_HEADERS
    ${src}/modbus.h
    ${src}/modbus-rtu.h
    ${src}/modbus-tcp.h
    ${CMAKE_BINARY_DIR}/modbus-version.h
)

set(LIBMODBUS_PRIVATE_HEADERS
    ${src}/modbus-tcp-private.h
    ${src}/modbus-tcp-private.h
    ${CMAKE_BINARY_DIR}/config.h
)

# Create shared library
add_library(${PROJECT_NAME} SHARED ${LIBMODBUS_SOURCES} ${LIBMODBUS_PUBLIC_HEADERS} ${LIBMODBUS_PRIVATE_HEADERS})

set_target_properties(${PROJECT_NAME} PROPERTIES PREFIX "" IMPORT_PREFIX "")

target_include_directories(${PROJECT_NAME} PUBLIC
    $<BUILD_INTERFACE:${src}> $<BUILD_INTERFACE:${CMAKE_BINARY_DIR}>
    $<INSTALL_INTERFACE:include>)

target_compile_definitions(${PROJECT_NAME} PRIVATE $<$<OR:$<C_COMPILER_ID:MSVC>,$<AND:$<C_COMPILER_ID:Clang>,$<PLATFORM_ID:Windows>>>:DLLBUILD;_CRT_SECURE_NO_WARNINGS;_WINSOCK_DEPRECATED_NO_WARNINGS>)
target_link_libraries(${PROJECT_NAME} PRIVATE $<$<PLATFORM_ID:Windows>:ws2_32>)

set_target_properties(${PROJECT_NAME} PROPERTIES PUBLIC_HEADER "${LIBMODBUS_PUBLIC_HEADERS}")

# Install package
install(TARGETS ${PROJECT_NAME} EXPORT ${PROJECT_NAME}_export
        RUNTIME DESTINATION bin
        ARCHIVE DESTINATION lib
        PUBLIC_HEADER DESTINATION include)
install(FILES ${CMAKE_SOURCE_DIR}/libmodbus-config.cmake DESTINATION lib/cmake/${PROJECT_NAME})
install(FILES ${LIBMODBUS_SOURCES} DESTINATION src)

install(EXPORT ${PROJECT_NAME}_export NAMESPACE Libmodbus:: DESTINATION lib/cmake/${PROJECT_NAME})

# Export from build tree
configure_file(${CMAKE_SOURCE_DIR}/libmodbus-config.cmake ${CMAKE_BINARY_DIR}/libmodbus-config.cmake COPYONLY)
export(EXPORT ${PROJECT_NAME}_export NAMESPACE Libmodbus::)
