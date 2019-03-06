## CMake build script.

cmake_minimum_required (VERSION 2.8.8)

project (gperftools)

set (VERSION_MAJOR 2)
set (VERSION_MINOR 7)
set (VERSION_PATCH 0)

set (TCMALLOC_SO_VERSION 9:3:5)
set (PROFILER_SO_VERSION 4:18:4)

set (TC_VERSION_MAJOR VERSION_MAJOR)
set (TC_VERSION_MINOR VERSION_MINOR)
set (TC_VERSION_PATCH VERSION_PATCH)

## Update this value for every release!  (A:B:C will map to foo.so.(A-C).C.B)
## http://www.gnu.org/software/libtool/manual/html_node/Updating-version-info.html
#TCMALLOC_SO_VERSION=9:3:5
#PROFILER_SO_VERSION=4:18:4

if (WIN32)
    set (SRCS
        src/base/dynamic_annotations.c
        src/base/logging.cc
        src/base/low_level_alloc.cc
        src/base/spinlock.cc
        src/base/spinlock_internal.cc
        src/base/sysinfo.cc
        src/central_freelist.cc
        src/common.cc
        src/fake_stacktrace_scope.cc
        src/heap-profile-table.cc
        src/internal_logging.cc
        src/malloc_extension.cc
        src/malloc_hook.cc
        src/memory_region_map.cc
        src/page_heap.cc
        src/raw_printer.cc
        src/sampler.cc
        src/span.cc
        src/stacktrace.cc
        src/stack_trace_table.cc
        src/static_vars.cc
        src/symbolize.cc
        src/thread_cache.cc
        src/windows/ia32_modrm_map.cc
        src/windows/ia32_opcode_map.cc
        src/windows/mini_disassembler.cc
        src/windows/override_functions.cc)
endif ()

