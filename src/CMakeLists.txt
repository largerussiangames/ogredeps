# provide configuration options to choose which of the dependencies to build
if(WIN32)
  option(OGREDEPS_BUILD_CG "Build Cg dependency" TRUE)
endif()
option(OGREDEPS_BUILD_FREEIMAGE "Build FreeImage dependency" TRUE)
option(OGREDEPS_BUILD_FREETYPE "Build FreeType dependency" TRUE)
option(OGREDEPS_BUILD_OIS "Build OIS dependency" TRUE)
option(OGREDEPS_BUILD_ZLIB "Build zlib dependency" TRUE)
option(OGREDEPS_BUILD_ZZIPLIB "Build zziplib dependency" TRUE)
option(OGREDEPS_INSTALL_DEV "Install development headers and libraries" TRUE)
set(OGREDEPS_LIB_DIRECTORY "lib${LIB_SUFFIX}" CACHE STRING "Install path for libraries, e.g. 'lib64' on some 64-bit Linux distros.")
mark_as_advanced(OGREDEPS_LIB_DIRECTORY)

# choose paths to build and install dependencies to
if (NOT OGREDEPS_RUNTIME_OUTPUT)
  set(OGREDEPS_RUNTIME_OUTPUT ${OGREDEPS_BINARY_DIR}/bin)
endif ()
if (WIN32)
  set(OGREDEPS_RELEASE_PATH "/Release")
  set(OGREDEPS_DEBUG_PATH "/Debug")
else ()
  set(OGREDEPS_RELEASE_PATH "")
  set(OGREDEPS_DEBUG_PATH "")
endif ()

# postfix debug libraries with a _d
if (NOT APPLE)
set(CMAKE_DEBUG_POSTFIX "_d")
endif()

# determine if we are compiling for a 32bit or 64bit system
include(CheckTypeSize)
CHECK_TYPE_SIZE("void*" OGREDEPS_PTR_SIZE BUILTIN_TYPES_ONLY)
if (OGREDEPS_PTR_SIZE EQUAL 8)
  set(OGREDEPS_PLATFORM_X64 TRUE)
else ()
  set(OGREDEPS_PLATFORM_X64 FALSE)
endif ()

# Add -fPIC flag to x64 GCC builds
if (CMAKE_COMPILER_IS_GNUCXX AND NOT MINGW)
  if (OGREDEPS_PLATFORM_X64 AND NOT APPLE)
    add_definitions(-fPIC)
  endif ()
endif ()


# by default, install to ogredeps subfolder in build directory
if (CMAKE_INSTALL_PREFIX_INITIALIZED_TO_DEFAULT)
  if (OGREDEPS_SOURCE_DIR STREQUAL CMAKE_SOURCE_DIR)
      set(CMAKE_INSTALL_PREFIX "${OGREDEPS_BINARY_DIR}/ogredeps" CACHE PATH "OGREDEPS install prefix" FORCE)
  endif ()
endif ()

# enforce that CMAKE_BUILD_TYPE is set, otherwise install commands
# will not work for Makefile based generators.
if (NOT CMAKE_BUILD_TYPE)
  set(CMAKE_BUILD_TYPE Release CACHE STRING "Choose the type of build, options are: Debug, Release" FORCE)
endif ()

# configure and install dependency LIBRARY
# INCLUDE_DIR specifies where header files need to be installed
# Any additional parameters are treated as a list of header files
# to install.
macro(install_dep LIBRARY INCLUDE_DIR)
  set_target_properties(${LIBRARY} PROPERTIES
    RUNTIME_OUTPUT_DIRECTORY ${OGREDEPS_RUNTIME_OUTPUT}
  )

  if(APPLE AND OGRE_BUILD_PLATFORM_APPLE_IOS)
    add_custom_command(TARGET ${LIBRARY} POST_BUILD
      COMMAND if [[ -e ${OGREDEPS_BINARY_DIR}/src/${LIBRARY}/$(CONFIGURATION)-iphonesimulator/lib${LIBRARY}.a && -e ${OGREDEPS_BINARY_DIR}/src/${LIBRARY}/$(CONFIGURATION)-iphoneos/lib${LIBRARY}.a ]] \; then lipo ARGS -create ${OGREDEPS_BINARY_DIR}/src/${LIBRARY}/$(CONFIGURATION)-iphoneos/lib${LIBRARY}.a -arch i386 ${OGREDEPS_BINARY_DIR}/src/${LIBRARY}/$(CONFIGURATION)-iphonesimulator/lib${LIBRARY}.a
      -output "${OGREDEPS_BINARY_DIR}/../${OGREDEPS_LIB_DIRECTORY}/lib${LIBRARY}.a"\; fi
    )
  endif()

  # always install runtime libraries (i.e. DLL, so)
  install(TARGETS ${LIBRARY}
    RUNTIME DESTINATION "bin${OGREDEPS_RELEASE_PATH}" CONFIGURATIONS Release
    LIBRARY DESTINATION "${OGREDEPS_LIB_DIRECTORY}${OGREDEPS_RELEASE_PATH}" CONFIGURATIONS Release
    ARCHIVE DESTINATION "${OGREDEPS_BINARY_DIR}/notwanted" CONFIGURATIONS Release
  )
  install(TARGETS ${LIBRARY}
    RUNTIME DESTINATION "bin${OGREDEPS_DEBUG_PATH}" CONFIGURATIONS Debug
    LIBRARY DESTINATION "${OGREDEPS_LIB_DIRECTORY}${OGREDEPS_DEBUG_PATH}" CONFIGURATIONS Debug
    ARCHIVE DESTINATION "${OGREDEPS_BINARY_DIR}/notwanted" CONFIGURATIONS Debug
  )

  if (OGREDEPS_INSTALL_DEV)
    # also install development parts
    install(TARGETS ${LIBRARY}
      RUNTIME DESTINATION "bin${OGREDEPS_RELEASE_PATH}" CONFIGURATIONS Release
      LIBRARY DESTINATION "${OGREDEPS_LIB_DIRECTORY}${OGREDEPS_RELEASE_PATH}" CONFIGURATIONS Release
      ARCHIVE DESTINATION "${OGREDEPS_LIB_DIRECTORY}${OGREDEPS_RELEASE_PATH}" CONFIGURATIONS Release
    )
    install(TARGETS ${LIBRARY}
      RUNTIME DESTINATION "bin${OGREDEPS_DEBUG_PATH}" CONFIGURATIONS Debug
      LIBRARY DESTINATION "${OGREDEPS_LIB_DIRECTORY}${OGREDEPS_DEBUG_PATH}" CONFIGURATIONS Debug
      ARCHIVE DESTINATION "${OGREDEPS_LIB_DIRECTORY}${OGREDEPS_DEBUG_PATH}" CONFIGURATIONS Debug
    )
    install(FILES ${ARGN} DESTINATION ${INCLUDE_DIR})
  endif ()
endmacro()

# setup the individual dependencies
if(NOT OGRE_BUILD_PLATFORM_APPLE_IOS)
  if (OGREDEPS_BUILD_CG)
    add_subdirectory(Cg)
  endif ()
endif ()
if (OGREDEPS_BUILD_FREEIMAGE)
  add_subdirectory(FreeImage)
endif ()
if (OGREDEPS_BUILD_FREETYPE)
  add_subdirectory(freetype)
endif ()
if (OGREDEPS_BUILD_OIS)
  add_subdirectory(ois)
endif ()
if (OGREDEPS_BUILD_ZLIB)
  add_subdirectory(zlib)
endif ()
if (OGREDEPS_BUILD_ZZIPLIB)
  add_subdirectory(zziplib)
endif ()
