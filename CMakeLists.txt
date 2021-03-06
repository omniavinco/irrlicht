CMAKE_MINIMUM_REQUIRED( VERSION 2.6 )
PROJECT( irrlicht )
CMAKE_POLICY(SET CMP0012 NEW)

IF(NOT CMAKE_LIBRARY_OUTPUT_DIRECTORY )
	SET( CMAKE_LIBRARY_OUTPUT_DIRECTORY "${CMAKE_SOURCE_DIR}/lib" )
ENDIF(  )
IF(NOT CMAKE_ARCHIVE_OUTPUT_DIRECTORY )
	SET( CMAKE_ARCHIVE_OUTPUT_DIRECTORY "${CMAKE_SOURCE_DIR}/lib" )
ENDIF(  )
IF(NOT CMAKE_RUNTIME_OUTPUT_DIRECTORY )
	SET( CMAKE_RUNTIME_OUTPUT_DIRECTORY "${CMAKE_SOURCE_DIR}/bin" )
ENDIF(  )

SET( IRRLICHT_ROOT_DIR "${CMAKE_CURRENT_SOURCE_DIR}" )
SET( IRRLICHT_SRC_DIR "${CMAKE_CURRENT_SOURCE_DIR}/source/irrlicht" )
SET( IRRLICHT_EXT_DIR "${CMAKE_CURRENT_SOURCE_DIR}/source/irrlicht" )
SET( IRRLICHT_INC_DIR "${CMAKE_CURRENT_SOURCE_DIR}/include" )

LIST( APPEND CMAKE_MODULE_PATH "${IRRLICHT_ROOT_DIR}/CMake" )

INCLUDE_DIRECTORIES( "${IRRLICHT_INC_DIR}" "${IRRLICHT_SRC_DIR}" )

FILE( GLOB IRRLICHT_SRC_FILES "${IRRLICHT_SRC_DIR}/*.cpp" "${IRRLICHT_SRC_DIR}/*.c" "${IRRLICHT_SRC_DIR}/*.h" "${IRRLICHT_SRC_DIR}/*.hpp" )

FILE( GLOB AESGLADMAN_SRC_FILES "${IRRLICHT_EXT_DIR}/aesGladman/*.cpp" )
INCLUDE_DIRECTORIES( "${IRRLICHT_EXT_DIR}/aesGladman" )

FILE( GLOB BZIP2_SRC_FILES "${IRRLICHT_EXT_DIR}/bzip2/*.c" )
INCLUDE_DIRECTORIES( "${IRRLICHT_EXT_DIR}/bzip2" )
LIST( REMOVE_ITEM BZIP2_SRC_FILES "${IRRLICHT_EXT_DIR}/bzip2/bzip2.c" "${IRRLICHT_EXT_DIR}/bzip2/bzip2recover.c" "${IRRLICHT_EXT_DIR}/bzip2/dlltest.c" "${IRRLICHT_EXT_DIR}/bzip2/mk251.c" "${IRRLICHT_EXT_DIR}/bzip2/spewG.c" "${IRRLICHT_EXT_DIR}/bzip2/unzcrash.c" )

FILE( GLOB LZMA_SRC_FILES "${IRRLICHT_EXT_DIR}/lzma/*.c" )
INCLUDE_DIRECTORIES( "${IRRLICHT_EXT_DIR}/lzma" )

FILE( GLOB ZLIB_SRC_FILES "${IRRLICHT_EXT_DIR}/zlib/*.c" )
INCLUDE_DIRECTORIES( "${IRRLICHT_EXT_DIR}/zlib" )

LIST( REMOVE_ITEM ZLIB_SRC_FILES "${IRRLICHT_EXT_DIR}/zlib/gzclose.c" "${IRRLICHT_EXT_DIR}/zlib/gzlib.c" "${IRRLICHT_EXT_DIR}/zlib/gzread.c" "${IRRLICHT_EXT_DIR}/zlib/gzwrite.c" "${IRRLICHT_EXT_DIR}/zlib/infback.c" )
FILE( GLOB JPEGLIB_SRC_FILES "${IRRLICHT_EXT_DIR}/jpeglib/j*.c" )
INCLUDE_DIRECTORIES( "${IRRLICHT_EXT_DIR}/jpeglib" )

LIST( REMOVE_ITEM JPEGLIB_SRC_FILES "${IRRLICHT_EXT_DIR}/jpeglib/jmemdos.c" "${IRRLICHT_EXT_DIR}/jpeglib/jmemmac.c" "${IRRLICHT_EXT_DIR}/jpeglib/jmemname.c" "${IRRLICHT_EXT_DIR}/jpeglib/jmemansi.c" "${IRRLICHT_EXT_DIR}/jpeglib/jpegtran.c" )
FILE( GLOB LIBPNG_SRC_FILES "${IRRLICHT_EXT_DIR}/libpng/*.c" )
INCLUDE_DIRECTORIES( "${IRRLICHT_EXT_DIR}/libpng" )

IF( APPLE )
	FILE( GLOB IRRLICHT_OSX_SRC_FILES "${IRRLICHT_SRC_DIR}/MacOSX/*.mm" "${IRRLICHT_SRC_DIR}/MacOSX/*.h" )
	LIST( APPEND IRRLICHT_SRC_FILES ${IRRLICHT_OSX_SRC_FILES} )
	INCLUDE_DIRECTORIES( "${IRRLICHT_SRC_DIR}/MacOSX" )
	FIND_LIBRARY( OPENGL_LIB OpenGL )
	FIND_LIBRARY( COCOA_LIB Cocoa )
	FIND_LIBRARY( CARBON_LIB Carbon )
	FIND_LIBRARY( IOKIT_LIB IOKit )
	LIST( APPEND IRRLICHT_LINK_LIBS ${OPENGL_LIB} ${IOKIT_LIB} ${COCOA_LIB} ${CARBON_LIB} )
	ADD_DEFINITIONS( -DMACOSX )

	SET_SOURCE_FILES_PROPERTIES("${IRRLICHT_SRC_DIR}/Irrlicht.cpp" "${IRRLICHT_SRC_DIR}/COpenGLDriver.cpp" PROPERTIES COMPILE_FLAGS "-x objective-c++")
ELSEIF( WIN32 )
	INCLUDE( FindDirectX )
	IF( DirectX_FOUND )
		INCLUDE_DIRECTORIES( ${DirectX_INCLUDE_DIR} )
		LIST( APPEND IRRLICHT_LINK_LIBS ${DirectX_LIBRARIES} )
	ELSE(  )
		MESSAGE( WARNING "Can't find DirectX SDK in your build system" )
		ADD_DEFINITIONS( -DNO_IRR_COMPILE_WITH_DIRECT3D_9_)
	ENDIF(  )
	#LIST( APPEND JPEGLIB_SRC_FILES ${IRRLICHT_SRC_DIR}/jpeglib/jmemdos.c )
	ADD_DEFINITIONS( -D_USRDLL -DIRRLICHT_EXPORTS -D_CRT_SECURE_NO_DEPRECATE )
ENDIF(  )

SOURCE_GROUP( Irrlicht\\libraries\\aesGladman FILES ${AESGLADMAN_SRC_FILES} )
SOURCE_GROUP( Irrlicht\\libraries\\bzip2 FILES ${BZIP2_SRC_FILES} )
SOURCE_GROUP( Irrlicht\\libraries\\lzma FILES ${LZMA_SRC_FILES} )
SOURCE_GROUP( Irrlicht\\libraries\\zlib FILES ${ZLIB_SRC_FILES} )
SOURCE_GROUP( Irrlicht\\libraries\\jpeglib FILES ${JPEGLIB_SRC_FILES} )
SOURCE_GROUP( Irrlicht\\libraries\\libpng FILES ${LIBPNG_SRC_FILES} )
SOURCE_GROUP( Irrlicht\\Engine FILES ${IRRLICHT_SRC_FILES} )

SET( IRRLICHT_LIB_FILES ${AESGLADMAN_SRC_FILES} ${BZIP2_SRC_FILES} ${LZMA_SRC_FILES} ${ZLIB_SRC_FILES} ${JPEGLIB_SRC_FILES} ${LIBPNG_SRC_FILES} )


ADD_LIBRARY( irrlicht_static STATIC ${IRRLICHT_SRC_FILES} ${IRRLICHT_LIB_FILES} )
ADD_LIBRARY( irrlicht_shared SHARED ${IRRLICHT_SRC_FILES} ${IRRLICHT_LIB_FILES} )

IF( WIN32 )
	SET_TARGET_PROPERTIES( irrlicht_shared PROPERTIES RUNTIME_OUTPUT_NAME irrlicht )
ELSEIF( APPLE )
	IF( XCODE_VERSION MATCHES ^4\\. )
		# To compile irrlicht on Xcode4, we need to use llvm gcc
		SET_TARGET_PROPERTIES( irrlicht_static PROPERTIES XCODE_ATTRIBUTE_GCC_VERSION "com.apple.compilers.llvmgcc42" )
		SET_TARGET_PROPERTIES( irrlicht_shared PROPERTIES XCODE_ATTRIBUTE_GCC_VERSION "com.apple.compilers.llvmgcc42" )
	ENDIF(  )
ENDIF(  )
SET_TARGET_PROPERTIES( irrlicht_static PROPERTIES LIBRARY_OUTPUT_NAME irrlicht ARCHIVE_OUTPUT_NAME irrlicht )
TARGET_LINK_LIBRARIES( irrlicht_shared ${IRRLICHT_LINK_LIBS} )

################################################################################
# Export Irrlicht link libraries & include directory
IF( NOT (CMAKE_CURRENT_SOURCE_DIR STREQUAL CMAKE_SOURCE_DIR) )
	SET( IRRLICHT_LINK_LIBS ${IRRLICHT_LINK_LIBS} PARENT_SCOPE )
	SET( IRRLICHT_INC_DIR "${CMAKE_CURRENT_SOURCE_DIR}/include" PARENT_SCOPE )
ENDIF(  )
################################################################################
