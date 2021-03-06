cmake_minimum_required(VERSION 2.6)
PROJECT(rbind_opencv CXX)

set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} "${CMAKE_CURRENT_SOURCE_DIR}/cmake")
include(FindPkgConfig)
pkg_check_modules(OPENCV REQUIRED opencv)
add_definitions(${OPENCV_CFLAGS})
include_directories(${OPENCV_INCLUDE_DIRS})
link_directories(${OPENCV_LIBRARY_DIRS})


SET(RBIND_SRC
    "${CMAKE_CURRENT_SOURCE_DIR}/types.cc"
    "${CMAKE_CURRENT_SOURCE_DIR}/operations.cc"
    "${CMAKE_CURRENT_SOURCE_DIR}/conversions.cc")

add_custom_command(OUTPUT ${RBIND_SRC}
       COMMAND ruby "${CMAKE_CURRENT_SOURCE_DIR}/../rbind.rb")

ADD_LIBRARY(rbind_opencv SHARED ${RBIND_SRC})
TARGET_LINK_LIBRARIES(rbind_opencv ${OPENCV_LIBS} ${OPENCV_LDFLAGS_OTHER} )

set(ROOT_FOLDER ${CMAKE_CURRENT_SOURCE_DIR}/../..)
install(TARGETS rbind_opencv LIBRARY DESTINATION ${ROOT_FOLDER}/lib/ropencv)

# global install
if(NOT GEM_INSTALL)
    FIND_PACKAGE(Ruby)
    IF(NOT RUBY_INCLUDE_PATH)
        MESSAGE(STATUS "Ruby library not found. Cannot install ruby extensions")
    ELSE(NOT RUBY_INCLUDE_PATH)
        STRING(REGEX REPLACE ".*lib(32|64)?/?" "lib/" RUBY_LIBRARY_INSTALL_DIR ${RUBY_RUBY_LIB_DIR})
        INSTALL(DIRECTORY ${ROOT_FOLDER}/lib/ DESTINATION ${RUBY_LIBRARY_INSTALL_DIR}
            FILES_MATCHING PATTERN "*.rb" PATTERN "*.rbind" PATTERN "*.so" PATTERN "*.dylib")
    ENDIF(NOT RUBY_INCLUDE_PATH)
ENDIF(NOT GEM_INSTALL)
