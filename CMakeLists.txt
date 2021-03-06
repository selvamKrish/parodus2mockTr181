#   Copyright 2010-2018 Comcast Cable Communications Management, LLC
#
#   Licensed under the Apache License, Version 2.0 (the "License");
#   you may not use this file except in compliance with the License.
#   You may obtain a copy of the License at
#
#       http://www.apache.org/licenses/LICENSE-2.0
#
#   Unless required by applicable law or agreed to in writing, software
#   distributed under the License is distributed on an "AS IS" BASIS,
#   WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
#   See the License for the specific language governing permissions and
#   limitations under the License.

cmake_minimum_required(VERSION 2.8.7)

project(parodus2mockTr181)
include(ExternalProject)

# Compile options/flags
#-------------------------------------------------------------------------------
add_definitions(-std=c99)
set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -D_GNU_SOURCE -Werror -Wall")
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -D_GNU_SOURCE -Werror -Wall")
set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -D_GNU_SOURCE -DNOPOLL_LOGGER ")

set(INSTALL_DIR ${CMAKE_CURRENT_BINARY_DIR}/_install)
set(PREFIX_DIR ${CMAKE_CURRENT_BINARY_DIR}/_prefix)
set(INCLUDE_DIR ${INSTALL_DIR}/include)
set(LIBRARY_DIR ${INSTALL_DIR}/lib${LIB_SUFFIX})
set(LIBRARY_DIR64 ${INSTALL_DIR}/lib64)
set(COMMON_LIBRARY_DIR ${INSTALL_DIR}/lib/${CMAKE_LIBRARY_ARCHITECTURE})

# Includes
#--------------------------------------------------------------------------------
include_directories(${INCLUDE_DIR}
                    ${INCLUDE_DIR}/cjson
                    ${INCLUDE_DIR}/msgpack
                    ${INCLUDE_DIR}/trower-base64
                    ${INCLUDE_DIR}/wrp-c
                    ${INCLUDE_DIR}/wdmp-c                   
                    ${INCLUDE_DIR}/libparodus
                    ${INCLUDE_DIR}/cimplog
                   )

include_directories(${INCLUDE_DIR}/cJSON
                    ${CURL_INCLUDE_DIRS})
include_directories (SYSTEM /usr/include)


# pthread external dependency
#-------------------------------------------------------------------------------
find_package (Threads)

if (NOT BUILD_YOCTO)

# base64 external dependency
#-------------------------------------------------------------------------------
ExternalProject_Add(trower-base64
    PREFIX ${CMAKE_CURRENT_BINARY_DIR}/_prefix/trower-base64
    GIT_REPOSITORY https://github.com/Comcast/trower-base64.git
    GIT_TAG "master"
    CMAKE_ARGS += -DCMAKE_INSTALL_PREFIX=${INSTALL_DIR}
)
add_library(libtrower-base64 STATIC SHARED IMPORTED)
add_dependencies(libtrower-base64 trower-base64)

# nanoMsg external dependency
#-------------------------------------------------------------------------------
ExternalProject_Add(nanomsg
    PREFIX ${CMAKE_CURRENT_BINARY_DIR}/_prefix/nanomsg
    GIT_REPOSITORY https://github.com/nanomsg/nanomsg.git
    #GIT_TAG "096998834451219ee7813d8977f6a4027b0ccb43"
    CMAKE_ARGS += -DCMAKE_INSTALL_PREFIX=${INSTALL_DIR}
)
add_library(libnanomsg STATIC SHARED IMPORTED)
add_dependencies(libnanomsg nanomsg)

# cJSON external dependency
#-------------------------------------------------------------------------------
ExternalProject_Add(cJSON
    #PREFIX ${PREFIX_DIR}/cJSON
    PREFIX ${CMAKE_CURRENT_BINARY_DIR}/_prefix/cJSON
    GIT_REPOSITORY https://github.com/DaveGamble/cJSON.git
    #GIT_TAG "master"
    GIT_TAG "aafb64a1c549b7b927e339df6d35b1d5059dc235"
    CMAKE_ARGS += -DCMAKE_INSTALL_PREFIX=${INSTALL_DIR}
)
add_library(libcJSON STATIC SHARED IMPORTED)
add_dependencies(libcJSON cJSON)

# msgpack-c external dependency
#-------------------------------------------------------------------------------
ExternalProject_Add(msgpack
    PREFIX ${CMAKE_CURRENT_BINARY_DIR}/_prefix/msgpack
    GIT_REPOSITORY https://github.com/msgpack/msgpack-c.git
    ## GIT_TAG "c6e6dbc608366090c12b142b3832604e6aa12f54"
    GIT_TAG "7a98138f27f27290e680bf8fbf1f8d1b089bf138"
    CMAKE_ARGS += -DCMAKE_INSTALL_PREFIX=${INSTALL_DIR}
                  -DMSGPACK_ENABLE_CXX=OFF
                  -DMSGPACK_BUILD_EXAMPLES=OFF
)
add_library(libmsgpack STATIC SHARED IMPORTED)
add_dependencies(libmsgpack msgpack)

# cimplog external dependency
#-------------------------------------------------------------------------------
ExternalProject_Add(cimplog
    PREFIX ${CMAKE_CURRENT_BINARY_DIR}/_prefix/cimplog
    GIT_REPOSITORY https://github.com/Comcast/cimplog.git
    GIT_TAG "master"
    CMAKE_ARGS += -DCMAKE_INSTALL_PREFIX=${INSTALL_DIR}
)
add_library(libcimplog STATIC SHARED IMPORTED)
add_dependencies(libcimplog cimplog)

# wrp-c external dependency
#-------------------------------------------------------------------------------
ExternalProject_Add(wrp-c
    DEPENDS trower-base64 msgpack cimplog
    PREFIX ${CMAKE_CURRENT_BINARY_DIR}/_prefix/wrp-c
    GIT_REPOSITORY https://github.com/Comcast/wrp-c.git
    GIT_TAG "master"
    CMAKE_ARGS += -DCMAKE_INSTALL_PREFIX=${INSTALL_DIR}
                  -DMSGPACK_ENABLE_CXX=OFF
                  -DMSGPACK_BUILD_EXAMPLES=OFF
                  -DBUILD_TESTING=OFF
                  -DMAIN_PROJ_BUILD=ON
                  -DMAIN_PROJ_LIB_PATH=${LIBRARY_DIR}
                  -DMAIN_PROJ_INCLUDE_PATH=${INCLUDE_DIR}
)
add_library(libwrp-c STATIC SHARED IMPORTED)
add_dependencies(libwrp-c wrp-c)

# wdmp-c external dependency
#-------------------------------------------------------------------------------
ExternalProject_Add(wdmp-c
    PREFIX ${CMAKE_CURRENT_BINARY_DIR}/_prefix/wdmp-c
    GIT_REPOSITORY https://github.com/Comcast/wdmp-c.git
    GIT_TAG "master"
    CMAKE_ARGS += -DCMAKE_INSTALL_PREFIX=${INSTALL_DIR}
                  -DBUILD_TESTING=OFF
)
add_library(libwdmp-c STATIC SHARED IMPORTED)
add_dependencies(libwdmp-c wdmp-c)

# libparodus external dependency
#-------------------------------------------------------------------------------
ExternalProject_Add(libparodus
    DEPENDS trower-base64 msgpack nanomsg wrp-c
    PREFIX ${CMAKE_CURRENT_BINARY_DIR}/_prefix/libparodus
    GIT_REPOSITORY https://github.com/Comcast/libparodus.git
    #    GIT_TAG "7de6c7c43f8d773713aad6f0c5b4e5a8d2809b1a"
    GIT_TAG "master"
    CMAKE_ARGS += -DCMAKE_INSTALL_PREFIX=${INSTALL_DIR} -DBUILD_TESTING=OFF
    		  -DMAIN_PROJ_BUILD=ON
    		  -DMAIN_PROJ_LIB_PATH=${LIBRARY_DIR}
		  -DMAIN_PROJ_LIB64_PATH=${LIBRARY_DIR64}
		  -DMAIN_PROJ_COMMON_PATH=${COMMON_LIBRARY_DIR}
    		  -DMAIN_PROJ_INCLUDE_PATH=${INCLUDE_DIR}
)
add_library(liblibparodus STATIC SHARED IMPORTED)
add_dependencies(liblibparodus libparodus)


endif ()

enable_testing()

link_directories ( ${LIBRARY_DIR} ${COMMON_LIBRARY_DIR} ${LIBRARY_DIR64} )
add_subdirectory(src)
add_subdirectory(tests)
