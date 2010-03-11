# Make sure the compiler can find include files from our cminpack library.
include_directories (${CMINPACK_SOURCE_DIR})

# Make sure the linker can find the cminpack library once it is built.
link_directories (${CMINPACK_BINARY_DIR}/Hello)

set (FPGM tchkder thybrd thybrd1 thybrj thybrj1 tlmder tlmder1 tlmdif tlmdif1 tlmstr tlmstr1)
set (CPGM ${FPGM} tfdjac2)

# cmpfiles could be used by runtest.cmake... for now it's unused
add_executable (cmpfiles cmpfiles.c)

# inspired by http://www.netlib.org/clapack/clapack-3.2.1-CMAKE/TESTING/CMakeLists.txt
# except that here we have to compare the output to a reference
macro(add_minpack_test testname reference)
  set(TEST_OUTPUT "${CMINPACK_BINARY_DIR}/examples/${testname}.out")
  set(TEST_REFERENCE "${CMINPACK_SOURCE_DIR}/examples/ref/${reference}.ref")
  get_target_property(TEST_LOC ${testname} LOCATION)
  add_test(${testname} "${CMAKE_COMMAND}"
    -DTEST=${TEST_LOC}
    -DOUTPUT=${TEST_OUTPUT} 
    -DREFERENCE=${TEST_REFERENCE} 
    -DINTDIR=${CMAKE_CFG_INTDIR}
    -P "${CMINPACK_SOURCE_DIR}/examples/runtest.cmake")
endmacro(add_minpack_test)


enable_language (Fortran)
if (CMAKE_Fortran_COMPILER_WORKS)
  foreach (FPGM_item ${FPGM})
    list (APPEND FPGM_SRCS ${CMINPACK_SOURCE_DIR}/examples/${FPGM_item}.f)
  endforeach (FPGM_item ${FPGM})

  # add the executable that will do the generation
  add_executable (genf77tests genf77tests.c)
  get_target_property (GENF77TESTS_EXE genf77tests LOCATION)
  add_custom_command (
    OUTPUT ${FPGM_SRCS}
    COMMAND ${GENF77TESTS_EXE} ${CMINPACK_SOURCE_DIR}/doc/minpack-documentation.txt
    WORKING_DIRECTORY ${CMINPACK_SOURCE_DIR}/examples
    DEPENDS genf77tests
    MAIN_DEPENDENCY ${CMINPACK_SOURCE_DIR}/doc/minpack-documentation.txt
    )
  foreach (source ${FPGM})
    add_executable (${source} ${source}.f)
    target_link_libraries (${source} cminpack) 
    add_minpack_test(${source} ${source})
  endforeach (source)
endif (CMAKE_Fortran_COMPILER_WORKS)

foreach (source ${CPGM})
  add_executable (${source}_ ${source}_.c)
  target_link_libraries (${source}_ cminpack)
  add_minpack_test(${source}_ ${source}c)
  add_executable (${source}c ${source}c.c)
  target_link_libraries (${source}c cminpack)
  add_minpack_test(${source}c ${source}c)
endforeach(source)