find_package(nanomsg CONFIG REQUIRED)

IF(!nanomsg_FOUND)
    IF (WIN32)
    # This is not advanced yet, as it does not work for custom install locations.
    # The general directory, however, is found by `find_package`.
    # --> Throw an error, if nanomsg cannot be found
        message(FATAL_ERROR "nanomsg library is required, but not found in standard location.")
    ELSE()
        # on UNIX platforms pkg_config can be used as fallback solution
        find_package(PkgConfig REQUIRED)
        pkg_check_modules(nanomsg REQUIRED nanomsg)
    ENDIF()
ENDIF()


set(NANOMSG_SRCS ./nanomsg/ext/nnxx_ext.c)

set(NNXX_SRCS   ./nnxx/error.cpp
                ./nnxx/message.cpp
                ./nnxx/message_control.cpp
                ./nnxx/message_istream.cpp
                ./nnxx/message_iterator.cpp
                ./nnxx/message_ostream.cpp
                ./nnxx/message_streambuf.cpp
                ./nnxx/nn.cpp
                ./nnxx/poll.cpp
                ./nnxx/pubsub.cpp
                ./nnxx/reqrep.cpp
                ./nnxx/socket.cpp
                ./nnxx/survey.cpp
                ./nnxx/tcp.cpp
                ./nnxx/timeout.cpp)

add_library(${PROJECT_NAME} ${NANOMSG_SRCS} ${NNXX_SRCS})
add_library(nnxx::nnxx ALIAS ${PROJECT_NAME})
target_include_directories(${PROJECT_NAME}
    PUBLIC
        $<BUILD_INTERFACE:${CMAKE_CURRENT_LIST_DIR}>
        $<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}/nnxx>
        ${NANOMSG_INCLUDE_DIRS})
target_link_libraries(${PROJECT_NAME} PUBLIC nanomsg)

########################
# installation
########################
# set up libbasecamp_service install
set(INSTALL_CONFIGDIR ${CMAKE_INSTALL_LIBDIR}/cmake/Nnxx)

install(
    TARGETS
        ${PROJECT_NAME}
    EXPORT NnxxTargets
    LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
    ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
    RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
)
install(
    DIRECTORY nanomsg/ext
    DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/nnxx
    PATTERN "*.c*" EXCLUDE
)
install(
    DIRECTORY nnxx
    DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/nnxx
    PATTERN "*.c*" EXCLUDE
)
install(EXPORT NnxxTargets
    FILE NnxxTargets.cmake
    NAMESPACE nnxx::
    DESTINATION ${INSTALL_CONFIGDIR}
)

########################
# ConfigVersion file
########################
include(CMakePackageConfigHelpers)
write_basic_package_version_file(
    ${CMAKE_CURRENT_BINARY_DIR}/NnxxConfigVersion.cmake
    VERSION ${PROJECT_VERSION}
    COMPATIBILITY AnyNewerVersion
)

configure_package_config_file(
    ${CMAKE_SOURCE_DIR}/CMake/NnxxConfig.cmake.in
    ${CMAKE_CURRENT_BINARY_DIR}/NnxxConfig.cmake
    INSTALL_DESTINATION ${INSTALL_CONFIGDIR}
)

# Install all the helper files
install(
    FILES
      ${CMAKE_CURRENT_BINARY_DIR}/NnxxConfig.cmake
      ${CMAKE_CURRENT_BINARY_DIR}/NnxxConfigVersion.cmake
    DESTINATION ${INSTALL_CONFIGDIR}
)