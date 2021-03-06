#-----------------------------------------------------------------------------
#
#  CMake Config
#
#  Osmium Tool Tests
#
#-----------------------------------------------------------------------------

include_directories(include)
include_directories(../src)
include_directories(../include)

file(GLOB ALL_SETUP_TESTS */test_setup.cpp)
file(GLOB ALL_COMMANDS ../src/command_*.cpp ../src/io.cpp ../src/cmd_factory.cpp)
add_executable(unit_tests unit_tests.cpp ${ALL_COMMANDS} ${ALL_SETUP_TESTS})
target_link_libraries(unit_tests ${Boost_LIBRARIES} ${OSMIUM_LIBRARIES})
add_test(NAME unit_tests COMMAND unit_tests)


#-----------------------------------------------------------------------------

function(do_test _name _command _regex)
    separate_arguments(_command)
    add_test(NAME ${_name} COMMAND ${_command})
    set_tests_properties(${_name} PROPERTIES
                         ENVIRONMENT "MANPATH=${PROJECT_BINARY_DIR}/man"
                         PASS_REGULAR_EXPRESSION ${_regex}
    )
endfunction()

#-----------------------------------------------------------------------------

function(check_output _dir _name _command _reference)
    set(_cmd "${PROJECT_BINARY_DIR}/src/osmium ${_command}")
    add_test(
        NAME "${_dir}-${_name}"
        COMMAND ${CMAKE_COMMAND}
        -Dcmd=${_cmd}
        -Ddir=${PROJECT_SOURCE_DIR}/test
        -Dreference=${PROJECT_SOURCE_DIR}/test/${_reference}
        -Doutput=${PROJECT_BINARY_DIR}/test/${_dir}/cmd-output-${_name}
        -P ${CMAKE_SOURCE_DIR}/cmake/run_test_compare_output.cmake
    )
endfunction()

function(check_output2 _dir _name _command1 _command2 _reference)
    set(_cmd1 "${PROJECT_BINARY_DIR}/src/osmium ${_command1}")
    set(_cmd2 "${PROJECT_BINARY_DIR}/src/osmium ${_command2}")
    add_test(
        NAME "${_dir}-${_name}"
        COMMAND ${CMAKE_COMMAND}
        -Dcmd=${_cmd1}
        -Dcmd2=${_cmd2}
        -Ddir=${PROJECT_SOURCE_DIR}/test
        -Dreference=${PROJECT_SOURCE_DIR}/test/${_reference}
        -Doutput=${PROJECT_BINARY_DIR}/test/${_dir}/cmd-output-${_name}
        -P ${CMAKE_SOURCE_DIR}/cmake/run_test_compare_output.cmake
    )
endfunction()


#-----------------------------------------------------------------------------
#
#  Configure tests in all subdirectories
#
#-----------------------------------------------------------------------------

file(GLOB _testcmakes "test" "*/CMakeLists.txt")

foreach(_tc IN LISTS _testcmakes)
    string(REPLACE "/CMakeLists.txt" "" _dir ${_tc})
    message(STATUS "Adding tests in ${_dir}")
    add_subdirectory(${_dir})
endforeach()


#-----------------------------------------------------------------------------
