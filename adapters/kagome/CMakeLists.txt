# Copyright (c) 2019-2021 Web 3.0 Technologies Foundation
#
# This file is part of the Polkadot Test Suite.
#
# The Polkadot Test Suite is free software: you can redistribute it and/or
# modify it under the terms of the GNU General Public License as published by
# the Free Software Foundation, either version 3 of the License, or
# (at your option) any later version.
#
# The Polkadot Test Suite is distributed in the hope that it will be useful,
# but WITHOUT ANY WARRANTY; without even the implied warranty of
# MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
# GNU General Public License for more details.
#
# You should have received a copy of the GNU General Public License
# along with the Polkadot Test Suite. If not, see
# <https://www.gnu.org/licenses/>.

cmake_minimum_required(VERSION 3.12)


# Setup toolchain (required by kagome and libp2p)
set(CMAKE_TOOLCHAIN_FILE
  "${CMAKE_SOURCE_DIR}/cmake/Toolchain.cmake"
  CACHE FILEPATH "Default toolchain"
)

# Propagate CMake build type to dependencies
set(
  HUNTER_CONFIGURATION_TYPES "${CMAKE_BUILD_TYPE}"
  CACHE STRING "Hunter configuration type"
)

# Enable upload to binary cache if username and password are supplied
if(DEFINED ENV{HUNTER_UPLOAD_USERNAME} AND DEFINED ENV{HUNTER_UPLOAD_PASSWORD})
  set(HUNTER_RUN_UPLOAD YES CACHE BOOL "Upload to binary cache")
else()
  set(HUNTER_RUN_UPLOAD NO CACHE BOOL "Upload to binary cache")
endif ()

set(
  HUNTER_PASSWORDS_PATH "${CMAKE_SOURCE_DIR}/cmake/HunterPasswords.cmake"
  CACHE FILEPATH "Hunter passwords files"
)

message(STATUS "Upload to binary cache: ${HUNTER_RUN_UPLOAD}")

# Setup binary cache and enable if upload is enabled
set(
  HUNTER_USE_CACHE_SERVERS ${HUNTER_RUN_UPLOAD}
  CACHE STRING "Binary cache server"
)

set(
  HUNTER_CACHE_SERVERS "https://github.com/w3f/hunter-binary-cache"
  CACHE STRING "Binary cache server"
)

message(STATUS "Download from binary cache: ${HUNTER_USE_CACHE_SERVERS}")

# Setup hunter
include(cmake/HunterGate.cmake)
HunterGate(
  URL "https://github.com/soramitsu/soramitsu-hunter/archive/tags/v0.23.257-soramitsu17.tar.gz"
  SHA1 "c7ccd337314b27485b75d0f0f5d5b42e7e3c2629"
  FILEPATH "${CMAKE_SOURCE_DIR}/cmake/HunterConfig.cmake"
)

set(CMAKE_FIND_PACKAGE_PREFER_CONFIG ${HUNTER_ENABLED})


# General config
project(kagome-adapter LANGUAGES CXX C)

include(cmake/dependencies.cmake)

# Compile and link tester
add_executable(kagome-adapter
  src/main.cpp
  src/assert.cpp
  src/scale_codec.cpp
  src/state_trie.cpp
  src/host_api.cpp
  src/host_api/helpers.cpp
  src/host_api/allocator.cpp
  src/host_api/crypto.cpp
  src/host_api/hashing.cpp
  src/host_api/trie.cpp
  src/host_api/storage.cpp
  src/host_api/child_storage.cpp
)

# FIXME This should not be necessary, but kagome lacks a modern package config
target_include_directories(kagome-adapter PRIVATE ${kagome_INCLUDE_DIRS})

target_link_libraries(kagome-adapter
  Boost::program_options
  yaml-cpp
  ${kagome_LIBRARIES}
  kagome::binaryen_wasm_memory_factory
)


# Install result
include(GNUInstallDirs)

install(TARGETS kagome-adapter
  RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
)
