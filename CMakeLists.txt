cmake_minimum_required(VERSION 3.16)
project(AMD_IBS_Toolkit)

set(BASE_DIR ${CMAKE_CURRENT_SOURCE_DIR})

add_subdirectory(${BASE_DIR}/driver ibs-driver)