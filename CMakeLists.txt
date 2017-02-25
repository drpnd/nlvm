cmake_minimum_required (VERSION 2.6)

project (nlvm)
set (nlvm_VERSION "current")
#set (nlvm_VERSION_MAJOR "0")
#set (nlvm_VERSION_MINOR "1")
#set (nlvm_VERSION_PATCH "0")

# Define the build time
execute_process (COMMAND uname -a
  OUTPUT_VARIABLE _output OUTPUT_STRIP_TRAILING_WHITESPACE)
if(WIN32)
 execute_process (COMMAND cmd /C win_date.bat
    OUTPUT_VARIABLE BUILD_DATETIME)
else(WIN32)
  execute_process (COMMAND date "+%Y-%m-%d %H:%M:%S"
    OUTPUT_VARIABLE BUILD_DATETIME)
endif(WIN32)
string(STRIP ${BUILD_DATETIME} BUILD_DATETIME)

execute_process (COMMAND git symbolic-ref --short HEAD
  OUTPUT_VARIABLE GIT_CURRENT_BRANCH)

add_definitions (-DBUILD_DATETIME=\"${BUILD_DATETIME}\")

set (CMAKE_BINARY_DIR "build")
set (CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR})

# nlang
add_executable(nlang nlang/main.c)

# vm runtime
add_executable(nlvm vm/main.c)

# testing
include(CTest)
set (CTEST_PROJECT_NAME "nlvm")
# does the application run
add_executable(nlvm_test_basic tests/basic.c)
add_test (BasicTest ${CMAKE_RUNTIME_OUTPUT_DIRECTORY}/nlvm_test_basic)
add_test (BasicTestArg ${CMAKE_RUNTIME_OUTPUT_DIRECTORY}/nlvm_test_basic)


# build a CPack driven installer package
include (InstallRequiredSystemLibraries)
set (CPACK_PACKAGE_VENDOR "Hirochika Asai")
set (CPACK_RESOURCE_FILE_LICENSE
     "${CMAKE_CURRENT_SOURCE_DIR}/License.txt")
if(nlvm_VERSION)
  set (CPACK_PACKAGE_VERSION "${nlvm_VERSION}")
  #set (CPACK_SOURCE_PACKAGE_FILE_NAME "nlvm-${nlvm_VERSION}")
else(nlvm_VERSION)
  set (CPACK_PACKAGE_VERSION_MAJOR "${nlvm_VERSION_MAJOR}")
  set (CPACK_PACKAGE_VERSION_MINOR "${nlvm_VERSION_MINOR}")
  set (CPACK_PACKAGE_VERSION_PATCH "${nlvm_VERSION_PATCH}")
endif(nlvm_VERSION)
include (CPack)

add_custom_target(dist COMMAND ${CMAKE_MAKE_PROGRAM} package_source)
set (CPACK_IGNORE_FILES "/CVS/;/\\.svn/;/\\.bzr/;/\\.hg/;/\\.git/;\\.swp\$;\\.#;/#;/build/")
add_custom_target(git-dist
  COMMAND cmake ..
  COMMAND make dist
  COMMAND cp nlvm-*-Source.tar.* ../../
  WORKING_DIRECTORY .working/build
  DEPENDS git-working-prepare)

add_custom_target(git-working-prepare
  COMMAND rm -rf .working
  COMMAND git clone . .working -b ${GIT_CURRENT_BRANCH}
  COMMAND mkdir .working/build
  WORKING_DIRECTORY .)

add_custom_target(git-test
  COMMAND cmake ..
  COMMAND make
  COMMAND make test
  WORKING_DIRECTORY .working/build
  DEPENDS git-working-prepare)
