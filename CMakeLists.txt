cmake_minimum_required(VERSION 3.9)
include(GNUInstallDirs)

project(pplx VERSION 1.0.0 DESCRIPTION "pplx")

if(${CMAKE_SYSTEM_NAME} MATCHES "Linux")

    set(BOOST_INCLUDE_LIBRARIES system thread date_time)
    set(BOOST_ENABLE_CMAKE ON)

    include(FetchContent)
    FetchContent_Declare(
      Boost
      URL https://github.com/boostorg/boost/releases/download/boost-1.81.0/boost-1.81.0.7z
      USES_TERMINAL_DOWNLOAD TRUE
      DOWNLOAD_NO_EXTRACT FALSE
    )
endif(${CMAKE_SYSTEM_NAME} MATCHES "Linux")

option(${PROJECT_NAME}_as_shared "build as a shared library" ON)
set(BUILD_SHARED_LIBS ${${PROJECT_NAME}_as_shared})
set(CMAKE_CXX_FLAGS ${CMAKE_CXX_FLAGS} -std=c++11)

add_library(${PROJECT_NAME} "")
set_target_properties(${PROJECT_NAME} PROPERTIES VERSION ${PROJECT_VERSION} POSITION_INDEPENDENT_CODE ON)

if(${CMAKE_SYSTEM_NAME} MATCHES "Linux")
    FetchContent_MakeAvailable(Boost)
    file(GLOB Boost_INCLUDE_DIRS LIST_DIRECTORIES true ${Boost_SOURCE_DIR}/libs/*/include)
    target_link_libraries(${PROJECT_NAME} PRIVATE Boost::thread Boost::date_time)
    target_include_directories(${PROJECT_NAME}
        PUBLIC ${Boost_INCLUDE_DIRS}
    )
endif(${CMAKE_SYSTEM_NAME} MATCHES "Linux")

target_include_directories(${PROJECT_NAME} PRIVATE include)
target_sources(${PROJECT_NAME}
    PUBLIC
        "${CMAKE_CURRENT_SOURCE_DIR}/include/pplx/cpprest_compat.h"
        "${CMAKE_CURRENT_SOURCE_DIR}/include/pplx/pplxcancellation_token.h"
        "${CMAKE_CURRENT_SOURCE_DIR}/include/pplx/pplx.h"
        "${CMAKE_CURRENT_SOURCE_DIR}/include/pplx/pplxinterface.h"
        "${CMAKE_CURRENT_SOURCE_DIR}/include/pplx/pplxtasks.h"
    PRIVATE
        "${CMAKE_CURRENT_SOURCE_DIR}/src/pplx.cpp"
        "${CMAKE_CURRENT_SOURCE_DIR}/src/stdafx.cpp"
        "${CMAKE_CURRENT_SOURCE_DIR}/src/stdafx.h"
)

# There's a dedicated thread pool implementation for the UNIX side, while Windows provides its own.
if(${CMAKE_SYSTEM_NAME} MATCHES "Linux")
    target_sources(${PROJECT_NAME}
        PUBLIC
            "${CMAKE_CURRENT_SOURCE_DIR}/include/pplx/pplxlinux.h"
        PRIVATE
            "${CMAKE_CURRENT_SOURCE_DIR}/src/pplxlinux.cpp"
            "${CMAKE_CURRENT_SOURCE_DIR}/src/threadpool.cpp"
            "${CMAKE_CURRENT_SOURCE_DIR}/src/threadpool.h"
    )
elseif(${CMAKE_SYSTEM_NAME} MATCHES "Darwin")
    target_sources(${PROJECT_NAME}
        PUBLIC
            "${CMAKE_CURRENT_SOURCE_DIR}/include/pplx/pplxlinux.h"
        PRIVATE
            "${CMAKE_CURRENT_SOURCE_DIR}/src/pplxapple.cpp"
    )
elseif(WIN32)
    target_sources(${PROJECT_NAME}
        PUBLIC
            "${CMAKE_CURRENT_SOURCE_DIR}/include/pplx/pplxwin.h"
        PRIVATE
            "${CMAKE_CURRENT_SOURCE_DIR}/src/pplxwin.cpp"
    )
endif(${CMAKE_SYSTEM_NAME} MATCHES "Linux")

set(include_install_dir "include")
set(lib_install_dir "lib")
set(bin_install_dir "bin")
install(
    DIRECTORY "${PROJECT_SOURCE_DIR}/include/pplx"
    DESTINATION ${include_install_dir}
)
install(TARGETS ${PROJECT_NAME}
    RUNTIME DESTINATION ${bin_install_dir}
    LIBRARY DESTINATION ${lib_install_dir}
    ARCHIVE DESTINATION ${lib_install_dir}
)