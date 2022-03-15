cmake_minimum_required(VERSION 3.4)

project(covscript-runtime)
include_directories(include)

# Compiler Options
set(CMAKE_CXX_STANDARD 14)

if (MSVC)
    set(CMAKE_CXX_FLAGS "/O2 /EHsc /utf-8 /w")
    set(CMAKE_WINDOWS_EXPORT_ALL_SYMBOLS ON)
    set(rc_flags "/nologo /c65001")
    set(CMAKE_RC_FLAGS ${rc_flags})
elseif (CMAKE_COMPILER_IS_GNUCXX)
    if (WIN32)
        set(CMAKE_CXX_FLAGS "--static -fPIC -s -O3")
    else ()
        set(CMAKE_CXX_FLAGS "-fPIC -s -O3")
    endif ()
else ()
    set(CMAKE_CXX_FLAGS "-fPIC -O3")
endif ()

if (DEFINED ENV{CS_COMPATIBILITY_MODE})
    add_compile_definitions(CS_COMPATIBILITY_MODE)
endif ()

# Source Code
set(SOURCE_CODE
        sources/compiler/codegen.cpp
        sources/compiler/compiler.cpp
        sources/compiler/lexer.cpp
        sources/compiler/parser.cpp
        sources/instance/type_ext.cpp
        sources/instance/instance.cpp
        sources/instance/runtime.cpp
        sources/instance/statement.cpp
        sources/system/common.cpp
        sources/covscript.cpp)

# Static Library
add_library(csrt STATIC ${SOURCE_CODE})

# Link libdl in UNIX and APPLE platform
if (UNIX OR APPLE)
    target_link_libraries(csrt pthread dl)
endif ()

# Main Executable
if (WIN32)
    add_executable(mincs sources/interpreter.cpp sources/win32_rc/interpreter.rc)
else ()
    add_executable(mincs sources/interpreter.cpp)
endif ()

target_link_libraries(mincs csrt)

# Tests
add_library(test-extension SHARED tests/extension.cpp)
add_library(test-reflection SHARED tests/reflection.cpp)
add_executable(test-covscript tests/function_invoker.cpp)

target_link_libraries(test-extension csrt)
target_link_libraries(test-reflection csrt)
target_link_libraries(test-covscript csrt)

set_target_properties(test-extension PROPERTIES OUTPUT_NAME my_ext)
set_target_properties(test-extension PROPERTIES PREFIX "")
set_target_properties(test-extension PROPERTIES SUFFIX ".cse")

set_target_properties(test-reflection PROPERTIES OUTPUT_NAME reflect)
set_target_properties(test-reflection PROPERTIES PREFIX "")
set_target_properties(test-reflection PROPERTIES SUFFIX ".cse")