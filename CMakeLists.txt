cmake_minimum_required(VERSION 2.8.12)
project(glew)

option(GLEW_USE_GLU "Use OpenGL Utilites library" OFF)

set(GLEW_VERSION_MAJOR 1)
set(GLEW_VERSION_MINOR 12)
set(GLEW_VERSION_PATCH 0)
set(GLEW_VERSION "${GLEW_VERSION_MAJOR}.${GLEW_VERSION_MINOR}.${GLEW_VERSION_PATCH}")

find_package(OpenGL REQUIRED)

list(APPEND GLEW_SOURCES src/glew.c)
if(MSVC)
	list(APPEND GLEW_SOURCES build/glew.rc)
endif()

add_library(glew ${GLEW_SOURCES})

if(WIN32)
	target_compile_definitions(glew PRIVATE -DWIN32_MEAN_AND_LEAN -DVC_EXTRALEAN -DGLEW_BUILD -D_CRT_SECURE_NO_WARNINGS)
endif()

if(NOT GLEW_USE_GLU)
	target_link_libraries(glew INTERFACE ${OPENGL_gl_LIBRARY})
	target_compile_definitions(glew PRIVATE -DGLEW_NO_GLU)
else()
	target_link_libraries(glew INTERFACE ${OPENGL_LIBRARIES})
endif()

target_include_directories(glew PUBLIC
	$<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
	$<INSTALL_INTERFACE:include>)

include(GNUInstallDirs)

install(TARGETS glew 
	EXPORT glewTargets 
	DESTINATION ${CMAKE_INSTALL_LIBDIR})

install(DIRECTORY include/GL DESTINATION ${CMAKE_INSTALL_INCLUDEDIR} 
	FILES_MATCHING PATTERN "*.h")

include(CMakePackageConfigHelpers)
set(GLEW_INSTALL_CONFIGDIR "${CMAKE_INSTALL_LIBDIR}/cmake/glew")

write_basic_package_version_file("${CMAKE_CURRENT_BINARY_DIR}/glewConfigVersion.cmake" 
	VERSION ${GLEW_VERSION} 
	COMPATIBILITY AnyNewerVersion)

configure_file(build/cmake/glewConfig.cmake
	"${CMAKE_CURRENT_BINARY_DIR}/glewConfig.cmake"
	COPYONLY)

export(EXPORT glewTargets
	FILE "${CMAKE_CURRENT_BINARY_DIR}/glewTargets.cmake")

install(EXPORT glewTargets 
	FILE glewTargets.cmake
	DESTINATION ${GLEW_INSTALL_CONFIGDIR})

install(
	FILES 
		"${CMAKE_CURRENT_BINARY_DIR}/glewConfig.cmake"
		"${CMAKE_CURRENT_BINARY_DIR}/glewConfigVersion.cmake"
	DESTINATION
		${GLEW_INSTALL_CONFIGDIR})