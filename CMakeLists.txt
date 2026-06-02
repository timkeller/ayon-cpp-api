cmake_minimum_required(VERSION 3.19 FATAL_ERROR)

# Ensure CMAKE_EXPORT_COMPILE_COMMANDS is set before project() call
set(CMAKE_EXPORT_COMPILE_COMMANDS ON)

set(AR_PROJECT_NAME AyonCppApi)

# set(BUILD_SHARED_LIBS ON)
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_POSITION_INDEPENDENT_CODE ON)

if(NOT DEFINED JTRACE)
  set(JTRACE 0)
endif()
add_compile_definitions(JTRACE=${JTRACE})

option(BUILD_TEST "Build the test application" OFF)

set(CMAKE_INSTALL_PREFIX ${CMAKE_CURRENT_SOURCE_DIR}/bin)
set(JSON_BuildTests OFF CACHE INTERNAL "")

if(WIN32)
  set(OPENSSL_ROOT_DIR "${CMAKE_CURRENT_SOURCE_DIR}/ext/opensslW")
elseif(APPLE)
  # macOS: no vendored darwin OpenSSL blob; use Homebrew openssl@3.
  # Honour a caller-provided OPENSSL_ROOT_DIR if one was passed in.
  if(NOT OPENSSL_ROOT_DIR)
    set(OPENSSL_ROOT_DIR "/opt/homebrew/opt/openssl@3")
  endif()
else()
  set(OPENSSL_ROOT_DIR "${CMAKE_CURRENT_SOURCE_DIR}/ext/opensslL")
  # find_package(OpenSSL REQUIRED)
endif()
include_directories(${OPENSSL_ROOT_DIR}/include)


add_subdirectory("${CMAKE_CURRENT_SOURCE_DIR}/ext/backward-cpp")
add_subdirectory("${CMAKE_CURRENT_SOURCE_DIR}/ext/cpp-httplib" EXCLUDE_FROM_ALL)
include_directories("${CMAKE_CURRENT_SOURCE_DIR}/ext/cpp-httplib")
add_subdirectory("${CMAKE_CURRENT_SOURCE_DIR}/ext/json" EXCLUDE_FROM_ALL)
include_directories("${CMAKE_CURRENT_SOURCE_DIR}/ext/json/include")
# add_subdirectory("${CMAKE_CURRENT_SOURCE_DIR}/ext/spdlog" EXCLUDE_FROM_ALL)
# include_directories("${CMAKE_CURRENT_SOURCE_DIR}/ext/spdlog/include")

set(AYON_CPPTOOLS_BUILD_LOGGER 1)
add_subdirectory("${CMAKE_CURRENT_SOURCE_DIR}/ext/ayon-cpp-dev-tools")

project(${AR_PROJECT_NAME} VERSION 1.0.0 LANGUAGES CXX)

if(AYON_CPP_API_ENABLE_GTEST)
  add_subdirectory("${CMAKE_CURRENT_SOURCE_DIR}/ext/googletest")
endif()
if(AYON_CPP_API_ENABLE_GBENCH)
  add_subdirectory("${CMAKE_CURRENT_SOURCE_DIR}/ext/benchmark")
endif()

if(AYON_CPP_API_ENABLE_GTEST OR AYON_CPP_API_ENABLE_GBENCH)
  add_subdirectory("${CMAKE_CURRENT_SOURCE_DIR}/test")  
endif()


add_subdirectory("${CMAKE_CURRENT_SOURCE_DIR}/src/AyonCppApi")
include_directories("${CMAKE_CURRENT_SOURCE_DIR}/src/AyonCppApi")

target_link_libraries(AyonCppApi PUBLIC AyonCppDevToolsLib)