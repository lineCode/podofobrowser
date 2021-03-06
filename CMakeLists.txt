CMAKE_MINIMUM_REQUIRED(VERSION 2.4)
PROJECT(PoDoFoBrowser)

# See CMake documentation on CMP0003
if(COMMAND cmake_policy)
  cmake_policy(SET CMP0003 NEW)
endif(COMMAND cmake_policy)

SET(CMAKE_MODULE_PATH "${CMAKE_SOURCE_DIR}/cmake/modules")

MESSAGE("Remember to set QTDIR in your environment if qmake is not found")
SET(QT_MIN_VERSION "4.2.0")
SET(QT_USE_QT3SUPPORT FALSE)
SET(QT_USE_QTUITOOLS TRUE)
SET(QT_REQUIRED TRUE)
FIND_PACKAGE(Qt4 REQUIRED)
INCLUDE(${QT_USE_FILE})

# These were commented out with no explanation as to why.
# PoDoFoBrowser cannot link to a static libpodofo without
# also linking to its dependencies, like libjpeg. On Mac OS
# X, this is even required to link to a dynamic libpodofo.
FIND_PACKAGE(ZLIB REQUIRED)
FIND_PACKAGE(LIBFREETYPE REQUIRED)
FIND_PACKAGE(LIBJPEG)

FIND_PACKAGE(LIBPODOFO)
IF(NOT LIBPODOFO_FOUND)
	MESSAGE("+++PoDoFo not found... building private copy")
	SET(PODOFO_BUILD_STATIC 1)
	SET(PODOFO_BUILD_SHARED 0)
	SET(PODOFO_BUILD_LIB_ONLY 1)
	ADD_SUBDIRECTORY(externals/required_libpodofo)
	SET(LIBPODOFO_H ${CMAKE_CURRENT_SOURCE_DIR}/externals/required_libpodofo)
	SET(LIBPODOFO_CONFIG_H ${CMAKE_CURRENT_BINARY_DIR}/externals/required_libpodofo)
	SET(LIBPODOFO_LIB podofo_static)
	MESSAGE("+++Done setting up private PoDoFo copy")
ENDIF(NOT LIBPODOFO_FOUND)


FIND_PACKAGE(LIBSTLPORT)
SET(stlport_libraries_if_use_stlport)
IF(USE_STLPORT)
	IF(LIBSTLPORT_FOUND)
		MESSAGE("Using STLPort. Make Qt and any other C++ libs you require also use it.")
		INCLUDE_DIRECTORIES(${LIBSTLPORT_HEADERS})
		LINK_DIRECTORIES(${LIBSTLPORT_LIB})
		SET(stlport_libraries_if_use_stlport stlport)
		# Use the threaded STLPort
		ADD_DEFINITIONS(-D_PTHREADS)
	ELSE(LIBSTLPORT_FOUND)
		MESSAGE(FATAL_ERROR "STLPort use requested, but STLPort not found.")
	ENDIF(LIBSTLPORT_FOUND)
ENDIF(USE_STLPORT)

INCLUDE_DIRECTORIES(
	${LIBZ_H}
	${LIBFREETYPE_FT2BUILD_H}
	${LIBFREETYPE_FTHEADER_H}
	${LIBFONTCONFIG_H}
	${LIBPODOFO_H}
	${LIBPODOFO_CONFIG_H}
	)

MESSAGE("ZLIB headers: ${LIBZ_H}")
MESSAGE("LIBFREETYPE headers: ${LIBFREETYPE_FT2BUILD_H} ${LIBFREETYPE_FTHEADER_H}")
MESSAGE("LIBJPEG headers: ${LIBJPEG_H}")

#IF(NOT WIN32)
#	FIND_PACKAGE(LIBFONTCONFIG)
#	IF(LIBFONTCONFIG_FOUND)
#		INCLUDE_DIRECTORIES(${LIBFONTCONFIG_H})
#	ENDIF(LIBFONTCONFIG_FOUND)

#	MESSAGE("LIBFONTCONFIG headers: ${LIBFONTCONFIG_H} (found: ${LIBFONTCONFIG_FOUND})")
#ENDIF(NOT WIN32)

ADD_SUBDIRECTORY(src)
