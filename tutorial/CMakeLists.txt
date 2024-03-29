cmake_minimum_required(VERSION 3.6)

set(CMAKE_BUILD_TYPE RelWithDebInfo CACHE STRING "build type")

project(tutorial
		LANGUAGES C CXX
)

if (WIN32)
	option(WORKFLOW_TUTORIAL_BUILD_STATIC_RUNTIME "Use static runtime" ON)
endif ()

set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${PROJECT_SOURCE_DIR})

find_library(LIBRT rt)
find_package(OpenSSL REQUIRED)
find_package(workflow REQUIRED CONFIG HINTS ..)
include_directories(${OPENSSL_INCLUDE_DIR} ${WORKFLOW_INCLUDE_DIR})
link_directories(${WORKFLOW_LIB_DIR})

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

set(TUTORIAL_LIST
	tutorial-01-wget
	tutorial-02-redis_cli
	tutorial-03-wget_to_redis
	tutorial-04-http_echo_server
	tutorial-05-http_proxy
	tutorial-06-parallel_wget
	tutorial-07-sort_task
	tutorial-08-matrix_multiply
	tutorial-09-http_file_server
	tutorial-12-mysql_cli
)

if (APPLE)
	set(WORKFLOW_LIB workflow pthread OpenSSL::SSL OpenSSL::Crypto)
elseif (WIN32)
	set(WORKFLOW_LIB workflow ws2_32 wsock32 OpenSSL::SSL OpenSSL::Crypto)
else ()
	set(WORKFLOW_LIB workflow pthread OpenSSL::SSL OpenSSL::Crypto ${LIBRT})
endif ()

foreach(src ${TUTORIAL_LIST})
	string(REPLACE "-" ";" arr ${src})
	list(GET arr -1 bin_name)
	add_executable(${bin_name} ${src}.cc)
	target_link_libraries(${bin_name} ${WORKFLOW_LIB})
	if (WIN32)
		target_compile_definitions(
			${bin_name} PRIVATE
			strdup=_strdup
			strcasecmp=_stricmp
			strncasecmp=_strnicmp
		)
	endif ()
endforeach()

if (KAFKA STREQUAL "y")
	add_executable("kafka_cli" "tutorial-13-kafka_cli.cc")
	if(NOT WIN32)
		target_link_libraries("kafka_cli" wfkafka workflow z snappy lz4 zstd rt)
	else()
		find_package(OpenSSL REQUIRED)
		find_package(ZLIB REQUIRED)
		find_package(Snappy CONFIG REQUIRED)
		find_package(lz4 CONFIG REQUIRED)
		find_package(zstd CONFIG REQUIRED)

		target_link_libraries("kafka_cli" wfkafka ${WORKFLOW_LIB} OpenSSL::Crypto ZLIB::ZLIB Snappy::snappy lz4::lz4 zstd::libzstd_shared)
		target_compile_definitions(
			"kafka_cli" PRIVATE
			strdup=_strdup
			strcasecmp=_stricmp
			strncasecmp=_strnicmp
		)	
	endif(NOT WIN32)
endif ()

set(DIR10 tutorial-10-user_defined_protocol)
add_executable(server ${DIR10}/server.cc ${DIR10}/message.cc)
add_executable(client ${DIR10}/client.cc ${DIR10}/message.cc)
target_link_libraries(server ${WORKFLOW_LIB})
target_link_libraries(client ${WORKFLOW_LIB})

set_target_properties(server PROPERTIES
	RUNTIME_OUTPUT_DIRECTORY ${PROJECT_SOURCE_DIR}/${DIR10})
set_target_properties(client PROPERTIES
	RUNTIME_OUTPUT_DIRECTORY ${PROJECT_SOURCE_DIR}/${DIR10})

