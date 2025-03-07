# Building DNG SDK
cmake_minimum_required(VERSION 3.15)

# Avoid warning about DOWNLOAD_EXTRACT_TIMESTAMP
if(CMAKE_VERSION VERSION_GREATER_EQUAL "3.24.0")
    cmake_policy(SET CMP0135 NEW)
endif()

# default to release build except Windows
set(CMAKE_BUILD_TYPE "Release" CACHE STRING "build type")

# Define project name and (optional) version
project(dng VERSION 1.7.1 DESCRIPTION "DNG SDK dynamic library" LANGUAGES C CXX)

# need >= C++11
set(CMAKE_CXX_STANDARD 14)

# CMake functionality
include(GNUInstallDirs)
include(FetchContent)
include(ExternalProject)
include(ProcessorCount)

# configs
if(WIN32)
    # Windows - force to OFF
    message("On Windows, dng_validate will not be built...")
    set(CMAKE_WINDOWS_EXPORT_ALL_SYMBOLS ON)
    set(BUILD_DNG_VALIDATE OFF CACHE BOOL "Build dng_validate command")
else()
    set(BUILD_DNG_VALIDATE ON CACHE BOOL "Build dng_validate command")
endif()

# main processing
FetchContent_Declare(
    dngsdk
    URL https://download.adobe.com/pub/adobe/dng/dng_sdk_1_7_1.zip
    URL_HASH MD5=b73ba6cc243d9617aa4371a0f6d76e4b
)
FetchContent_MakeAvailable(dngsdk)

# List source files for the library
set(DNG_VALIDATE_SRC "${FETCHCONTENT_BASE_DIR}/dngsdk-src/dng_sdk/source/dng_validate.cpp")
file(
    GLOB SRC_FILES
    ${FETCHCONTENT_BASE_DIR}/dngsdk-src/xmp/toolkit/source/XMP_ProgressTracker.cpp
    ${FETCHCONTENT_BASE_DIR}/dngsdk-src/xmp/toolkit/source/UnicodeConversions.cpp
    ${FETCHCONTENT_BASE_DIR}/dngsdk-src/xmp/toolkit/source/PerfUtils.cpp
    ${FETCHCONTENT_BASE_DIR}/dngsdk-src/xmp/toolkit/source/XML_Node.cpp
    ${FETCHCONTENT_BASE_DIR}/dngsdk-src/xmp/toolkit/source/XIO.cpp
    ${FETCHCONTENT_BASE_DIR}/dngsdk-src/xmp/toolkit/source/IOUtils.cpp
    ${FETCHCONTENT_BASE_DIR}/dngsdk-src/xmp/toolkit/source/XMPFiles_IO.cpp
    ${FETCHCONTENT_BASE_DIR}/dngsdk-src/xmp/toolkit/source/SafeStringAPIs.cpp
    ${FETCHCONTENT_BASE_DIR}/dngsdk-src/xmp/toolkit/source/XMP_LibUtils.cpp
    ${FETCHCONTENT_BASE_DIR}/dngsdk-src/xmp/toolkit/public/include/XMPCommon/source/*.cpp
    ${FETCHCONTENT_BASE_DIR}/dngsdk-src/xmp/toolkit/public/include/XMPCore/source/*.cpp
    ${FETCHCONTENT_BASE_DIR}/dngsdk-src/xmp/toolkit/third-party/zuid/sources/*.cpp
    ${FETCHCONTENT_BASE_DIR}/dngsdk-src/xmp/toolkit/XMPCommon/source/*.cpp
    ${FETCHCONTENT_BASE_DIR}/dngsdk-src/xmp/toolkit/XMPCore/source/DOMSerializerImpl.cpp
    ${FETCHCONTENT_BASE_DIR}/dngsdk-src/xmp/toolkit/XMPCore/source/IPathSegment_I.cpp
    ${FETCHCONTENT_BASE_DIR}/dngsdk-src/xmp/toolkit/XMPCore/source/INode_I.cpp
    ${FETCHCONTENT_BASE_DIR}/dngsdk-src/xmp/toolkit/XMPCore/source/WXMPMeta.cpp
    ${FETCHCONTENT_BASE_DIR}/dngsdk-src/xmp/toolkit/XMPCore/source/ClientDOMSerializerWrapperImpl.cpp
    ${FETCHCONTENT_BASE_DIR}/dngsdk-src/xmp/toolkit/XMPCore/source/XMPIterator2.cpp
    ${FETCHCONTENT_BASE_DIR}/dngsdk-src/xmp/toolkit/XMPCore/source/ICoreObjectFactory_I.cpp
    ${FETCHCONTENT_BASE_DIR}/dngsdk-src/xmp/toolkit/XMPCore/source/ArrayNodeImpl.cpp
    ${FETCHCONTENT_BASE_DIR}/dngsdk-src/xmp/toolkit/XMPCore/source/NameSpacePrefixMapImpl.cpp
    ${FETCHCONTENT_BASE_DIR}/dngsdk-src/xmp/toolkit/XMPCore/source/INameSpacePrefixMap_I.cpp
    ${FETCHCONTENT_BASE_DIR}/dngsdk-src/xmp/toolkit/XMPCore/source/IArrayNode_I.cpp
    ${FETCHCONTENT_BASE_DIR}/dngsdk-src/xmp/toolkit/XMPCore/source/WXMPDocOps.cpp
    ${FETCHCONTENT_BASE_DIR}/dngsdk-src/xmp/toolkit/XMPCore/source/CoreConfigurationManagerImpl.cpp
    ${FETCHCONTENT_BASE_DIR}/dngsdk-src/xmp/toolkit/XMPCore/source/XMPDocOps.cpp
    ${FETCHCONTENT_BASE_DIR}/dngsdk-src/xmp/toolkit/XMPCore/source/INodeIterator_I.cpp
    ${FETCHCONTENT_BASE_DIR}/dngsdk-src/xmp/toolkit/XMPCore/source/MetadataConverterUtilsImpl.cpp
    ${FETCHCONTENT_BASE_DIR}/dngsdk-src/xmp/toolkit/XMPCore/source/ICompositeNode_I.cpp
    ${FETCHCONTENT_BASE_DIR}/dngsdk-src/xmp/toolkit/XMPCore/source/XMPCore_Impl.cpp
    ${FETCHCONTENT_BASE_DIR}/dngsdk-src/xmp/toolkit/XMPCore/source/PathImpl.cpp
    ${FETCHCONTENT_BASE_DIR}/dngsdk-src/xmp/toolkit/XMPCore/source/ParseRDF.cpp
    ${FETCHCONTENT_BASE_DIR}/dngsdk-src/xmp/toolkit/XMPCore/source/ClientDOMParserWrapperImpl.cpp
    ${FETCHCONTENT_BASE_DIR}/dngsdk-src/xmp/toolkit/XMPCore/source/StructureNodeImpl.cpp
    ${FETCHCONTENT_BASE_DIR}/dngsdk-src/xmp/toolkit/XMPCore/source/ICoreConfigurationManager_I.cpp
    ${FETCHCONTENT_BASE_DIR}/dngsdk-src/xmp/toolkit/XMPCore/source/XMPUtils.cpp
    ${FETCHCONTENT_BASE_DIR}/dngsdk-src/xmp/toolkit/XMPCore/source/ISimpleNode_I.cpp
    ${FETCHCONTENT_BASE_DIR}/dngsdk-src/xmp/toolkit/XMPCore/source/IDOMParser_I.cpp
    ${FETCHCONTENT_BASE_DIR}/dngsdk-src/xmp/toolkit/XMPCore/source/XMPIterator.cpp
    ${FETCHCONTENT_BASE_DIR}/dngsdk-src/xmp/toolkit/XMPCore/source/IStructureNode_I.cpp
    ${FETCHCONTENT_BASE_DIR}/dngsdk-src/xmp/toolkit/XMPCore/source/XMPUtils2.cpp
    ${FETCHCONTENT_BASE_DIR}/dngsdk-src/xmp/toolkit/XMPCore/source/WXMPIterator.cpp
    ${FETCHCONTENT_BASE_DIR}/dngsdk-src/xmp/toolkit/XMPCore/source/IDOMImplementationRegistry_I.cpp
    ${FETCHCONTENT_BASE_DIR}/dngsdk-src/xmp/toolkit/XMPCore/source/IPath_I.cpp
    ${FETCHCONTENT_BASE_DIR}/dngsdk-src/xmp/toolkit/XMPCore/source/MetadataImpl.cpp
    ${FETCHCONTENT_BASE_DIR}/dngsdk-src/xmp/toolkit/XMPCore/source/XMPDocOps2.cpp
    ${FETCHCONTENT_BASE_DIR}/dngsdk-src/xmp/toolkit/XMPCore/source/IDOMSerializer_I.cpp
    ${FETCHCONTENT_BASE_DIR}/dngsdk-src/xmp/toolkit/XMPCore/source/PathSegmentImpl.cpp
    ${FETCHCONTENT_BASE_DIR}/dngsdk-src/xmp/toolkit/XMPCore/source/RDFDOMParserImpl.cpp
    ${FETCHCONTENT_BASE_DIR}/dngsdk-src/xmp/toolkit/XMPCore/source/ExpatAdapter.cpp
    ${FETCHCONTENT_BASE_DIR}/dngsdk-src/xmp/toolkit/XMPCore/source/DOMImplementationRegistryImpl.cpp
    ${FETCHCONTENT_BASE_DIR}/dngsdk-src/xmp/toolkit/XMPCore/source/IMetadata_I.cpp
    ${FETCHCONTENT_BASE_DIR}/dngsdk-src/xmp/toolkit/XMPCore/source/RDFDOMSerializerImpl.cpp
    ${FETCHCONTENT_BASE_DIR}/dngsdk-src/xmp/toolkit/XMPCore/source/DOMParserImpl.cpp
    ${FETCHCONTENT_BASE_DIR}/dngsdk-src/xmp/toolkit/XMPCore/source/CompositeNodeImpl.cpp
    ${FETCHCONTENT_BASE_DIR}/dngsdk-src/xmp/toolkit/XMPCore/source/XMPMeta.cpp
    ${FETCHCONTENT_BASE_DIR}/dngsdk-src/xmp/toolkit/XMPCore/source/CoreObjectFactoryImpl.cpp
    ${FETCHCONTENT_BASE_DIR}/dngsdk-src/xmp/toolkit/XMPCore/source/SimpleNodeImpl.cpp
    ${FETCHCONTENT_BASE_DIR}/dngsdk-src/xmp/toolkit/XMPCore/source/WXMPUtils.cpp
    ${FETCHCONTENT_BASE_DIR}/dngsdk-src/xmp/toolkit/XMPCore/source/NodeImpl.cpp
    ${FETCHCONTENT_BASE_DIR}/dngsdk-src/xmp/toolkit/XMPCore/source/IMetadataConverterUtils_I.cpp
    ${FETCHCONTENT_BASE_DIR}/dngsdk-src/xmp/toolkit/XMPCore/source/XMPMeta-Parse.cpp
    ${FETCHCONTENT_BASE_DIR}/dngsdk-src/xmp/toolkit/XMPCore/source/XMPDocOps-Utils.cpp
    ${FETCHCONTENT_BASE_DIR}/dngsdk-src/xmp/toolkit/XMPCore/source/XMPMeta-Serialize.cpp
    ${FETCHCONTENT_BASE_DIR}/dngsdk-src/xmp/toolkit/XMPCore/source/XMPMeta-GetSet.cpp
    ${FETCHCONTENT_BASE_DIR}/dngsdk-src/xmp/toolkit/XMPCore/source/XMPMeta2-GetSet.cpp
    ${FETCHCONTENT_BASE_DIR}/dngsdk-src/xmp/toolkit/XMPCore/source/XMPUtils-FileInfo.cpp
    ${FETCHCONTENT_BASE_DIR}/dngsdk-src/xmp/toolkit/XMPCore/third-party/expat/public/lib/*.c
    ${FETCHCONTENT_BASE_DIR}/dngsdk-src/xmp/toolkit/XMPFiles/source/*.cpp
    ${FETCHCONTENT_BASE_DIR}/dngsdk-src/xmp/toolkit/XMPFiles/source/FileHandlers/*.cpp
    ${FETCHCONTENT_BASE_DIR}/dngsdk-src/xmp/toolkit/XMPFiles/source/FormatSupport/*.cpp
    ${FETCHCONTENT_BASE_DIR}/dngsdk-src/xmp/toolkit/XMPFiles/source/FormatSupport/AIFF/*.cpp
    ${FETCHCONTENT_BASE_DIR}/dngsdk-src/xmp/toolkit/XMPFiles/source/FormatSupport/IFF/*.cpp
    ${FETCHCONTENT_BASE_DIR}/dngsdk-src/xmp/toolkit/XMPFiles/source/FormatSupport/WAVE/*.cpp
    ${FETCHCONTENT_BASE_DIR}/dngsdk-src/xmp/toolkit/XMPFiles/source/FormatSupport/WebP/*.cpp
    ${FETCHCONTENT_BASE_DIR}/dngsdk-src/xmp/toolkit/XMPFiles/source/NativeMetadataSupport/*.cpp
    ${FETCHCONTENT_BASE_DIR}/dngsdk-src/xmp/toolkit/XMPFiles/source/PluginHandler/FileHandlerInstance.cpp
    ${FETCHCONTENT_BASE_DIR}/dngsdk-src/xmp/toolkit/XMPFiles/source/PluginHandler/HostAPIImpl.cpp
    ${FETCHCONTENT_BASE_DIR}/dngsdk-src/xmp/toolkit/XMPFiles/source/PluginHandler/Module.cpp
    ${FETCHCONTENT_BASE_DIR}/dngsdk-src/xmp/toolkit/XMPFiles/source/PluginHandler/PluginManager.cpp
    ${FETCHCONTENT_BASE_DIR}/dngsdk-src/xmp/toolkit/XMPFiles/source/PluginHandler/XMPAtoms.cpp
    ${FETCHCONTENT_BASE_DIR}/dngsdk-src/xmp/toolkit/XMPFiles/source/WXMPFiles.cpp
    ${FETCHCONTENT_BASE_DIR}/dngsdk-src/xmp/toolkit/public/include/client-glue/*.incl_cpp
    ${FETCHCONTENT_BASE_DIR}/dngsdk-src/dng_sdk/source/*.cpp
)

# remove source of dng_validate
list(REMOVE_ITEM SRC_FILES DNG_VALIDATE_SRC)
# add platform-specific source files
if(APPLE)
    list(
        APPEND SRC_FILES
        ${FETCHCONTENT_BASE_DIR}/dngsdk-src/xmp/toolkit/XMPFiles/source/PluginHandler/OS_Utils_Mac.cpp
        ${FETCHCONTENT_BASE_DIR}/dngsdk-src/xmp/toolkit/source/Host_IO-POSIX.cpp
    )
elseif(WIN32)
    list(
        APPEND SRC_FILES
        ${FETCHCONTENT_BASE_DIR}/dngsdk-src/xmp/toolkit/XMPFiles/source/PluginHandler/OS_Utils_WIN.cpp
        ${FETCHCONTENT_BASE_DIR}/dngsdk-src/xmp/toolkit/source/Host_IO-Win.cpp
        # Windows - add source files directly
        ${FETCHCONTENT_BASE_DIR}/dngsdk-src/libjpeg/jaricom.c
        ${FETCHCONTENT_BASE_DIR}/dngsdk-src/libjpeg/jcapimin.c
        ${FETCHCONTENT_BASE_DIR}/dngsdk-src/libjpeg/jcapistd.c
        ${FETCHCONTENT_BASE_DIR}/dngsdk-src/libjpeg/jcarith.c
        ${FETCHCONTENT_BASE_DIR}/dngsdk-src/libjpeg/jccoefct.c
        ${FETCHCONTENT_BASE_DIR}/dngsdk-src/libjpeg/jccolor.c
        ${FETCHCONTENT_BASE_DIR}/dngsdk-src/libjpeg/jcdctmgr.c
        ${FETCHCONTENT_BASE_DIR}/dngsdk-src/libjpeg/jchuff.c
        ${FETCHCONTENT_BASE_DIR}/dngsdk-src/libjpeg/jcinit.c
        ${FETCHCONTENT_BASE_DIR}/dngsdk-src/libjpeg/jcmainct.c
        ${FETCHCONTENT_BASE_DIR}/dngsdk-src/libjpeg/jcmarker.c
        ${FETCHCONTENT_BASE_DIR}/dngsdk-src/libjpeg/jcmaster.c
        ${FETCHCONTENT_BASE_DIR}/dngsdk-src/libjpeg/jcomapi.c
        ${FETCHCONTENT_BASE_DIR}/dngsdk-src/libjpeg/jcparam.c
        ${FETCHCONTENT_BASE_DIR}/dngsdk-src/libjpeg/jcprepct.c
        ${FETCHCONTENT_BASE_DIR}/dngsdk-src/libjpeg/jcsample.c
        ${FETCHCONTENT_BASE_DIR}/dngsdk-src/libjpeg/jctrans.c
        ${FETCHCONTENT_BASE_DIR}/dngsdk-src/libjpeg/jdapimin.c
        ${FETCHCONTENT_BASE_DIR}/dngsdk-src/libjpeg/jdapistd.c
        ${FETCHCONTENT_BASE_DIR}/dngsdk-src/libjpeg/jdarith.c
        ${FETCHCONTENT_BASE_DIR}/dngsdk-src/libjpeg/jdatadst.c
        ${FETCHCONTENT_BASE_DIR}/dngsdk-src/libjpeg/jdatasrc.c
        ${FETCHCONTENT_BASE_DIR}/dngsdk-src/libjpeg/jdcoefct.c
        ${FETCHCONTENT_BASE_DIR}/dngsdk-src/libjpeg/jdcolor.c
        ${FETCHCONTENT_BASE_DIR}/dngsdk-src/libjpeg/jddctmgr.c
        ${FETCHCONTENT_BASE_DIR}/dngsdk-src/libjpeg/jdhuff.c
        ${FETCHCONTENT_BASE_DIR}/dngsdk-src/libjpeg/jdinput.c
        ${FETCHCONTENT_BASE_DIR}/dngsdk-src/libjpeg/jdmainct.c
        ${FETCHCONTENT_BASE_DIR}/dngsdk-src/libjpeg/jdmarker.c
        ${FETCHCONTENT_BASE_DIR}/dngsdk-src/libjpeg/jdmaster.c
        ${FETCHCONTENT_BASE_DIR}/dngsdk-src/libjpeg/jdmerge.c
        ${FETCHCONTENT_BASE_DIR}/dngsdk-src/libjpeg/jdpostct.c
        ${FETCHCONTENT_BASE_DIR}/dngsdk-src/libjpeg/jdsample.c
        ${FETCHCONTENT_BASE_DIR}/dngsdk-src/libjpeg/jdtrans.c
        ${FETCHCONTENT_BASE_DIR}/dngsdk-src/libjpeg/jerror.c
        ${FETCHCONTENT_BASE_DIR}/dngsdk-src/libjpeg/jfdctflt.c
        ${FETCHCONTENT_BASE_DIR}/dngsdk-src/libjpeg/jfdctfst.c
        ${FETCHCONTENT_BASE_DIR}/dngsdk-src/libjpeg/jfdctint.c
        ${FETCHCONTENT_BASE_DIR}/dngsdk-src/libjpeg/jidctflt.c
        ${FETCHCONTENT_BASE_DIR}/dngsdk-src/libjpeg/jidctfst.c
        ${FETCHCONTENT_BASE_DIR}/dngsdk-src/libjpeg/jidctint.c
        ${FETCHCONTENT_BASE_DIR}/dngsdk-src/libjpeg/jmemmgr.c
        ${FETCHCONTENT_BASE_DIR}/dngsdk-src/libjpeg/jmemnobs.c
        ${FETCHCONTENT_BASE_DIR}/dngsdk-src/libjpeg/jquant1.c
        ${FETCHCONTENT_BASE_DIR}/dngsdk-src/libjpeg/jquant2.c
        ${FETCHCONTENT_BASE_DIR}/dngsdk-src/libjpeg/jutils.c
    )
else()  # UNIX/Linux
    list(
        APPEND SRC_FILES
        ${FETCHCONTENT_BASE_DIR}/dngsdk-src/xmp/toolkit/XMPFiles/source/PluginHandler/OS_Utils_Linux.cpp
        ${FETCHCONTENT_BASE_DIR}/dngsdk-src/xmp/toolkit/source/Host_IO-POSIX.cpp
    )
endif()

# Create the shared library
add_library(dng SHARED ${SRC_FILES})

# ----- dependencies
# zlib
include(FindZLIB)
if(ZLIB_FOUND)
    message("found zlib")
    # message("using zlib ver. ${ZLIB_VERSION}")
else()
    message(SEND_ERROR "zlib not found...")
endif()

# libjpeg
if(WIN32)
    target_include_directories(dng PRIVATE ${FETCHCONTENT_BASE_DIR}/dngsdk-src/libjpeg)   
else()
    ProcessorCount(N) # for parallel execution
    ExternalProject_Add(
        libjpeg
        SOURCE_DIR ${FETCHCONTENT_BASE_DIR}/dngsdk-src/libjpeg
        CONFIGURE_COMMAND ${FETCHCONTENT_BASE_DIR}/dngsdk-src/libjpeg/configure --prefix=${CMAKE_INSTALL_PREFIX}
        BUILD_COMMAND make -j${N}
    )
endif()

# libjxl
ExternalProject_Add(
    libjxl
    SOURCE_DIR ${FETCHCONTENT_BASE_DIR}/dngsdk-src/libjxl/libjxl
    CMAKE_ARGS -DCMAKE_INSTALL_PREFIX=${CMAKE_INSTALL_PREFIX} -DCMAKE_BUILD_TYPE=${CMAKE_BUILD_TYPE} -DBUILD_SHARED_LIBS=ON -DBUILD_TESTING=OFF -DJPEGXL_ENABLE_SJPEG=OFF -DJPEGXL_ENABLE_EXAMPLES=OFF -DJPEGXL_ENABLE_TOOLS=OFF -DJPEGXL_ENABLE_BENCHMARK=OFF -DJPEGXL_ENABLE_MANPAGES=OFF -DJPEGXL_ENABLE_JPEGLI=OFF -DBROTLI_BUNDLED_MODE=ON -DHWY_ENABLE_INSTALL=OFF -DHWY_ENABLE_TESTS=OFF 
)

# set library version information
set_target_properties(
    dng PROPERTIES
    VERSION ${PROJECT_VERSION}
    SOVERSION ${PROJECT_VERSION_MAJOR}
)

# common setting
if(NOT WIN32)
    add_dependencies(dng libjpeg)
endif()
add_dependencies(dng libjxl)
target_compile_definitions(
    dng PRIVATE
    $<IF:$<CONFIG:Debug>,qDNGDebug=1,qDNGDebug=0>
    qDNGValidate=1 qDNGUseXMP=1 qDNGUseLibJPEG=1 AdobePrivate=1
    BUILDING_XMPCORE_LIB=1 BUILDING_XMPCORE_AS_STATIC=1 XMP_StaticBuild=1 XMP_COMPONENT_INT_NAMESPACE=AdobeXMPCore_Int
)

if(${BUILD_DNG_VALIDATE})
    target_compile_definitions(dng PUBLIC qDNGValidateTarget=1)
else()
    target_compile_definitions(dng PUBLIC qDNGValidateTarget=0)
endif()

# platform-specific donfigs
if(APPLE)
    target_compile_definitions(dng PUBLIC qMacOS=1 MAC_ENV XMP_64=1)
    target_link_libraries(dng PRIVATE "-framework CoreServices")
elseif(WIN32)
    target_compile_definitions(qWinOS=1 WIN_ENV NOMINMAX UNICODE)
else()  # UNIX/Linux
    target_compile_options(dng PUBLIC -fPIC -Wall -Wextra)
    target_compile_definitions(dng PUBLIC qLinux UNIX_ENV XMP_64=1)
endif()

# Set include directory for headers
target_include_directories(
    dng PRIVATE
    ${ZLIB_INCLUDE_DIRS}
    ${CMAKE_INSTALL_PREFIX}/include
    ${FETCHCONTENT_BASE_DIR}/dngsdk-src/xmp/toolkit
    ${FETCHCONTENT_BASE_DIR}/dngsdk-src/xmp/toolkit/public/include
    ${FETCHCONTENT_BASE_DIR}/dngsdk-src/xmp/toolkit/XMPFilesPlugins/api/source
    ${FETCHCONTENT_BASE_DIR}/dngsdk-src/xmp/toolkit/XMPCore/third-party/expat/public/lib
    ${FETCHCONTENT_BASE_DIR}/dngsdk-src/xmp/toolkit/XMPCore/third-party/boost
    ${FETCHCONTENT_BASE_DIR}/dngsdk-src/xmp/toolkit/public/include/XMPCore/Interfaces
    ${FETCHCONTENT_BASE_DIR}/dngsdk-src/xmp/toolkit/public/include/client-glue
)

target_link_directories(dng PRIVATE ${CMAKE_INSTALL_PREFIX}/lib)
if(NOT WIN32)
    target_link_libraries(dng PRIVATE jpeg)
endif()
target_link_libraries(dng PRIVATE ${ZLIB_LIBRARIES} jxl jxl_threads)

if(BUILD_DNG_VALIDATE)
    # build dng_validate command
    message("build dng_validate...")
    add_executable(dng_validate ${DNG_VALIDATE_SRC})
    target_compile_definitions(dng_validate PRIVATE qDNGValidateTarget=1 qDNGUseXMP=1)
    target_include_directories(
        dng_validate PRIVATE 
        ${FETCHCONTENT_BASE_DIR}/dngsdk-src/dng_sdk/source
        ${FETCHCONTENT_BASE_DIR}/dngsdk-src/libjxl/libjxl/lib/include
    )
    target_link_libraries(dng_validate PRIVATE dng)
endif(BUILD_DNG_VALIDATE)

# install header & library files (& dng_validate command)
install(
    DIRECTORY ${FETCHCONTENT_BASE_DIR}/dngsdk-src/dng_sdk/source/
    DESTINATION include/dng
    FILES_MATCHING PATTERN "*.h"
)
install(TARGETS dng LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR})
if(BUILD_DNG_VALIDATE)
    install(TARGETS dng_validate DESTINATION ${CMAKE_INSTALL_BINDIR})
endif(BUILD_DNG_VALIDATE)
