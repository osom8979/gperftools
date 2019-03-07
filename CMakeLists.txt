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

set (default_enable_libunwind YES)
set (default_enable_backtrace YES)

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
option (with_tcmalloc_pagesize "Set the tcmalloc internal page size to 8K, 32K or 64K" ${default_tcmalloc_pagesize})
option (with_tcmalloc_alignment "Set the tcmalloc allocation alignment to 8 or 16 bytes" ${default_tcmalloc_alignment})

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

## TODO(csilvers): we could remove a lot when WITH_CPU_PROFILER etc is "no".
#AC_CHECK_TYPES([__int64])       # defined in some windows platforms
#AC_CHECK_TYPES([struct mallinfo],,, [#include <malloc.h>])
#AC_CHECK_TYPES([Elf32_Versym],,, [#include <elf.h>])   # for vdso_support.h
#AC_CHECK_FUNCS(sbrk)            # for tcmalloc to get memory
#AC_CHECK_FUNCS(__sbrk)          # for tcmalloc to get memory
#AC_CHECK_FUNCS(geteuid)         # for turning off services when run as root
#AC_CHECK_FUNCS(fork)            # for the pthread_atfork setup
#AC_CHECK_HEADERS(features.h)    # for vdso_support.h
#AC_CHECK_HEADERS(malloc.h)      # some systems define stuff there, others not
#AC_CHECK_HEADERS(glob.h)        # for heap-profile-table (cleaning up profiles)
#AC_CHECK_HEADERS(execinfo.h)    # for stacktrace? and heapchecker_unittest
#AC_CHECK_HEADERS(unwind.h)      # for stacktrace
#AC_CHECK_HEADERS(sched.h)       # for being nice in our spinlock code
#AC_CHECK_HEADERS(conflict-signal.h)      # defined on some windows platforms?
#AC_CHECK_HEADERS(sys/prctl.h)   # for thread_lister (needed by leak-checker)
#AC_CHECK_HEADERS(linux/ptrace.h)# also needed by leak-checker
#AC_CHECK_HEADERS(sys/syscall.h)
#AC_CHECK_HEADERS(sys/socket.h)  # optional; for forking out to symbolizer
#AC_CHECK_HEADERS(sys/wait.h)    # optional; for forking out to symbolizer
#AC_CHECK_HEADERS(poll.h)        # optional; for forking out to symbolizer
#AC_CHECK_HEADERS(fcntl.h)       # for tcmalloc_unittest
#AC_CHECK_HEADERS(grp.h)         # for heapchecker_unittest
#AC_CHECK_HEADERS(pwd.h)         # for heapchecker_unittest
#AC_CHECK_HEADERS(sys/resource.h)         # for memalign_unittest.cc
#AC_CHECK_HEADERS(valgrind.h)    # we have a local copy if this isn't found
#AC_CHECK_HEADERS(sys/cdefs.h)   # Where glibc defines __THROW
#AC_CHECK_HEADERS(features.h)    # Where __GLIBC__ is defined
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

