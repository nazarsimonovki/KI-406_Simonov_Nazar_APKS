cmake_minimum_required(VERSION 3.20)
project(HelloWorldApp LANGUAGES CXX)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)

include(CTest)

add_executable(hello src/main.cpp)

if(BUILD_TESTING)
    include(FetchContent)

    FetchContent_Declare(
        googletest
        GIT_REPOSITORY https://github.com/google/googletest.git
        GIT_TAG v1.14.0
    )
    FetchContent_MakeAvailable(googletest)

    add_executable(project_tests tests/test_main.cpp)
    target_link_libraries(project_tests PRIVATE GTest::gtest_main)
    target_include_directories(project_tests PRIVATE src)

    include(GoogleTest)
    gtest_discover_tests(project_tests)
endif()
