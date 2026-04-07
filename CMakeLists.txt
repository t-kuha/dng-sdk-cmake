# Building DNG SDK
cmake_minimum_required(VERSION 3.15)

# Avoid warning about DOWNLOAD_EXTRACT_TIMESTAMP
if(CMAKE_VERSION VERSION_GREATER_EQUAL "3.24.0")
    cmake_policy(SET CMP0135 NEW)
endif()

project(dng VERSION 1.7.1 DESCRIPTION "DNG SDK dynamic library" LANGUAGES C CXX)

# --- Configuration
set(CMAKE_BUILD_TYPE "Release" CACHE STRING "Build type")
set(BUILD_DNG_VALIDATE ON CACHE BOOL "Build dng_validate command")
set(CMAKE_CXX_STANDARD 14) # need >= C++11

# RPATH settings
set(CMAKE_SKIP_BUILD_RPATH FALSE)
set(CMAKE_INSTALL_RPATH "${CMAKE_INSTALL_PREFIX}/lib")

include(GNUInstallDirs)
include(FetchContent)
include(ExternalProject)
include(ProcessorCount)
include(FindZLIB)

if(WIN32)
    # Windows: FIXME
    set(CMAKE_WINDOWS_EXPORT_ALL_SYMBOLS ON)    
endif()

# --- Dependencies Fetch
FetchContent_Declare(
    dngsdk
    URL https://download.adobe.com/pub/adobe/dng/dng_sdk_1_7_1_2502_20260303.zip
    URL_HASH MD5=8bc3809ea811fedef0906911350aa0cd
)
FetchContent_MakeAvailable(dngsdk)

set(SDK_ROOT "${FETCHCONTENT_BASE_DIR}/dngsdk-src")
set(DNG_VALIDATE_SRC "${SDK_ROOT}/dng_sdk/source/dng_validate.cpp")

# --- Source File Discovery
# Consolidate the massive list of XMPCore source files
set(XMP_CORE_SRC
    DOMSerializerImpl.cpp IPathSegment_I.cpp INode_I.cpp WXMPMeta.cpp ClientDOMSerializerWrapperImpl.cpp 
    XMPIterator2.cpp ICoreObjectFactory_I.cpp ArrayNodeImpl.cpp NameSpacePrefixMapImpl.cpp INameSpacePrefixMap_I.cpp
    IArrayNode_I.cpp WXMPDocOps.cpp CoreConfigurationManagerImpl.cpp XMPDocOps.cpp INodeIterator_I.cpp 
    MetadataConverterUtilsImpl.cpp ICompositeNode_I.cpp XMPCore_Impl.cpp PathImpl.cpp ParseRDF.cpp 
    ClientDOMParserWrapperImpl.cpp StructureNodeImpl.cpp ICoreConfigurationManager_I.cpp XMPUtils.cpp 
    ISimpleNode_I.cpp IDOMParser_I.cpp XMPIterator.cpp IStructureNode_I.cpp XMPUtils2.cpp WXMPIterator.cpp 
    IDOMImplementationRegistry_I.cpp IPath_I.cpp MetadataImpl.cpp XMPDocOps2.cpp IDOMSerializer_I.cpp 
    PathSegmentImpl.cpp RDFDOMParserImpl.cpp ExpatAdapter.cpp DOMImplementationRegistryImpl.cpp IMetadata_I.cpp 
    RDFDOMSerializerImpl.cpp DOMParserImpl.cpp CompositeNodeImpl.cpp XMPMeta.cpp CoreObjectFactoryImpl.cpp 
    SimpleNodeImpl.cpp WXMPUtils.cpp NodeImpl.cpp IMetadataConverterUtils_I.cpp XMPMeta-Parse.cpp 
    XMPDocOps-Utils.cpp XMPMeta-Serialize.cpp XMPMeta-GetSet.cpp XMPMeta2-GetSet.cpp XMPUtils-FileInfo.cpp
)

file(GLOB SRC_FILES
    "${SDK_ROOT}/xmp/toolkit/source/XMP_ProgressTracker.cpp"
    "${SDK_ROOT}/xmp/toolkit/source/UnicodeConversions.cpp"
    "${SDK_ROOT}/xmp/toolkit/source/PerfUtils.cpp"
    "${SDK_ROOT}/xmp/toolkit/source/XML_Node.cpp"
    "${SDK_ROOT}/xmp/toolkit/source/XIO.cpp"
    "${SDK_ROOT}/xmp/toolkit/source/IOUtils.cpp"
    "${SDK_ROOT}/xmp/toolkit/source/XMPFiles_IO.cpp"
    "${SDK_ROOT}/xmp/toolkit/source/SafeStringAPIs.cpp"
    "${SDK_ROOT}/xmp/toolkit/source/XMP_LibUtils.cpp"
    "${SDK_ROOT}/xmp/toolkit/public/include/XMPCommon/source/*.cpp"
    "${SDK_ROOT}/xmp/toolkit/public/include/XMPCore/source/*.cpp"
    "${SDK_ROOT}/xmp/toolkit/third-party/zuid/sources/*.cpp"
    "${SDK_ROOT}/xmp/toolkit/XMPCommon/source/*.cpp"
    "${SDK_ROOT}/xmp/toolkit/XMPCore/third-party/expat/public/lib/*.c"
    "${SDK_ROOT}/xmp/toolkit/XMPFiles/source/*.cpp"
    "${SDK_ROOT}/xmp/toolkit/XMPFiles/source/FileHandlers/*.cpp"
    "${SDK_ROOT}/xmp/toolkit/XMPFiles/source/FormatSupport/*.cpp"
    "${SDK_ROOT}/xmp/toolkit/XMPFiles/source/FormatSupport/AIFF/*.cpp"
    "${SDK_ROOT}/xmp/toolkit/XMPFiles/source/FormatSupport/IFF/*.cpp"
    "${SDK_ROOT}/xmp/toolkit/XMPFiles/source/FormatSupport/WAVE/*.cpp"
    "${SDK_ROOT}/xmp/toolkit/XMPFiles/source/FormatSupport/WebP/*.cpp"
    "${SDK_ROOT}/xmp/toolkit/XMPFiles/source/NativeMetadataSupport/*.cpp"
    "${SDK_ROOT}/xmp/toolkit/XMPFiles/source/PluginHandler/FileHandlerInstance.cpp"
    "${SDK_ROOT}/xmp/toolkit/XMPFiles/source/PluginHandler/HostAPIImpl.cpp"
    "${SDK_ROOT}/xmp/toolkit/XMPFiles/source/PluginHandler/Module.cpp"
    "${SDK_ROOT}/xmp/toolkit/XMPFiles/source/PluginHandler/PluginManager.cpp"
    "${SDK_ROOT}/xmp/toolkit/XMPFiles/source/PluginHandler/XMPAtoms.cpp"
    "${SDK_ROOT}/xmp/toolkit/XMPFiles/source/WXMPFiles.cpp"
    "${SDK_ROOT}/xmp/toolkit/public/include/client-glue/*.incl_cpp"
    "${SDK_ROOT}/dng_sdk/source/*.cpp"
)

foreach(src ${XMP_CORE_SRC})
    list(APPEND SRC_FILES "${SDK_ROOT}/xmp/toolkit/XMPCore/source/${src}")
endforeach()

# Clean up dng_validate from source list
list(REMOVE_ITEM SRC_FILES "${DNG_VALIDATE_SRC}")

# Platform-specific source additions
if(APPLE)
    list(APPEND SRC_FILES 
        "${SDK_ROOT}/xmp/toolkit/XMPFiles/source/PluginHandler/OS_Utils_Mac.cpp"
        "${SDK_ROOT}/xmp/toolkit/source/Host_IO-POSIX.cpp")
elseif(WIN32)
    list(APPEND SRC_FILES 
        "${SDK_ROOT}/xmp/toolkit/XMPFiles/source/PluginHandler/OS_Utils_WIN.cpp"
        "${SDK_ROOT}/xmp/toolkit/source/Host_IO-Win.cpp"
        "${SDK_ROOT}/libjpeg/*.c")
else()
    list(APPEND SRC_FILES 
        "${SDK_ROOT}/xmp/toolkit/XMPFiles/source/PluginHandler/OS_Utils_Linux.cpp"
        "${SDK_ROOT}/xmp/toolkit/source/Host_IO-POSIX.cpp")
endif()

add_library(dng SHARED ${SRC_FILES})
set_target_properties(dng PROPERTIES VERSION ${PROJECT_VERSION} SOVERSION ${PROJECT_VERSION_MAJOR})

# --- External Dependencies
if(NOT ZLIB_FOUND)
    message(SEND_ERROR "zlib not found...")
endif()

if(WIN32)
    target_include_directories(dng PRIVATE "${SDK_ROOT}/libjpeg")   
else()
    ProcessorCount(N)
    ExternalProject_Add(libjpeg
        SOURCE_DIR "${SDK_ROOT}/libjpeg"
        CONFIGURE_COMMAND "${SDK_ROOT}/libjpeg/configure" --prefix=${CMAKE_INSTALL_PREFIX}
        BUILD_COMMAND make -j${N}
    )
    add_dependencies(dng libjpeg)
endif()

ExternalProject_Add(libjxl
    SOURCE_DIR "${SDK_ROOT}/libjxl/libjxl"
    CMAKE_ARGS -DCMAKE_INSTALL_PREFIX=${CMAKE_INSTALL_PREFIX} -DCMAKE_BUILD_TYPE=${CMAKE_BUILD_TYPE} 
               -DBUILD_SHARED_LIBS=ON -DBUILD_TESTING=OFF -DBROTLI_BUNDLED_MODE=ON
               -DJPEGXL_ENABLE_SJPEG=OFF -DJPEGXL_ENABLE_EXAMPLES=OFF -DJPEGXL_ENABLE_TOOLS=OFF
)
add_dependencies(dng libjxl)

# --- Compile Definitions & Includes
target_compile_definitions(dng PRIVATE
    $<IF:$<CONFIG:Debug>,qDNGDebug=1,qDNGDebug=0>
    qDNGValidate=1 qDNGUseXMP=1 qDNGUseLibJPEG=1 AdobePrivate=1
    BUILDING_XMPCORE_LIB=1 BUILDING_XMPCORE_AS_STATIC=1 XMP_StaticBuild=1
    XMP_COMPONENT_INT_NAMESPACE=AdobeXMPCore_Int
    $<$<BOOL:${BUILD_DNG_VALIDATE}>:qDNGValidateTarget=1>
    $<$<NOT:$<BOOL:${BUILD_DNG_VALIDATE}>>:qDNGValidateTarget=0>
)

if(APPLE)
    target_compile_definitions(dng PUBLIC qMacOS=1 MAC_ENV XMP_64=1)
    target_link_libraries(dng PRIVATE "-framework CoreServices")
elseif(WIN32)
    target_compile_definitions(dng PUBLIC qWinOS=1 WIN_ENV NOMINMAX UNICODE)
else() # Linux
    target_compile_options(dng PUBLIC -fPIC -Wall -Wextra)
    target_compile_definitions(dng PUBLIC qLinux=1 UNIX_ENV XMP_64=1)
endif()

target_include_directories(dng PRIVATE
    ${ZLIB_INCLUDE_DIRS}
    "${CMAKE_INSTALL_PREFIX}/include"
    "${SDK_ROOT}/xmp/toolkit"
    "${SDK_ROOT}/xmp/toolkit/public/include"
    "${SDK_ROOT}/xmp/toolkit/XMPFilesPlugins/api/source"
    "${SDK_ROOT}/xmp/toolkit/XMPCore/third-party/expat/public/lib"
    "${SDK_ROOT}/xmp/toolkit/XMPCore/third-party/boost"
    "${SDK_ROOT}/xmp/toolkit/public/include/XMPCore/Interfaces"
    "${SDK_ROOT}/xmp/toolkit/public/include/client-glue"
)

target_link_directories(dng PRIVATE "${CMAKE_INSTALL_PREFIX}/lib")
target_link_libraries(dng PRIVATE ${ZLIB_LIBRARIES} jxl jxl_threads $<$<NOT:$<PLATFORM_ID:Windows>>:jpeg>)

# --- Tooling
if(BUILD_DNG_VALIDATE)
    add_executable(dng_validate ${DNG_VALIDATE_SRC})
    target_compile_definitions(dng_validate PRIVATE qDNGValidateTarget=1 qDNGUseXMP=1)
    target_include_directories(dng_validate PRIVATE 
        "${SDK_ROOT}/dng_sdk/source"
        "${SDK_ROOT}/libjxl/libjxl/lib/include")
    target_link_libraries(dng_validate PRIVATE dng)
endif()

# --- Installation
install(DIRECTORY "${SDK_ROOT}/dng_sdk/source/" DESTINATION include/dng FILES_MATCHING PATTERN "*.h")
install(TARGETS dng LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR})
if(BUILD_DNG_VALIDATE)
    install(TARGETS dng_validate DESTINATION ${CMAKE_INSTALL_BINDIR})
endif()
