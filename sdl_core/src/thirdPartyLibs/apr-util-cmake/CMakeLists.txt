# Copyright (c) 2014, Ford Motor Company
# All rights reserved.
#
# Redistribution and use in source and binary forms, with or without
# modification, are permitted provided that the following conditions are met:
#
# Redistributions of source code must retain the above copyright notice, this
# list of conditions and the following disclaimer.
#
# Redistributions in binary form must reproduce the above copyright notice,
# this list of conditions and the following
# disclaimer in the documentation and/or other materials provided with the
# distribution.
#
# Neither the name of the Ford Motor Company nor the names of its contributors
# may be used to endorse or promote products derived from this software
# without specific prior written permission.
#
# THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS"
# AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE
# IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE
# ARE DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT HOLDER OR CONTRIBUTORS BE
# LIABLE FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR
# CONSEQUENTIAL DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF
# SUBSTITUTE GOODS OR SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS
# INTERRUPTION) HOWEVER CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN
# CONTRACT, STRICT LIABILITY, OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE)
# ARISING IN ANY WAY OUT OF THE USE OF THIS SOFTWARE, EVEN IF ADVISED OF THE
# POSSIBILITY OF SUCH DAMAGE.

set(APR_UTIL_SOURCE_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/../apr-util-1.5.3)
set(APR_UTIL_BUILD_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}/../apr-util-build)
if(MODIFY_FUNCTION_FLAGS)
set(APR_UTIL_LIBS_DIRECTORY ${APR_UTIL_BUILD_DIRECTORY}/.libs)
set(APR_UTIL_LIBS_DIRECTORY ${APR_UTIL_LIBS_DIRECTORY} PARENT_SCOPE)
MESSAGE(STATUS "apr_util APR_UTIL_LIBS_DIRECTORY ${APR_UTIL_LIBS_DIRECTORY}")
endif()

file(MAKE_DIRECTORY
  ${APR_UTIL_BUILD_DIRECTORY}
)

set(COMMON_CONFIGURE_FLAGS "--with-apr=../apr-build" "--with-expat=../expat-2.1.0")

if(CMAKE_SYSTEM_NAME STREQUAL "QNX")
  set(CONFIGURE_FLAGS "--host=${CMAKE_SYSTEM_PROCESSOR}-nto-qnx" ${COMMON_CONFIGURE_FLAGS})
elseif(CMAKE_SYSTEM_NAME STREQUAL "Android")
  MESSAGE(STATUS "apr-util --host=arm-linux-androideabi")
  set(CONFIGURE_FLAGS "--host=arm-linux-androideabi" ${COMMON_CONFIGURE_FLAGS})
else()
  set(CONFIGURE_FLAGS ${COMMON_CONFIGURE_FLAGS})
endif()


add_custom_command(OUTPUT ${APR_UTIL_BUILD_DIRECTORY}/Makefile
  COMMAND CC=${CMAKE_C_COMPILER} ${APR_UTIL_SOURCE_DIRECTORY}/configure ${CONFIGURE_FLAGS}
  DEPENDS libapr-1
  DEPENDS expat
  WORKING_DIRECTORY ${APR_UTIL_BUILD_DIRECTORY}
)

add_custom_target(apr-util ALL make
  DEPENDS ${APR_UTIL_BUILD_DIRECTORY}/Makefile
  WORKING_DIRECTORY ${APR_UTIL_BUILD_DIRECTORY}
)

if(MODIFY_FUNCTION_FLAGS)
if(CMAKE_SYSTEM_NAME STREQUAL "Windows")
execute_process(
  COMMAND cp ${CMAKE_CURRENT_BINARY_DIR}/../expat-2.1.0/${CMAKE_BUILD_TYPE}/expat.dll ${CMAKE_CURRENT_BINARY_DIR}
)
elseif(CMAKE_SYSTEM_NAME STREQUAL "Darwin")

else()
execute_process(
  COMMAND cp ${CMAKE_CURRENT_BINARY_DIR}/../expat-2.1.0/libexpat.so ${CMAKE_CURRENT_BINARY_DIR}
)
endif()
endif()

install(
  DIRECTORY ${APR_UTIL_BUILD_DIRECTORY}/.libs/
  DESTINATION ${3RD_PARTY_INSTALL_PREFIX}/lib
  USE_SOURCE_PERMISSIONS
  FILES_MATCHING PATTERN libaprutil-1.so*
)
