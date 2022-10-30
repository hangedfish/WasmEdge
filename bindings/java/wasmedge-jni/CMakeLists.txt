cmake_minimum_required(VERSION 3.16)
project(wasmedge_jni C)

set(CMAKE_C_STANDARD 11)
set(TARGET wasmedge_jni)

find_package(JNI REQUIRED)
include_directories(${JNI_INCLUDE_DIRS})

list(APPEND CMAKE_MODULE_PATH ${CMAKE_CURRENT_SOURCE_DIR})
configure_file(wasmedge-version.properties.in ${CMAKE_CURRENT_BINARY_DIR}/wasmedge-version.properties)

find_package(WasmEdge QUIET)

if (${WasmEdge_FOUND})
        include_directories(${WasmEdge_INCLUDE_DIRS})
        list(APPEND needed_libs ${WasmEdge_LIBRARIES})
else()
        unset(WasmEdge_INCLUDE_DIRS-NOTFOUND CACHE)
        get_filename_component(WASMEDGE_SOURCE_DIR ${CMAKE_CURRENT_SOURCE_DIR}/../../.. ABSOLUTE)
        set(WASMEDGE_BUILD_AOT_RUNTIME OFF CACHE BOOL "" FORCE)
        set(WASMEDGE_BUILD_PLUGINS OFF CACHE BOOL "" FORCE)
        set(WASMEDGE_BUILD_TOOLS OFF CACHE BOOL "" FORCE)
        set(WASMEDGE_BUILD_SHARED_LIB ON CACHE BOOL "" FORCE)
        set(WASMEDGE_FORCE_DISABLE_LTO ON CACHE BOOL "" FORCE)
        if (CMAKE_GENERATOR STREQUAL Ninja)
                set(CMAKE_JOB_POOLS "link=2")
                set(CMAKE_JOB_POOL_LINK link)
        endif()
        add_subdirectory(${WASMEDGE_SOURCE_DIR} WasmEdge)
        list(APPEND needed_libs wasmedge_shared)
        include_directories(${WasmEdge_BINARY_DIR}/include/api)
endif()

add_library(${TARGET} SHARED
        lib/WasmEdge.c
        lib/WasmEdgeVM.c
        lib/common.c
        lib/StoreContext.c
        lib/ConfigureContext.c
        lib/FunctionTypeContext.c
        lib/FunctionTypeContext.h
        lib/GlobalTypeContext.c
        lib/TableTypeContext.c
        lib/CompilerContext.c
        lib/LoaderContext.c
        lib/AstModuleContext.c
        lib/StatisticsContext.c
        lib/ExecutorContext.c
        lib/ValidatorContext.c
        lib/GlobalInstanceContext.c
        lib/ValueType.c
        lib/TableInstanceContext.c
        lib/TableTypeContext.h
        lib/MemoryInstanceContext.c
        lib/MemoryTypeContext.c
        lib/MemoryTypeContext.h
        lib/ExternRef.c
        lib/ExportTypeContext.c
        lib/ImportTypeContext.c
        lib/ModuleInstanceContext.c
        lib/FunctionTypeInstance.c
        lib/FunctionTypeInstance.h
        lib/GlobalInstanceContext.h
        lib/MemoryInstanceContext.h
        lib/TableInstanceContext.h
        lib/ModuleInstanceContext.h
        lib/WasmEdgeAsync.h
        lib/WasmEdgeAsync.c)

target_link_libraries(${TARGET} ${needed_libs})

install(TARGETS ${TARGET}
        LIBRARY DESTINATION lib)
