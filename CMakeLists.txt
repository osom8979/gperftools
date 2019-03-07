## CMake build script.

cmake_minimum_required (VERSION 2.8.8)

project (gperftools)

set (VERSION_MAJOR 2)
set (VERSION_MINOR 7)
set (VERSION_PATCH 0)

## Update this value for every release!  (A:B:C will map to foo.so.(A-C).C.B)
## http://www.gnu.org/software/libtool/manual/html_node/Updating-version-info.html
set (TCMALLOC_SO_VERSION 9:3:5)
set (PROFILER_SO_VERSION 4:18:4)

set (TC_VERSION_MAJOR VERSION_MAJOR)
set (TC_VERSION_MINOR VERSION_MINOR)
set (TC_VERSION_PATCH VERSION_PATCH)

set (default_enable_cpu_profiler YES)
set (default_enable_heap_profiler YES)
set (default_enable_heap_checker YES)
set (default_enable_debugalloc YES)
set (default_enable_minimal NO)
set (default_tcmalloc_alignment 16)
set (need_nanosleep YES) # Used later, to decide if to run ACX_NANOSLEEP

if (MINGW)
    set (default_enable_minimal YES)
    set (default_enable_debugalloc NO)
elseif (CYGWIN)
    set (default_enable_heap_checker NO)
    set (default_enable_cpu_profiler NO)
elseif (CMAKE_SYSTEM_NAME MATCHES "^k?FreeBSD$")
    set (default_enable_heap_checker NO)
elseif (CMAKE_SYSTEM_NAME MATCHES "Darwin")
    set (default_enable_heap_checker NO)
endif ()

include (CheckCSourceCompiles)
include (CheckCXXSourceCompiles)
check_c_source_compiles ("int main() { return __s390__; }" HAVE___S390__)
check_c_source_compiles ("int main() { return __PPC64__; }" HAVE___PPC64__)

if (HAVE___S390__)
    set (default_enable_libunwind NO)
    set (default_enable_backtrace YES)
else ()
    set (default_enable_libunwind YES)
    set (default_enable_backtrace NO)
endif ()

if (HAVE___PPC64__)
    set (default_enable_libunwind NO)
    set (default_tcmalloc_pagesize 64)
else ()
    set (default_enable_libunwind YES)
    set (default_tcmalloc_pagesize 8)
endif ()

option (enable_cpu_profiler "build the cpu profiler" ${default_enable_cpu_profiler})
option (enable_heap_profiler "build the heap profiler" ${default_enable_heap_profiler})
option (enable_heap_checker "build the heap checker" ${default_enable_heap_checker})
option (enable_debugalloc "build versions of libs with debugalloc" ${default_enable_debugalloc})
option (enable_minimal "build only tcmalloc-minimal (and maybe tcmalloc-minimal-debug)" ${default_enable_minimal})

if (enable_minimal)
    set (enable_cpu_profiler NO)
    set (enable_heap_profiler NO)
    set (enable_heap_checker NO)
endif ()

option (enable_backtrace "enable use of backtrace() for stacktrace capturing (may deadlock)" ${default_enable_backtrace})
option (enable_libunwind "enable libunwind linking" ${default_enable_libunwind})

set (with_tcmalloc_pagesize ${default_tcmalloc_pagesize} CACHE STRING "Set the tcmalloc internal page size to 8K, 32K or 64K")
set (with_tcmalloc_alignment ${default_tcmalloc_alignment} CACHE STRING "Set the tcmalloc allocation alignment to 8 or 16 bytes")

if (${with_tcmalloc_pagesize} EQUAL 8)
    message (STATUS "Default tcmalloc page size")
elseif (${with_tcmalloc_pagesize} EQUAL 32)
    add_definitions (-DTCMALLOC_32K_PAGES=1)
    message (STATUS "Define 32K of internal pages size for tcmalloc")
elseif (${with_tcmalloc_pagesize} EQUAL 64)
    add_definitions (-DTCMALLOC_64K_PAGES=1)
    message (STATUS "Define 64K of internal pages size for tcmalloc")
else ()
    message (WARNING "${with_tcmalloc_pagesize}K size not supported, using default tcmalloc page size.")
endif ()

if (${with_tcmalloc_alignment} EQUAL 8)
    add_definitions (-DTCMALLOC_ALIGN_8BYTES=1)
    message (STATUS "Define 8 bytes of allocation alignment for tcmalloc")
elseif (${with_tcmalloc_alignment} EQUAL 16)
    message (STATUS "Default tcmalloc allocation alignment.")
else ()
    message (WARNING "${with_tcmalloc_alignment} bytes not supported, using default tcmalloc allocation alignment.")
endif ()

## Check if we have an objcopy installed that supports -W
#AC_CHECK_TOOL([OBJCOPY], [objcopy], [])
#AS_IF([test -n "$OBJCOPY"], [dnl
#  AC_CACHE_CHECK([if $OBJCOPY supports -W], gpt_cv_objcopy_weaken, [dnl
#    AC_LINK_IFELSE([AC_LANG_PROGRAM([void foo() {} int main() {return 0;}])], [dnl
#      AS_IF(["$OBJCOPY" -W foo conftest$ac_exeext /dev/null],
#            [gpt_cv_objcopy_weaken=yes], [gpt_cv_objcopy_weaken=no])],
#    [gpt_cv_objcopy_weaken=no])])],
#  [gpt_cv_objcopy_weaken=no])
#AM_CONDITIONAL(HAVE_OBJCOPY_WEAKEN, test $gpt_cv_objcopy_weaken = yes)

#AC_MSG_CHECKING(for __attribute__((aligned(N))) on functions)
#AC_CACHE_VAL(ac_cv___attribute__aligned_fn, [
#  AC_TRY_COMPILE(
#    [#include <stdlib.h>
#     void foo(void) __attribute__((aligned(128)));
#     void foo(void) { exit(1); }],
#    [],
#    ac_cv___attribute__aligned_fn=yes,
#    ac_cv___attribute__aligned_fn=no
#  )])
#if test "$ac_cv___attribute__aligned_fn" = "yes"; then
#  AC_DEFINE(HAVE___ATTRIBUTE__ALIGNED_FN, 1, [define if your compiler supports alignment of functions])
#fi
#AC_MSG_RESULT($ac_cv___attribute__aligned_fn)

include (CheckTypeSize)
check_type_size ("__int64" HAVE____INT64) # defined in some windows platforms
set (CMAKE_EXTRA_INCLUDE_FILES malloc.h)
check_type_size ("struct mallinfo" HAVE_STRUCT_MALLINFO)
set (CMAKE_EXTRA_INCLUDE_FILES elf.h)
check_type_size ("Elf32_Versym" HAVE_ELF32_VERSYM) # for vdso_support.h

include (CheckFunctionExists)
check_function_exists (sbrk HAVE_SBRK) # for tcmalloc to get memory
check_function_exists (__sbrk HAVE___SBRK) # for tcmalloc to get memory
check_function_exists (geteuid HAVE_GETEUID) # for turning off services when run as root
check_function_exists (fork HAVE_FORK) # for the pthread_atfork setup

include (CheckIncludeFile)
check_include_file (features.h HAVE_FEATURES_H) # for vdso_support.h
check_include_file (malloc.h HAVE_MALLOC_H) # some systems define stuff there, others not
check_include_file (glob.h HAVE_GLOB_H) # for heap-profile-table (cleaning up profiles)
check_include_file (execinfo.h HAVE_EXECINFO_H) # for stacktrace? and heapchecker_unittest
check_include_file (unwind.h HAVE_UNWIND_H) # for stacktrace
check_include_file (sched.h HAVE_SCHED_H) # for being nice in our spinlock code
check_include_file (conflict-signal.h HAVE_CONFLICT_SIGNAL_H) # defined on some windows platforms?
check_include_file (sys/prctl.h HAVE_SYS_PRCTL_H) # for thread_lister (needed by leak-checker)
check_include_file (linux/ptrace.h HAVE_LINUX_PTRACE_H) # also needed by leak-checker
check_include_file (sys/syscall.h HAVE_SYS_SYSCALL_H)
check_include_file (sys/socket.h HAVE_SYS_SOCKET_H) # optional; for forking out to symbolizer
check_include_file (sys/wait.h HAVE_SYS_WAIT_H) # optional; for forking out to symbolizer
check_include_file (poll.h HAVE_POLL_H) # optional; for forking out to symbolizer
check_include_file (fcntl.h HAVE_FCNTL_H) # for tcmalloc_unittest
check_include_file (grp.h HAVE_GRP_H) # for heapchecker_unittest
check_include_file (pwd.h HAVE_PWD_H) # for heapchecker_unittest
check_include_file (sys/resource.h HAVE_SYS_RESOURCE_H) # for memalign_unittest.cc
check_include_file (valgrind.h HAVE_VALGRIND_H) # we have a local copy if this isn't found
check_include_file (sys/cdefs.h HAVE_SYS_CDEFS_H) # Where glibc defines __THROW
check_include_file (features.h HAVE_FEATURES_H) # Where __GLIBC__ is defined

## We also need <ucontext.h>/<sys/ucontext.h>, but we get those from
## AC_PC_FROM_UCONTEXT, below.

## We override a lot of memory allocation routines, not all of which are
## standard.  For those the system doesn't declare, we'll declare ourselves.
#AC_CHECK_DECLS([cfree,
#                posix_memalign,
#                memalign,
#                valloc,
#                pvalloc],,,
#               [#define _XOPEN_SOURCE 600
#                #include <stdlib.h>
#                #include <malloc.h>])


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

