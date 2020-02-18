#
# Copyright 2020 Staysail Systems, Inc. <info@staysail.tech>
#
# This software is distributed under the terms of the GPLv3, a
# copy of which should be located as the file COPYING in the
# same directory as this file.  A copy of the license may also be
# found online at https://opensource.org/licenses/GPL-3.0
#
# Commercial users may obtain a commercial license without GPLv3
# restrictions.  Please contact info@staysail.tech for details.
#
# SPDX-License-Identifier: GPL-3.0-only
#

cmake_minimum_required(VERSION 3.10)

project(nng-wolfssl C)

include(CheckLibraryExists)

set(CMAKE_MODULE_PATH ${PROJECT_SOURCE_DIR}/cmake)

message(WARNING "
        ============================================================
        wolfSSL NNG Module is GPLv3.0
        ============================================================
        Linking against wolfSSL changes license terms (GPL v3.0)!
        Consult a lawyer and the license files for details.
        ============================================================")

add_library(nng-wolfssl OBJECT wolfssl.c)

if (TARGET wolfssl)
    target_link_libraries(nng-wolfssl nng-wolfssl)
    # presumably the includes are already set...
else ()
    find_package(wolfSSL REQUIRED)
    set(_LIBS ${WOLFSSL_LIBRARIES})
    set(_INCS ${WOLFSSL_INCLUDE_DIR})
    target_link_libraries(nng-wolfssl ${WOLFSSL_LIBRARIES})
    target_include_directories(nng-wolfssl PRIVATE ${WOLFSS_INCLUDE_DIR})
endif ()

# wolfss/options.h in theory has defines already for these checks, but they
# are poorly documented, and might not be reliable -- so we check ourselves.
check_library_exists(wolfssl wolfSSL_CTX_LoadCRLBuffer "" NNG_WOLFSSL_HAVE_CRL)
check_library_exists(wolfssl wolfSSL_get_verify_result "" NNG_WOLFSSL_HAVE_VERIFY)
check_library_exists(wolfssl wolfSSL_CTX_set_default_passwd_cb "" NNG_WOLFSSL_HAVE_PASSWORD)

macro(nng_wolfssl_define DEF)
    if (${DEF})
        target_compile_definitions(nng-wolfssl PRIVATE ${DEF})
    endif ()
endmacro()

# We want this file to be added into the nng project.
# This particular CMakeLists.txt only works in the context of
# of a the larger NNG project.

nng_wolfssl_define(NNG_WOLFSSL_HAVE_CRL)
nng_wolfssl_define(NNG_WOLFSSL_HAVE_VERIFY)
nng_wolfssl_define(NNG_WOLFSSL_HAVE_PASSWORD)

if (TARGET nng)
    target_compile_definitions(nng PRIVATE NNG_TLS_ENGINE_INIT=nng_tls_engine_init_wolf)
    target_compile_definitions(nng PRIVATE NNG_TLS_ENGINE_FINI=nng_tls_engine_fini_wolf)
    target_link_libraries(nng PRIVATE nng-wolfssl)
endif()

if (TARGET nng_testlib)
    target_compile_definitions(nng_testlib PRIVATE NNG_TLS_ENGINE_INIT=nng_tls_engine_init_wolf)
    target_compile_definitions(nng_testlib PRIVATE NNG_TLS_ENGINE_FINI=nng_tls_engine_fini_wolf)
    target_link_libraries(nng_testlib PRIVATE nng-wolfssl)
endif ()
