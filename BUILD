# -*- mode: python; -*-

# nsync is a C library that exports synchronization primitives, such as reader
# writer locks with conditional critical sections, designed to be open sourced
# in portable C.  See https://github.com/google/nsync
#
# See public/*.h for API.  When compiled with C++11 rather than C, it's in the
# "nsync" name space.
#
# BUILD file usage:
#   deps = "@nsync://nsync" for C version
#   deps = "@nsync://nsync_cpp" for C++11 version.
# The latter uses no OS-specific system calls or architecture-specific atomic
# operations.

package(default_visibility = ["//visibility:public"])

licenses(["notice"])  # Apache 2.0

exports_files([
    "LICENSE",
    "VERSION",
])

# ---------------------------------------------
# Parameters to the compilation:  compiler (e.g., for atomics), architecture
# (e.g., for load and store barrier behaviour), and OS.
# Bazel merges these into one, somewhat slippery, "cpu" string.
# Bazel uses a rather verbose mechanism for choosing which implementations
# are needed on given platforms; hence all the config_setting() rules below.

config_setting(
    name = "clang",
    flag_values = {"@bazel_tools//tools/cpp:compiler": "clang"},
)

config_setting(
    name = "gcc",
    flag_values = {"@bazel_tools//tools/cpp:compiler": "gcc"},
)

config_setting(
    name = "msvc",
    flag_values = {"@bazel_tools//tools/cpp:compiler": "msvc"},
)

config_setting(
    name = "windows_msvc",
    constraint_values = ["@platforms//os:windows"],
    flag_values = {"@bazel_tools//tools/cpp:compiler": "msvc"},
)

# ---------------------------------------------
# Compilation options.

# load(":bazel/pkg_path_name.bzl", "pkg_path_name")

NSYNC_INCLUDES_GENERIC = select({
    "@platforms//cpu:aarch32": ["platform/arm"],
    "@platforms//cpu:aarch64": ["platform/aarch64"],
    "@platforms//cpu:ppc64le": ["platform/ppc64"],
    "@platforms//cpu:s390x": ["platform/s390x"],
    "@platforms//cpu:x86_32": ["platform/x86_32"],
    "@platforms//cpu:x86_64": ["platform/x86_64"],
}) + [
    "public",
    "internal",
    "platform/posix",
]

NSYNC_INCLUDES = select({
    # Select the OS include directory.
    "@platforms//os:linux": ["platform/linux"],
    "@platforms//os:macos": ["platform/macos"],
    "@platforms//os:freebsd": ["platform/freebsd"],
    "@platforms//os:windows": ["platform/win32"],
    "//conditions:default": [],
}) + NSYNC_INCLUDES_GENERIC

NSYNC_INCLUDES_CPP = select({
    # On Linux, the C++11 library's synchronization primitives are
    # surprisingly slow.   See also NSYNC_SRC_PLATFORM_CPP, below.
    "@platforms//os:linux": ["platform/c++11.futex"],
    "@platforms//os:ios": ["platform/gcc_no_tls"],
    "@platforms//os:windows": ["platform/win32"],
    "//conditions:default": [],
}) + [
    "platform/c++11",
] + select({
    ":windows_msvc": [
        "platform/win32",
        "platform/msvc",
    ],
    "//conditions:default": ["platform/gcc"],
}) + NSYNC_INCLUDES_GENERIC

NSYNC_OPTS_GENERIC = select({
    "@platforms//os:windows": [],
    "@platforms//os:freebsd": ["-pthread"],
    "//conditions:default": [
        "-D_POSIX_C_SOURCE=200809L",
        "-pthread",
    ],
})

NSYNC_OPTS = select({
    # Apple deprecated their atomics library, yet recent versions have no
    # working version of stdatomic.h; so some recent versions need one, and
    # other versions prefer the other.  For the moment, just ignore the
    # depreaction.
    "@platforms//os:macos": ["-Wno-deprecated-declarations"],
    "//conditions:default": [],
}) + NSYNC_OPTS_GENERIC

# Options for C build, rather then C++11 build.
# Options for C++11 build, rather then C build.
NSYNC_OPTS_CPP = select({
    ":msvc": [
        "/TP",
    ],
    "//conditions:default": [
        "-x",
        "c++",
        "-std=c++11",
    ],
}) + select({
    # Some versions of MacOS (notably Sierra) require -D_DARWIN_C_SOURCE
    # to include some standard C++11 headers, like <mutex>.
    "@platforms//os:macos": ["-D_DARWIN_C_SOURCE"],
    "//conditions:default": [],
}) + [
    "-DNSYNC_ATOMIC_CPP11",
    "-DNSYNC_USE_CPP11_TIMEPOINT",
] + NSYNC_OPTS_GENERIC

# Link options (for tests) built in C (rather than C++11).
NSYNC_LINK_OPTS = select({
    ":msvc": [],
    "//conditions:default": ["-pthread"],
})

# Link options (for tests) built in C++11 (rather than C).
NSYNC_LINK_OPTS_CPP = select({
    ":msvc": [],
    "//conditions:default": ["-pthread"],
})

# ---------------------------------------------
# Header files the source may include.

# Internal library headers.
NSYNC_INTERNAL_HEADERS = [
    "internal/common.h",
    "internal/dll.h",
    "internal/headers.h",
    "internal/sem.h",
    "internal/wait_internal.h",
]

# Internal test headers.
NSYNC_TEST_HEADERS = NSYNC_INTERNAL_HEADERS + [
    "testing/array.h",
    "testing/atm_log.h",
    "testing/closure.h",
    "testing/heap.h",
    "testing/smprintf.h",
    "testing/testing.h",
    "testing/time_extra.h",
]

# Platform specific headers.
# This declares headers for all platforms, not just the one
# we're building for, to avoid a more complex build file.
NSYNC_INTERNAL_HEADERS_PLATFORM = [
    "platform/aarch64/cputype.h",
    "platform/alpha/cputype.h",
    "platform/arm/cputype.h",
    "platform/atomic_ind/atomic.h",
    "platform/c++11/atomic.h",
    "platform/c++11/platform.h",
    "platform/c++11.futex/platform.h",
    "platform/c11/atomic.h",
    "platform/clang/atomic.h",
    "platform/clang/compiler.h",
    "platform/cygwin/platform.h",
    "platform/decc/compiler.h",
    "platform/freebsd/platform.h",
    "platform/gcc/atomic.h",
    "platform/gcc/compiler.h",
    "platform/gcc_new/atomic.h",
    "platform/gcc_new_debug/atomic.h",
    "platform/gcc_no_tls/compiler.h",
    "platform/gcc_old/atomic.h",
    "platform/lcc/compiler.h",
    "platform/lcc/nsync_time_init.h",
    "platform/linux/platform.h",
    "platform/win32/atomic.h",
    "platform/macos/platform_c++11_os.h",
    "platform/msvc/compiler.h",
    "platform/netbsd/atomic.h",
    "platform/netbsd/platform.h",
    "platform/openbsd/platform.h",
    "platform/osf1/platform.h",
    "platform/macos/atomic.h",
    "platform/macos/platform.h",
    "platform/pmax/cputype.h",
    "platform/posix/cputype.h",
    "platform/posix/nsync_time_init.h",
    "platform/posix/platform_c++11_os.h",
    "platform/ppc32/cputype.h",
    "platform/ppc64/cputype.h",
    "platform/s390x/cputype.h",
    "platform/shark/cputype.h",
    "platform/tcc/compiler.h",
    "platform/win32/platform.h",
    "platform/win32/platform_c++11_os.h",
    "platform/x86_32/cputype.h",
    "platform/x86_64/cputype.h",
]

# ---------------------------------------------
# The nsync library.

# Linux-specific library source.
NSYNC_SRC_LINUX = [
    "platform/linux/src/nsync_semaphore_futex.c",
    "platform/posix/src/per_thread_waiter.c",
    "platform/posix/src/yield.c",
    "platform/posix/src/time_rep.c",
    "platform/posix/src/nsync_panic.c",
]

# Android-specific library source.
NSYNC_SRC_ANDROID = [
    "platform/posix/src/nsync_semaphore_sem_t.c",
    "platform/posix/src/per_thread_waiter.c",
    "platform/posix/src/yield.c",
    "platform/posix/src/time_rep.c",
    "platform/posix/src/nsync_panic.c",
]

# MacOS-specific library source.
NSYNC_SRC_MACOS = [
    "platform/posix/src/clock_gettime.c",
    "platform/posix/src/nsync_semaphore_mutex.c",
    "platform/posix/src/per_thread_waiter.c",
    "platform/posix/src/yield.c",
    "platform/posix/src/time_rep.c",
    "platform/posix/src/nsync_panic.c",
]

# Windows-specific library source.
NSYNC_SRC_WINDOWS = [
    "platform/posix/src/nsync_panic.c",
    "platform/posix/src/per_thread_waiter.c",
    "platform/posix/src/time_rep.c",
    "platform/posix/src/yield.c",
    "platform/win32/src/clock_gettime.c",
    "platform/win32/src/init_callback_win32.c",
    "platform/win32/src/nanosleep.c",
    "platform/win32/src/nsync_semaphore_win32.c",
    "platform/win32/src/pthread_cond_timedwait_win32.c",
    "platform/win32/src/pthread_key_win32.cc",
]

# FreeBSD-specific library source.
NSYNC_SRC_FREEBSD = [
    "platform/posix/src/nsync_semaphore_sem_t.c",
    "platform/posix/src/per_thread_waiter.c",
    "platform/posix/src/yield.c",
    "platform/posix/src/time_rep.c",
    "platform/posix/src/nsync_panic.c",
]

# OS-specific library source.
NSYNC_SRC_PLATFORM = select({
    "@platforms//os:linux": NSYNC_SRC_LINUX,
    "@platforms//os:macos": NSYNC_SRC_MACOS,
    "@platforms//os:freebsd": NSYNC_SRC_FREEBSD,
    "@platforms//os:windows": NSYNC_SRC_WINDOWS,
    "@platforms//os:android": NSYNC_SRC_ANDROID,
})

# C++11-specific (OS and architecture independent) library source.
NSYNC_SRC_PLATFORM_CPP = [
    "platform/c++11/src/time_rep_timespec.cc",
    "platform/c++11/src/nsync_panic.cc",
    "platform/c++11/src/yield.cc",
] + select({
    # On Linux, the C++11 library's synchronization primitives are surprisingly
    # slow, at least at the time or writing (early 2018).  Raw kernel
    # primitives are ten times faster for wakeups.
    "@platforms//os:linux": ["platform/linux/src/nsync_semaphore_futex.c"],
    "//conditions:default": ["platform/c++11/src/nsync_semaphore_mutex.cc"],
}) + select({
    # MacOS and Android don't have working C++11 thread local storage.
    "@platforms//os:macos": ["platform/posix/src/per_thread_waiter.c"],
    "@platforms//os:ios": ["platform/posix/src/per_thread_waiter.c"],
    "@platforms//os:android": ["platform/posix/src/per_thread_waiter.c"],
    "@platforms//os:windows": [
        "platform/win32/src/clock_gettime.c",
        # Windows has no thread-specific data with thread-exit destructors; we
        # must emulate it with C++ per-thread class destructors.
        "platform/win32/src/pthread_key_win32.cc",
        "platform/win32/src/per_thread_waiter.c",
    ],
    # It's dangerous to use C++ class destructors if we can avoid it, because
    # nsync may be linked into the address space multiple times.
    "//conditions:default": ["platform/posix/src/per_thread_waiter.c"],
})

# Generic library source.
NSYNC_SRC_GENERIC = [
    "internal/common.c",
    "internal/counter.c",
    "internal/cv.c",
    "internal/debug.c",
    "internal/dll.c",
    "internal/mu.c",
    "internal/mu_wait.c",
    "internal/note.c",
    "internal/once.c",
    "internal/sem_wait.c",
    "internal/time_internal.c",
    "internal/wait.c",
]

# Generic library header files.
NSYNC_HDR_GENERIC = [
    "public/nsync.h",
    "public/nsync_atomic.h",
    "public/nsync_counter.h",
    "public/nsync_cpp.h",
    "public/nsync_cv.h",
    "public/nsync_debug.h",
    "public/nsync_mu.h",
    "public/nsync_mu_wait.h",
    "public/nsync_note.h",
    "public/nsync_once.h",
    "public/nsync_time.h",
    "public/nsync_time_internal.h",
    "public/nsync_waiter.h",
]

# The library compiled in C, rather than C++11.
cc_library(
    name = "nsync",
    srcs = NSYNC_SRC_GENERIC + NSYNC_SRC_PLATFORM,
    hdrs = NSYNC_HDR_GENERIC,
    copts = NSYNC_OPTS,
    includes = ["public"] + NSYNC_INCLUDES,
    textual_hdrs = NSYNC_INTERNAL_HEADERS + NSYNC_INTERNAL_HEADERS_PLATFORM,
)

# The library compiled in C++11, rather than C.
cc_library(
    name = "nsync_cpp",
    srcs = NSYNC_SRC_GENERIC + NSYNC_SRC_PLATFORM_CPP,
    hdrs = NSYNC_HDR_GENERIC,
    copts = NSYNC_OPTS_CPP,
    includes = ["public"] + NSYNC_INCLUDES_CPP,
    textual_hdrs = NSYNC_INTERNAL_HEADERS + NSYNC_INTERNAL_HEADERS_PLATFORM,
)

# nsync_headers provides just the header files for use in projects that need to
# build shared libraries for dynamic loading.  Bazel seems unable to cope
# otherwise.
cc_library(
    name = "nsync_headers",
    hdrs = glob(["public/*.h"]),
    includes = ["public"],
)

# ---------------------------------------------
# Test code.

# Linux-specific test library source.
NSYNC_TEST_SRC_LINUX = [
    "platform/posix/src/start_thread.c",
]

# Android-specific test library source.
NSYNC_TEST_SRC_ANDROID = [
    "platform/posix/src/start_thread.c",
]

# MacOS-specific test library source.
NSYNC_TEST_SRC_MACOS = [
    "platform/posix/src/start_thread.c",
]

# Windows-specific test library source.
NSYNC_TEST_SRC_WINDOWS = [
    "platform/win32/src/start_thread.c",
]

# FreeBSD-specific test library source.
NSYNC_TEST_SRC_FREEBSD = [
    "platform/posix/src/start_thread.c",
]

# OS-specific test library source.
NSYNC_TEST_SRC_PLATFORM = select({
    ":gcc_linux_x86_32_1": NSYNC_TEST_SRC_LINUX,
    ":gcc_linux_x86_64_1": NSYNC_TEST_SRC_LINUX,
    ":gcc_linux_x86_64_2": NSYNC_TEST_SRC_LINUX,
    ":gcc_linux_aarch64": NSYNC_TEST_SRC_LINUX,
    ":gcc_linux_ppc64": NSYNC_TEST_SRC_LINUX,
    ":gcc_linux_s390x": NSYNC_TEST_SRC_LINUX,
    ":clang_macos_x86_64": NSYNC_TEST_SRC_MACOS,
    ":freebsd": NSYNC_TEST_SRC_FREEBSD,
    ":ios_x86_64": NSYNC_TEST_SRC_MACOS,
    ":android_x86_32": NSYNC_TEST_SRC_ANDROID,
    ":android_x86_64": NSYNC_TEST_SRC_ANDROID,
    ":android_armeabi": NSYNC_TEST_SRC_ANDROID,
    ":android_arm": NSYNC_TEST_SRC_ANDROID,
    ":android_arm64": NSYNC_TEST_SRC_ANDROID,
    ":msvc_windows_x86_64": NSYNC_TEST_SRC_WINDOWS,
})

# C++11-specific (OS and architecture independent) test library source.
NSYNC_TEST_SRC_PLATFORM_CPP = [
    "platform/c++11/src/start_thread.cc",
]

# Generic test library source.
NSYNC_TEST_SRC_GENERIC = [
    "testing/array.c",
    "testing/atm_log.c",
    "testing/closure.c",
    "testing/smprintf.c",
    "testing/testing.c",
    "testing/time_extra.c",
]

# The test library compiled in C, rather than C++11.
cc_library(
    name = "nsync_test_lib",
    testonly = 1,
    srcs = NSYNC_TEST_SRC_GENERIC + NSYNC_TEST_SRC_PLATFORM,
    hdrs = ["testing/testing.h"],
    copts = NSYNC_OPTS,
    textual_hdrs = NSYNC_TEST_HEADERS + NSYNC_INTERNAL_HEADERS_PLATFORM,
    deps = [":nsync"],
)

# The test library compiled in C++11, rather than C.
cc_library(
    name = "nsync_test_lib_cpp",
    testonly = 1,
    srcs = NSYNC_TEST_SRC_GENERIC + NSYNC_TEST_SRC_PLATFORM_CPP,
    hdrs = ["testing/testing.h"],
    copts = NSYNC_OPTS_CPP,
    textual_hdrs = NSYNC_TEST_HEADERS + NSYNC_INTERNAL_HEADERS_PLATFORM,
    deps = [":nsync_cpp"],
)

# ---------------------------------------------
# The tests, compiled in C rather than C++11.

cc_test(
    name = "counter_test",
    size = "small",
    srcs = ["testing/counter_test.c"],
    copts = NSYNC_OPTS,
    linkopts = NSYNC_LINK_OPTS,
    deps = [
        ":nsync",
        ":nsync_test_lib",
    ],
)

cc_test(
    name = "cv_mu_timeout_stress_test",
    size = "small",
    srcs = ["testing/cv_mu_timeout_stress_test.c"],
    copts = NSYNC_OPTS,
    linkopts = NSYNC_LINK_OPTS,
    deps = [
        ":nsync",
        ":nsync_test_lib",
    ],
)

cc_test(
    name = "cv_test",
    size = "small",
    srcs = ["testing/cv_test.c"],
    copts = NSYNC_OPTS,
    linkopts = NSYNC_LINK_OPTS,
    deps = [
        ":nsync",
        ":nsync_test_lib",
    ],
)

cc_test(
    name = "cv_wait_example_test",
    size = "small",
    srcs = ["testing/cv_wait_example_test.c"],
    copts = NSYNC_OPTS,
    linkopts = NSYNC_LINK_OPTS,
    deps = [
        ":nsync",
        ":nsync_test_lib",
    ],
)

cc_test(
    name = "dll_test",
    size = "small",
    srcs = ["testing/dll_test.c"],
    copts = NSYNC_OPTS,
    linkopts = NSYNC_LINK_OPTS,
    deps = [
        ":nsync",
        ":nsync_test_lib",
    ],
)

cc_test(
    name = "mu_starvation_test",
    size = "small",
    srcs = ["testing/mu_starvation_test.c"],
    copts = NSYNC_OPTS,
    linkopts = NSYNC_LINK_OPTS,
    deps = [
        ":nsync",
        ":nsync_test_lib",
    ],
)

cc_test(
    name = "mu_test",
    size = "small",
    srcs = ["testing/mu_test.c"],
    copts = NSYNC_OPTS,
    linkopts = NSYNC_LINK_OPTS,
    deps = [
        ":nsync",
        ":nsync_test_lib",
    ],
)

cc_test(
    name = "mu_wait_example_test",
    size = "small",
    srcs = ["testing/mu_wait_example_test.c"],
    copts = NSYNC_OPTS,
    linkopts = NSYNC_LINK_OPTS,
    deps = [
        ":nsync",
        ":nsync_test_lib",
    ],
)

cc_test(
    name = "mu_wait_test",
    size = "small",
    srcs = ["testing/mu_wait_test.c"],
    copts = NSYNC_OPTS,
    linkopts = NSYNC_LINK_OPTS,
    deps = [
        ":nsync",
        ":nsync_test_lib",
    ],
)

cc_test(
    name = "note_test",
    size = "small",
    srcs = ["testing/note_test.c"],
    copts = NSYNC_OPTS,
    linkopts = NSYNC_LINK_OPTS,
    deps = [
        ":nsync",
        ":nsync_test_lib",
    ],
)

cc_test(
    name = "once_test",
    size = "small",
    srcs = ["testing/once_test.c"],
    copts = NSYNC_OPTS,
    linkopts = NSYNC_LINK_OPTS,
    deps = [
        ":nsync",
        ":nsync_test_lib",
    ],
)

cc_test(
    name = "pingpong_test",
    size = "small",
    srcs = ["testing/pingpong_test.c"],
    copts = NSYNC_OPTS,
    linkopts = NSYNC_LINK_OPTS,
    deps = [
        ":nsync",
        ":nsync_test_lib",
    ],
)

cc_test(
    name = "wait_test",
    size = "small",
    srcs = ["testing/wait_test.c"],
    copts = NSYNC_OPTS,
    linkopts = NSYNC_LINK_OPTS,
    deps = [
        ":nsync",
        ":nsync_test_lib",
    ],
)

# ---------------------------------------------
# The tests, compiled in C++11, rather than C.

cc_test(
    name = "counter_cpp_test",
    size = "small",
    srcs = ["testing/counter_test.c"],
    copts = NSYNC_OPTS_CPP,
    linkopts = NSYNC_LINK_OPTS_CPP,
    deps = [
        ":nsync_cpp",
        ":nsync_test_lib_cpp",
    ],
)

cc_test(
    name = "cv_mu_timeout_stress_cpp_test",
    size = "small",
    srcs = ["testing/cv_mu_timeout_stress_test.c"],
    copts = NSYNC_OPTS_CPP,
    linkopts = NSYNC_LINK_OPTS_CPP,
    deps = [
        ":nsync_cpp",
        ":nsync_test_lib_cpp",
    ],
)

cc_test(
    name = "cv_cpp_test",
    size = "small",
    srcs = ["testing/cv_test.c"],
    copts = NSYNC_OPTS_CPP,
    linkopts = NSYNC_LINK_OPTS_CPP,
    deps = [
        ":nsync_cpp",
        ":nsync_test_lib_cpp",
    ],
)

cc_test(
    name = "cv_wait_example_cpp_test",
    size = "small",
    srcs = ["testing/cv_wait_example_test.c"],
    copts = NSYNC_OPTS_CPP,
    linkopts = NSYNC_LINK_OPTS_CPP,
    deps = [
        ":nsync_cpp",
        ":nsync_test_lib_cpp",
    ],
)

cc_test(
    name = "dll_cpp_test",
    size = "small",
    srcs = ["testing/dll_test.c"],
    copts = NSYNC_OPTS_CPP,
    linkopts = NSYNC_LINK_OPTS_CPP,
    deps = [
        ":nsync_cpp",
        ":nsync_test_lib_cpp",
    ],
)

cc_test(
    name = "mu_starvation_cpp_test",
    size = "small",
    srcs = ["testing/mu_starvation_test.c"],
    copts = NSYNC_OPTS_CPP,
    linkopts = NSYNC_LINK_OPTS_CPP,
    deps = [
        ":nsync_cpp",
        ":nsync_test_lib_cpp",
    ],
)

cc_test(
    name = "mu_cpp_test",
    size = "small",
    srcs = ["testing/mu_test.c"],
    copts = NSYNC_OPTS_CPP,
    linkopts = NSYNC_LINK_OPTS_CPP,
    deps = [
        ":nsync_cpp",
        ":nsync_test_lib_cpp",
    ],
)

cc_test(
    name = "mu_wait_example_cpp_test",
    size = "small",
    srcs = ["testing/mu_wait_example_test.c"],
    copts = NSYNC_OPTS_CPP,
    linkopts = NSYNC_LINK_OPTS_CPP,
    deps = [
        ":nsync_cpp",
        ":nsync_test_lib_cpp",
    ],
)

cc_test(
    name = "mu_wait_cpp_test",
    size = "small",
    srcs = ["testing/mu_wait_test.c"],
    copts = NSYNC_OPTS_CPP,
    linkopts = NSYNC_LINK_OPTS_CPP,
    deps = [
        ":nsync_cpp",
        ":nsync_test_lib_cpp",
    ],
)

cc_test(
    name = "note_cpp_test",
    size = "small",
    srcs = ["testing/note_test.c"],
    copts = NSYNC_OPTS_CPP,
    linkopts = NSYNC_LINK_OPTS_CPP,
    deps = [
        ":nsync_cpp",
        ":nsync_test_lib_cpp",
    ],
)

cc_test(
    name = "once_cpp_test",
    size = "small",
    srcs = ["testing/once_test.c"],
    copts = NSYNC_OPTS_CPP,
    linkopts = NSYNC_LINK_OPTS_CPP,
    deps = [
        ":nsync_cpp",
        ":nsync_test_lib_cpp",
    ],
)

cc_test(
    name = "pingpong_cpp_test",
    size = "small",
    srcs = ["testing/pingpong_test.c"],
    copts = NSYNC_OPTS_CPP,
    linkopts = NSYNC_LINK_OPTS_CPP,
    deps = [
        ":nsync_cpp",
        ":nsync_test_lib_cpp",
    ],
)

cc_test(
    name = "wait_cpp_test",
    size = "small",
    srcs = ["testing/wait_test.c"],
    copts = NSYNC_OPTS_CPP,
    linkopts = NSYNC_LINK_OPTS_CPP,
    deps = [
        ":nsync_cpp",
        ":nsync_test_lib_cpp",
    ],
)
