cmake_minimum_required(VERSION 3.20)
project(HelloWorldApp LANGUAGES CXX)

include(CTest)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)

add_executable(hello src/main.cpp)

if(BUILD_TESTING)
	include(FetchContent)

	set(gtest_force_shared_crt ON CACHE BOOL "" FORCE)
	if(POLICY CMP0135)
		cmake_policy(SET CMP0135 NEW)
	endif()
	FetchContent_Declare(
		googletest
		URL https://github.com/google/googletest/archive/refs/tags/v1.14.0.zip
	)
	FetchContent_MakeAvailable(googletest)

	add_executable(project_tests tests/test_main.cpp)
	target_include_directories(project_tests PRIVATE src)
	target_link_libraries(project_tests PRIVATE GTest::gtest_main)

	include(GoogleTest)
	gtest_discover_tests(project_tests)
endif()
