
# Anonymous-Cellphones-Project-Protoype-1.0-1.0 #

None (for now at least, but that is only in the present. In the future, there will be much much more.)

# README.md #

# Project setup #
cmake_minimum_required(VERSION 2.6)
project(gr-amps CXX C)
enable_testing()

# Select the release build type by default to get optimization flags #
if(NOT CMAKE_BUILD_TYPE)
   set(CMAKE_BUILD_TYPE "Release")
   message(STATUS "Build type not specified: defaulting to release.")
endif(NOT CMAKE_BUILD_TYPE)
set(CMAKE_BUILD_TYPE ${CMAKE_BUILD_TYPE} CACHE STRING "")

# Make sure our local CMake Modules path comes first #
list(INSERT CMAKE_MODULE_PATH 0 ${CMAKE_SOURCE_DIR}/cmake/Modules)

# Compiler specific setup #
if(CMAKE_COMPILER_IS_GNUCXX AND NOT WIN32)
    #http://gcc.gnu.org/wiki/Visibility
    add_definitions(-fvisibility=hidden)
endif()

# Find boost #
if(UNIX AND EXISTS "/usr/lib64")
    list(APPEND BOOST_LIBRARYDIR "/usr/lib64") #fedora 64-bit fix
endif(UNIX AND EXISTS "/usr/lib64")
set(Boost_ADDITIONAL_VERSIONS
    "1.35.0" "1.35" "1.36.0" "1.36" "1.37.0" "1.37" "1.38.0" "1.38" "1.39.0" "1.39"
    "1.40.0" "1.40" "1.41.0" "1.41" "1.42.0" "1.42" "1.43.0" "1.43" "1.44.0" "1.44"
    "1.45.0" "1.45" "1.46.0" "1.46" "1.47.0" "1.47" "1.48.0" "1.48" "1.49.0" "1.49"
    "1.50.0" "1.50" "1.51.0" "1.51" "1.52.0" "1.52" "1.53.0" "1.53" "1.54.0" "1.54"
    "1.55.0" "1.55" "1.56.0" "1.56" "1.57.0" "1.57" "1.58.0" "1.58" "1.59.0" "1.59"
    "1.60.0" "1.60" "1.61.0" "1.61" "1.62.0" "1.62" "1.63.0" "1.63" "1.64.0" "1.64"
    "1.65.0" "1.65" "1.66.0" "1.66" "1.67.0" "1.67" "1.68.0" "1.68" "1.69.0" "1.69"
)
find_package(Boost "1.35" COMPONENTS filesystem system)
if(NOT Boost_FOUND)
    message(FATAL_ERROR "Boost required to compile amps")
endif()

# Install directories #
include(GrPlatform) #define LIB_SUFFIX
set(GR_RUNTIME_DIR      bin)
set(GR_LIBRARY_DIR      lib${LIB_SUFFIX})
set(GR_INCLUDE_DIR      include/amps)
set(GR_DATA_DIR         share)
set(GR_PKG_DATA_DIR     ${GR_DATA_DIR}/${CMAKE_PROJECT_NAME})
set(GR_DOC_DIR          ${GR_DATA_DIR}/doc)
set(GR_PKG_DOC_DIR      ${GR_DOC_DIR}/${CMAKE_PROJECT_NAME})
set(GR_CONF_DIR         etc)
set(GR_PKG_CONF_DIR     ${GR_CONF_DIR}/${CMAKE_PROJECT_NAME}/conf.d)
set(GR_LIBEXEC_DIR      libexec)
set(GR_PKG_LIBEXEC_DIR  ${GR_LIBEXEC_DIR}/${CMAKE_PROJECT_NAME})
set(GRC_BLOCKS_DIR      ${GR_PKG_DATA_DIR}/grc/blocks)

# On Apple only, set install name and use rpath correctly, if not already set #
if(APPLE)
    if(NOT CMAKE_INSTALL_NAME_DIR)
        set(CMAKE_INSTALL_NAME_DIR
            ${CMAKE_INSTALL_PREFIX}/${GR_LIBRARY_DIR} CACHE
            PATH "Library Install Name Destination Directory" FORCE)
    endif(NOT CMAKE_INSTALL_NAME_DIR)
    if(NOT CMAKE_INSTALL_RPATH)
        set(CMAKE_INSTALL_RPATH
            ${CMAKE_INSTALL_PREFIX}/${GR_LIBRARY_DIR} CACHE
            PATH "Library Install RPath" FORCE)
    endif(NOT CMAKE_INSTALL_RPATH)
    if(NOT CMAKE_BUILD_WITH_INSTALL_RPATH)
        set(CMAKE_BUILD_WITH_INSTALL_RPATH ON CACHE
            BOOL "Do Build Using Library Install RPath" FORCE)
    endif(NOT CMAKE_BUILD_WITH_INSTALL_RPATH)
endif(APPLE)

# Find gnuradio build dependencies #
find_package(CppUnit)
find_package(Doxygen)
find_package(ITPP)

# Search for GNU Radio and its components and versions. Add any #
# components required to the list of GR_REQUIRED_COMPONENTS (in all #
# caps such as FILTER or FFT) and change the version to the minimum #
# API compatible version required. #
set(GR_REQUIRED_COMPONENTS RUNTIME)
find_package(Gnuradio "3.7.2" REQUIRED)
list(INSERT CMAKE_MODULE_PATH 0 ${CMAKE_SOURCE_DIR}/cmake/Modules)

if(NOT CPPUNIT_FOUND)
    message(FATAL_ERROR "CppUnit required to compile amps")
endif()

# Setup doxygen option #
if(DOXYGEN_FOUND)
	option(ENABLE_DOXYGEN "Build docs using Doxygen" ON)
else(DOXYGEN_FOUND)
	option(ENABLE_DOXYGEN "Build docs using Doxygen" OFF)
endif(DOXYGEN_FOUND)

# Setup the include and linker paths #
include_directories(
    ${CMAKE_SOURCE_DIR}/lib
    ${CMAKE_SOURCE_DIR}/include
    ${CMAKE_BINARY_DIR}/lib
    ${CMAKE_BINARY_DIR}/include
    ${Boost_INCLUDE_DIRS}
    ${ITPP_INCLUDE_DIR}
    ${CPPUNIT_INCLUDE_DIRS}
    ${GNURADIO_ALL_INCLUDE_DIRS}
)
link_directories(
    ${Boost_LIBRARY_DIRS}
    ${CPPUNIT_LIBRARY_DIRS}
    ${GNURADIO_RUNTIME_LIBRARY_DIRS}
)

# Set component parameters #
set(GR_AMPS_INCLUDE_DIRS ${CMAKE_CURRENT_SOURCE_DIR}/include CACHE INTERNAL "" FORCE)
set(GR_AMPS_SWIG_INCLUDE_DIRS ${CMAKE_CURRENT_SOURCE_DIR}/swig CACHE INTERNAL "" FORCE)

# Create uninstall target #
configure_file(
    ${CMAKE_SOURCE_DIR}/cmake/cmake_uninstall.cmake.in
    ${CMAKE_CURRENT_BINARY_DIR}/cmake_uninstall.cmake
    	       @ONLY)
add_custom_target(uninstall
    ${CMAKE_COMMAND} -P ${CMAKE_CURRENT_BINARY_DIR}/cmake_uninstall.cmake
)

# Add subdirectories #
add_subdirectory(include/amps)
add_subdirectory(lib)
add_subdirectory(swig)
add_subdirectory(python)
add_subdirectory(grc)
add_subdirectory(apps)
add_subdirectory(docs)

# Install cmake search helper for this library #
if(NOT CMAKE_MODULES_DIR)
  set(CMAKE_MODULES_DIR lib${LIB_SUFFIX}/cmake)
endif(NOT CMAKE_MODULES_DIR)
install(FILES cmake/Modules/ampsConfig.cmake
    DESTINATION ${CMAKE_MODULES_DIR}/amps
)
