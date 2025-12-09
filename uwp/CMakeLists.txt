cmake_minimum_required(VERSION 3.15)

include(FetchContent)

# Setup UWP platform
set(CMAKE_SYSTEM_NAME WindowsStore)
set(CMAKE_SYSTEM_VERSION 10.0.19041.0)
set(CMAKE_CXX_STANDARD 20)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

if(NOT CMAKE_BUILD_TYPE)
  set(CMAKE_BUILD_TYPE "Release" CACHE STRING "Build type (Debug, Release, RelWithDebInfo, MinSizeRel)" FORCE)
endif()

# Define project
project(openjk-uwp LANGUAGES CXX)
set_directory_properties(PROPERTIES VS_STARTUP_PROJECT ${PROJECT_NAME})

# Config for main project path
set(OPENJK_DIR "" CACHE PATH "")
if(NOT OPENJK_DIR)
	message(FATAL_ERROR "OPENJK_DIR must be set to a valid path pointing to the openjk cmake build dir")
endif()

FetchContent_Declare(UwpDep
	GIT_REPOSITORY https://github.com/worleydl/uwp-dep.git
	GIT_TAG da75dd2240b6eca7848aebc2f72433bce1a15e3b
)
FetchContent_MakeAvailable(UwpDep)

add_executable(${PROJECT_NAME} WIN32
	src/main.cpp
)
set_target_properties(${PROJECT_NAME} PROPERTIES VS_WINDOWS_TARGET_PLATFORM_MIN_VERSION 10.0.19041.0)

set(BinDeps
	# OpenJK DLL's
	${OPENJK_DIR}/${CMAKE_BUILD_TYPE}/openjk_sp.x86_64.dll
	${OPENJK_DIR}/${CMAKE_BUILD_TYPE}/jagamex86_64.dll
	${OPENJK_DIR}/${CMAKE_BUILD_TYPE}/rdsp-vanilla_x86_64.dll

	# Dependency DLL's
	${CMAKE_BINARY_DIR}/_deps/uwpdep-src/x64/bin/dxil.dll
	${CMAKE_BINARY_DIR}/_deps/uwpdep-src/x64/bin/libuwp.dll
	${CMAKE_BINARY_DIR}/_deps/uwpdep-src/x64/bin/libgallium_wgl.dll
	${CMAKE_BINARY_DIR}/_deps/uwpdep-src/x64/bin/opengl32.dll
	${CMAKE_BINARY_DIR}/_deps/uwpdep-src/x64/bin/SDL2.dll
	${CMAKE_BINARY_DIR}/_deps/uwpdep-src/x64/bin/z-1.dll

	Package.appxmanifest
)
source_group("BinDeps" FILES ${BinDeps})
set_source_files_properties(${BinDeps} PROPERTIES
	VS_COPY_TO_OUT_DIR Always
	VS_DEPLOYMENT_CONTENT TRUE
	VS_DEPLOYMENT_LOCATION "."
)

set(PkgAssets
	Assets/LockScreenLogo.scale-200.png
	Assets/Square44x44Logo.targetsize-24_altform-unplated.png
	Assets/Square44x44Logo.scale-200.png
	Assets/Square150x150Logo.scale-200.png
	Assets/StoreLogo.png
)
source_group("PkgAssets" FILES ${PkgAssets})
set_source_files_properties(${PkgAssets} PROPERTIES
	#VS_COPY_TO_OUT_DIR Always
	VS_DEPLOYMENT_CONTENT TRUE
	VS_DEPLOYMENT_LOCATION "Assets"
)

target_sources(${PROJECT_NAME} PRIVATE
	${BinDeps}
	${PkgAssets}
)

target_include_directories(${PROJECT_NAME} PRIVATE
	${CMAKE_BINARY_DIR}/_deps/uwpdep-src/x64/include
)

set(BinLibs
	# OpenJK libs
	${OPENJK_DIR}/code/${CMAKE_BUILD_TYPE}/openjk_sp.x86_64.lib

	# Other dep libs
	${CMAKE_BINARY_DIR}/_deps/uwpdep-src/x64/lib/libuwp.lib
	${CMAKE_BINARY_DIR}/_deps/uwpdep-src/x64/lib/SDL2.lib

	# System libs
	OpenGL32
	OneCore
	WindowsApp
)

target_link_libraries(${PROJECT_NAME} PRIVATE ${BinLibs})
