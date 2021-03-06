cmake_minimum_required(VERSION 2.8)
project(lifthttp CXX)

find_program(CLANG_TIDY_BIN NAMES "clang-tidy" DOC "clang-tidy binary location")
if(CLANG_TIDY_BIN)
    set(CLANG_TIDY_COMMAND "${CLANG_TIDY_BIN}" "-checks=*,-cppcoreguidelines-pro-type-union-access,-hicpp-signed-bitwise,-cppcoreguidelines-pro-bounds-pointer-arithmetic,-google-runtime-int,-hicpp-vararg,-cppcoreguidelines-pro-type-vararg,-cert-err58-cpp,-clang-analyzer-alpha.*,-fuchsia-statically-constructed-objects,-fuchsia-overloaded-operator,-fuchsia-trailing-return,-fuchsia-default-arguments,-readability-else-after-return,-google-readability-namespace-comments,-llvm-namespace-comment,-hicpp-no-array-decay,-cppcoreguidelines-pro-bounds-array-to-pointer-decay,-google-default-arguments,-llvm-header-guard,-google-runtime-references,-misc-macro-parentheses,-bugprone-macro-parentheses,-bugprone-unused-return-value" "-header-filter=.*")
endif()

set(CMAKE_EXPORT_COMPILE_COMMANDS ON)

if(NOT DEFINED CURL_INCLUDE)
    set(CURL_INCLUDE "" CACHE PATH "User specified libcurl include directory.")
endif()
if(NOT DEFINED LIBSSL)
    set(LIBSSL "" CACHE FILEPATH "User specified libssl static location.")
endif()
if(NOT DEFINED LIBCRYPTO)
    set(LIBCRYPTO "" CACHE FILEPATH "User specificed libcrypto static location.")
endif()
if(NOT DEFINED LIBCURL)
    set(LIBCURL "-lcurl" CACHE FILEPATH "User specified libcurl static location.")
endif()
if(NOT DEFINED LIBCARES)
    set(LIBCARES "" CACHE FILEPATH "User specified libcares static location.")
endif()

message("${PROJECT_NAME} CURL_INCLUDE = ${CURL_INCLUDE}")
message("${PROJECT_NAME} LIBSSL = ${LIBSSL}")
message("${PROJECT_NAME} LIBCRYPTO = ${LIBCRYPTO}")
message("${PROJECT_NAME} LIBCURL = ${LIBCURL}")
message("${PROJECT_NAME} LIBCARES = ${LIBCARES}")

set(LIFT_SYSTEM_LIBRARY_DEPENDENCIES
    uv
    pthread
    z
    dl
    stdc++fs
)

set(LIFT_CURL_LIBRARY_DEPENDENCIES
    "-Wl,--start-group ${LIBSSL} ${LIBCRYPTO} ${LIBCURL} ${LIBCARES} -Wl,--end-group"
)

set(LIFT_LIB_DEPS "${LIFT_SYSTEM_LIBRARY_DEPENDENCIES} ${LIFT_CURL_LIBRARY_DEPENDENCIES}")

set(LIBLIFT_SOURCE_FILES
    inc/lift/Escape.hpp src/Escape.cpp
    inc/lift/EventLoop.hpp inc/lift/EventLoop.tcc src/EventLoop.cpp
    inc/lift/HeaderView.hpp src/HeaderView.cpp
    inc/lift/Lift.hpp src/Lift.cpp
    inc/lift/Http.hpp src/Http.cpp
    inc/lift/QueryBuilder.hpp src/QueryBuilder.cpp
    inc/lift/Request.hpp src/Request.cpp
    inc/lift/RequestHandle.hpp src/RequestHandle.cpp
    inc/lift/RequestPool.hpp src/RequestPool.cpp
    inc/lift/RequestStatus.hpp src/RequestStatus.cpp
    inc/lift/ResolveHost.hpp src/ResolveHost.cpp
)

add_library(${PROJECT_NAME} STATIC SHARED ${LIBLIFT_SOURCE_FILES})
target_compile_features(${PROJECT_NAME} PUBLIC cxx_std_17)

target_include_directories(${PROJECT_NAME} SYSTEM PUBLIC ${CURL_INCLUDE})
target_include_directories(${PROJECT_NAME} PUBLIC ${CMAKE_CURRENT_SOURCE_DIR}/inc)

target_link_libraries(${PROJECT_NAME} PUBLIC ${LIFT_LIB_DEPS})

if(${CMAKE_CXX_COMPILER_ID} MATCHES "GNU")
    target_compile_options(
        ${PROJECT_NAME} PRIVATE
        -Wno-unknown-pragmas
    )
endif()

if(CLANG_TIDY_BIN)
    set_target_properties(${PROJECT_NAME} PROPERTIES CXX_CLANG_TIDY "${CLANG_TIDY_COMMAND}")
endif()

add_subdirectory(examples)
enable_testing()
add_subdirectory(test)
