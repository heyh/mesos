# Licensed to the Apache Software Foundation (ASF) under one
# or more contributor license agreements.  See the NOTICE file
# distributed with this work for additional information
# regarding copyright ownership.  The ASF licenses this file
# to you under the Apache License, Version 2.0 (the
# "License"); you may not use this file except in compliance
# with the License.  You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

# OpenSSL: Cryptography and SSL/TLS Toolkit.
# https://www.openssl.org
############################################
if (ENABLE_SSL)
  find_package(OpenSSL REQUIRED)
endif ()


# SOURCE FILES FOR THE PROCESS LIBRARY.
#######################################
set(PROCESS_SRC
  authenticator.cpp
  authenticator_manager.cpp
  authenticator_manager.hpp
  clock.cpp
  config.hpp
  decoder.hpp
  encoder.hpp
  event_loop.hpp
  event_queue.hpp
  firewall.cpp
  gate.hpp
  gtest_constants.cpp
  help.cpp
  http.cpp
  http_proxy.cpp
  http_proxy.hpp
  io.cpp
  latch.cpp
  logging.cpp
  memory_profiler.cpp
  memory_profiler.hpp
  metrics/metrics.cpp
  mime.cpp
  pid.cpp
  poll_socket.cpp
  poll_socket.hpp
  process.cpp
  process_reference.hpp
  profiler.cpp
  reap.cpp
  run_queue.hpp
  semaphore.hpp
  socket.cpp
  socket_manager.hpp
  subprocess.cpp
  time.cpp)

if (WIN32)
  list(APPEND PROCESS_SRC
    subprocess_windows.cpp)
else ()
  list(APPEND PROCESS_SRC
    subprocess_posix.cpp)
endif ()

if (ENABLE_GRPC)
  list(APPEND PROCESS_SRC
    grpc.cpp)
endif ()

if (ENABLE_LIBEVENT)
  list(APPEND PROCESS_SRC
    libevent.hpp
    libevent.cpp
    libevent_poll.cpp)
else ()
  list(APPEND PROCESS_SRC
    libev.hpp
    libev.cpp
    libev_poll.cpp)
endif ()

if (ENABLE_SSL)
  list(APPEND PROCESS_SRC
    jwt.cpp
    jwt_authenticator.cpp
    openssl.cpp
    ssl/utilities.cpp)

  if (ENABLE_LIBEVENT)
    list(APPEND PROCESS_SRC
      libevent_ssl_socket.cpp)
  endif ()
endif ()

add_library(process ${PROCESS_SRC})

target_link_libraries(
  process PUBLIC
  stout
  http_parser
  $<$<BOOL:${ENABLE_GRPC}>:grpc>
  $<$<BOOL:${ENABLE_SSL}>:OpenSSL::SSL>
  $<$<BOOL:${ENABLE_SSL}>:OpenSSL::Crypto>)

target_link_libraries(
  process PRIVATE
  concurrentqueue
  $<IF:$<BOOL:${ENABLE_LIBEVENT}>,libevent,libev>)

target_compile_definitions(
  process PRIVATE
  $<$<BOOL:${ENABLE_LOCK_FREE_RUN_QUEUE}>:LOCK_FREE_RUN_QUEUE>
  $<$<BOOL:${ENABLE_LOCK_FREE_EVENT_QUEUE}>:LOCK_FREE_EVENT_QUEUE>
  $<$<BOOL:${ENABLE_LAST_IN_FIRST_OUT_FIXED_SIZE_SEMAPHORE}>:LAST_IN_FIRST_OUT_FIXED_SIZE_SEMAPHORE>)

if (LINUX)
  target_compile_definitions(process PRIVATE LIBPROCESS_ALLOW_JEMALLOC)
endif ()

target_include_directories(process PUBLIC ../include)
target_include_directories(process PRIVATE .)
