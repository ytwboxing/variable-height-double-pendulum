# adding support for checking the tests with valgrind
if(RUN_VALGRIND_TESTS)
     find_package(Valgrind REQUIRED)
     if(VALGRIND_FOUND)
         set(CTEST_MEMORYCHECK_COMMAND ${VALGRIND_PROGRAM})
         set(MEMORYCHECK_COMMAND ${VALGRIND_PROGRAM})
         if (APPLE)
             set(MEMORYCHECK_SUPPRESSIONS "--suppressions=${PROJECT_SOURCE_DIR}/cmake/valgrind-macos.supp")
         else ()
             set(MEMORYCHECK_SUPPRESSIONS "")
         endif ()
         set(MEMORYCHECK_COMMAND_OPTIONS "--leak-check=full --error-exitcode=1 ${MEMORYCHECK_SUPPRESSIONS}"  CACHE STRING "Options to pass to the memory checker")
         mark_as_advanced(MEMORYCHECK_COMMAND_OPTIONS)
         set(MEMCHECK_COMMAND_COMPLETE "${MEMORYCHECK_COMMAND} ${MEMORYCHECK_COMMAND_OPTIONS}")
         separate_arguments(MEMCHECK_COMMAND_COMPLETE)
     endif()
endif()

# TODO avoid using directly the source files to test it
set(testsrc ResponderTest.cpp)
set(testbinary ResponderUnitTest)
set(testname   UnitTestResponder)
add_executable(${testbinary} ${testsrc})
target_link_libraries(${testbinary} ${PROJECT_NAME}_responderLibrary)
target_include_directories(${testbinary} PUBLIC "$<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/../src/include>")
ament_target_dependencies(${testbinary} rclcpp controller_msgs)
add_test(NAME ${testname} COMMAND ${testbinary})

if(RUN_VALGRIND_TESTS)
    add_test(NAME memcheck_${testname} COMMAND ${MEMCHECK_COMMAND_COMPLETE} $<TARGET_FILE:${testbinary}>)
endif()
