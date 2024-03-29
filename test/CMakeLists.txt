cmake_minimum_required(VERSION 3.6)

set(CMAKE_BUILD_TYPE RelWithDebInfo CACHE STRING "build type")

project(workflow_test
		LANGUAGES C CXX
)

if (WIN32)
	option(WORKFLOW_TUTORIAL_BUILD_STATIC_RUNTIME "Use static runtime" ON)
endif ()

find_library(LIBRT rt)
find_package(OpenSSL REQUIRED)
find_package(workflow REQUIRED CONFIG HINTS ..)
include_directories(${OPENSSL_INCLUDE_DIR} ${WORKFLOW_INCLUDE_DIR})
link_directories(${WORKFLOW_LIB_DIR})

find_program(CMAKE_MEMORYCHECK_COMMAND valgrind)
set(memcheck_command ${CMAKE_MEMORYCHECK_COMMAND} ${CMAKE_MEMORYCHECK_COMMAND_OPTIONS} --error-exitcode=1 --leak-check=full)

add_custom_target(check COMMAND ${CMAKE_CTEST_COMMAND})

enable_testing()
find_package(GTest CONFIG REQUIRED)

if (WIN32)
	if (WORKFLOW_TUTORIAL_BUILD_STATIC_RUNTIME)
		set(CompilerFlags
				CMAKE_CXX_FLAGS
				CMAKE_CXX_FLAGS_DEBUG
				CMAKE_CXX_FLAGS_RELEASE
				CMAKE_CXX_FLAGS_MINSIZEREL
				CMAKE_C_FLAGS
				CMAKE_C_FLAGS_DEBUG
				CMAKE_C_FLAGS_RELEASE
				CMAKE_C_FLAGS_MINSIZEREL
				)
		foreach(CompilerFlag ${CompilerFlags})
			string(REPLACE "/MD" "/MT" ${CompilerFlag} "${${CompilerFlag}}")
		endforeach ()
	endif ()
endif ()

if (WIN32)
		set(CMAKE_C_FLAGS   "${CMAKE_C_FLAGS}   /MP /wd4200")
		set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} /MP /wd4200 /Zc:__cplusplus /std:c++14")
else ()
		set(CMAKE_C_FLAGS   "${CMAKE_C_FLAGS}   -Wall -fPIC -pipe -std=gnu90")
		set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wall -fPIC -pipe -std=c++11 -fno-exceptions")
endif ()

set(TEST_LIST
	task_unittest
	algo_unittest
	http_unittest
	redis_unittest
	mysql_unittest
	facilities_unittest
	upstream_unittest
)

set(GTEST_LIB GTest::GTest GTest::Main)

if (APPLE)
	set(WORKFLOW_LIB workflow pthread OpenSSL::SSL OpenSSL::Crypto)
elseif (WIN32)
	set(WORKFLOW_LIB workflow ws2_32 wsock32 OpenSSL::SSL OpenSSL::Crypto)
	set(GTEST_LIB GTest::gtest GTest::gtest_main)
else ()
	set(WORKFLOW_LIB workflow ${LIBRT})
endif ()

foreach(src ${TEST_LIST})
	add_executable(${src} EXCLUDE_FROM_ALL ${src}.cc)
	target_link_libraries(${src} ${WORKFLOW_LIB} ${GTEST_LIB})
	if (WIN32)
		target_compile_definitions(
			${src} PRIVATE
			strdup=_strdup
			strcasecmp=_stricmp
			strncasecmp=_strnicmp
		)
	endif ()
	add_test(${src} ${src})
	add_dependencies(check ${src})
endforeach()

foreach(src ${TEST_LIST})
	add_test(${src}-memory-check ${memcheck_command} ./${src})
endforeach()

