cmake_minimum_required (VERSION 2.6)

project (nlvm)
#set (nlvm_VERSION "0.1.0")
set (nlvm_VERSION_MAJOR "0")
set (nlvm_VERSION_MINOR "1")
set (nlvm_VERSION_PATCH "1")

# Define the build time
execute_process(COMMAND uname -a
  OUTPUT_VARIABLE _output OUTPUT_STRIP_TRAILING_WHITESPACE)
if(WIN32)
 execute_process(COMMAND cmd /C win_date.bat
    OUTPUT_VARIABLE BUILD_TIME)
else(WIN32)
  execute_process(COMMAND date "+%Y-%m-%d-%H:%M"
    OUTPUT_VARIABLE BUILD_TIME)
endif(WIN32)
string(STRIP ${BUILD_TIME} BUILD_TIME)

add_executable(nlvm_test_basic tests/basic.c)


# testing
include(CTest)
set (CTEST_PROJECT_NAME "nlvm")
# does the application run
add_test (BasicTest nlvm_test_basic)

# does it sqrt of 25
#add_test (TutorialComp25 Tutorial 25)
#set_tests_properties (TutorialComp25 PROPERTIES PASS_REGULAR_EXPRESSION "25 is 5")


# build a CPack driven installer package
include (InstallRequiredSystemLibraries)
set (CPACK_PACKAGE_VENDOR "Hirochika Asai")
set (CPACK_RESOURCE_FILE_LICENSE
     "${CMAKE_CURRENT_SOURCE_DIR}/License.txt")
set (CPACK_PACKAGE_VERSION_MAJOR "${nlvm_VERSION_MAJOR}")
set (CPACK_PACKAGE_VERSION_MINOR "${nlvm_VERSION_MINOR}")
set (CPACK_PACKAGE_VERSION_PATCH "${nlvm_VERSION_PATCH}")
#set (CPACK_SOURCE_PACKAGE_FILE_NAME "nlvm-0.1.1")
include (CPack)

add_custom_target(dist COMMAND ${CMAKE_MAKE_PROGRAM} package_source)
set (CPACK_IGNORE_FILES "/CVS/;/\\.svn/;/\\.bzr/;/\\.hg/;/\\.git/;\\.swp\$;\\.#;/#;/build/")
add_custom_target(git-dist
  COMMAND cmake ..
  COMMAND make dist
  WORKING_DIRECTORY .dist/build
  DEPENDS git-dist2)
add_custom_target(git-dist2
  COMMAND rm -rf .dist
  COMMAND git clone . .dist
  COMMAND mkdir .dist/build
  #COMMAND make git-dist2
  WORKING_DIRECTORY .)
