cmake_minimum_required ( VERSION 3.10 FATAL_ERROR )
list(INSERT CMAKE_MODULE_PATH 0 "${CMAKE_SOURCE_DIR}/cmake")
find_package(Dotnet 2.0 REQUIRED)

project(AsmTool NONE)
if(WIN32)
	set(REKO_COMPILER "Visual Studio 17 2022")
endif()

set(target_platform "")
if(WIN32)
	# required for asmdll interop
	set(target_platform PLATFORM x86)
endif()

ADD_DOTNET(${CMAKE_SOURCE_DIR}/AsmTool.sln
	${target_platform}
	CONFIG ${CMAKE_BUILD_TYPE}
	TARGET_NAME AsmTool_sln
)

if(UNIX)
	enable_language(C)
	add_subdirectory(AsmTool/Linux)
	add_dependencies(AsmTool_sln AsmIoLinux)
endif()

include(FindSevenZip)
execute_process(
	COMMAND git rev-parse HEAD
	WORKING_DIRECTORY ${CMAKE_SOURCE_DIR}
	OUTPUT_VARIABLE head_sha
)
string(SUBSTRING "${head_sha}" 0 10 build_shorthash)

set(file_list "")
list(APPEND file_list
	${CMAKE_BINARY_DIR}/AsmTool/net8.0/*
)

if(UNIX)
	list(APPEND file_list $<TARGET_FILE:AsmIoLinux>)
endif()

add_custom_target(create_zip
	COMMENT "Compressing..."
	COMMAND ${SEVENZIP_BIN} a
		${CMAKE_BINARY_DIR}/AsmTool-${build_shorthash}.zip
		${file_list}
)
add_dependencies(create_zip AsmTool_sln)


add_custom_target(main ALL)
add_dependencies(main create_zip)
