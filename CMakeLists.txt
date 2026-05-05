cmake_minimum_required(VERSION 3.18)

project(Timer)

set(CMAKE_EXPORT_COMPILE_COMMANDS 1)

if(CPPTEST_COVERAGE)
  include(cpptest-coverage.cmake)
endif()

add_compile_definitions(STDINPUT)

add_executable(timer
 clock.c
 driver.c
 timer.c
 stdinout.c)
