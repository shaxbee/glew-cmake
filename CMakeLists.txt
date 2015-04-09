cmake_minimum_required(VERSION 2.8.12)
project(glew LANGUAGES C)
set(GLEW_VERSION "1.12.0")

option(GLEW_USE_GLU "Use OpenGL Utilites library" OFF)

find_package(OpenGL REQUIRED)

list(APPEND GLEW_SOURCES src/glew.c)
list(APPEND GLEW_HEADERS include/GL/glew.h)
if(MSVC)
	list(APPEND GLEW_SOURCES build/glew.rc)
	list(APPEND GLEW_HEADERS include/GL/wglew.h)
elseif(OPENGL_XMESA_FOUND)
	list(APPEND GLEW_HEADERS include/GL/glxew.h)
endif()

add_library(glew ${GLEW_SOURCES} ${GLEW_HEADERS})

if(WIN32)
	target_compile_definitions(glew PRIVATE -DWIN32_MEAN_AND_LEAN -DVC_EXTRALEAN -DGLEW_BUILD -D_CRT_SECURE_NO_WARNINGS)
endif()

if(NOT GLEW_USE_GLU)
	target_compile_definitions(glew PUBLIC -DGLEW_NO_GLU)
endif()

target_link_libraries(glew INTERFACE ${OPENGL_LIBRARIES})

target_include_directories(
   glew PUBLIC
	$<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
	$<INSTALL_INTERFACE:include>)

include(GNUInstallDirs)

install(
    TARGETS glew 
	EXPORT glewTargets
	DESTINATION ${CMAKE_INSTALL_LIBDIR})

install(
    FILES ${GLEW_HEADERS}
	DESTINATION ${CMAKE_INSTALL_INCLUDEDIR})

include(CMakePackageConfigHelpers)
set(GLEW_INSTALL_CONFIGDIR "${CMAKE_INSTALL_LIBDIR}/cmake/glew")

write_basic_package_version_file(
    "${CMAKE_CURRENT_BINARY_DIR}/glewConfigVersion.cmake" 
	VERSION ${GLEW_VERSION} 
	COMPATIBILITY AnyNewerVersion)

export(
    EXPORT glewTargets
	FILE "${CMAKE_CURRENT_BINARY_DIR}/glewTargets.cmake")

configure_file(build/cmake/glewConfig.cmake
	glewConfig.cmake
	COPYONLY)

export(PACKAGE glew)

install(
    EXPORT glewTargets 
	FILE glewTargets.cmake
	DESTINATION ${GLEW_INSTALL_CONFIGDIR})

install(
	FILES 
		"${CMAKE_CURRENT_BINARY_DIR}/glewConfig.cmake"
		"${CMAKE_CURRENT_BINARY_DIR}/glewConfigVersion.cmake"
	DESTINATION
		${GLEW_INSTALL_CONFIGDIR})