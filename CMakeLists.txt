esma_set_this()

# library to archive (libneural.a)
set (SRCS 
   src/lib/mod_activation.F90
   src/lib/mod_io.F90
   src/lib/mod_kinds.F90
   src/lib/mod_layer.F90
   src/lib/mod_dense_layer.F90
   src/lib/mod_dropout_layer.F90
   src/lib/mod_batchnorm_layer.F90
   src/lib/mod_mnist.F90
   src/lib/mod_network.F90
   src/lib/mod_ensemble.F90
   src/lib/mod_random.F90
   )

esma_add_library(${this}
   SRCS ${SRCS}
   )

target_link_libraries(${this} PRIVATE OpenMP::OpenMP_Fortran)

# handle integer size
if(FKB_INT)
   message(STATUS "Configuring FKB build for ${FKB_INT}-bit integers")
   target_compile_definitions(${this} PRIVATE -DINT${FKB_INT})
else()
   message(STATUS "Configuring FKB build for 32-bit integers")
   target_compile_definitions(${this} PRIVATE -DINT32)
endif()

# handle real size
if(FKB_REAL)
   message(STATUS "Configuring FKB build for ${FKB_REAL}-bit reals")
   target_compile_definitions(${this} PRIVATE -DREAL${FKB_INT})
else()
   message(STATUS "Configuring FKB build for 32-bit reals")
   target_compile_definitions(${this} PRIVATE -DREAL32)
endif()

option(FKB_SERIAL "Build FKB for serial execution" ON)
if(FKB_SERIAL)
   message(STATUS "Configuring FKB build for serial execution")
else()
   message(STATUS "Configuring FKB build for parallel execution")
   list(APPEND SRCS src/lib/mod_parallel.F90)
   target_compile_definitions(${this} PRIVATE -DCAF)
endif()

set (TESTSRCS
   src/tests/test_bulk.F90
   src/tests/test_ensembles.F90
   src/tests/test_keras.F90
   src/tests/test_save_and_load.F90
   src/tests/test_training.F90
   )

foreach (SRC ${TESTSRCS})
   string(REPLACE ".F90" "" TESTEXEC_WITH_PATH ${SRC})
   get_filename_component(TESTEXEC ${TESTEXEC_WITH_PATH} NAME)
   ecbuild_add_executable(
      TARGET ${TESTEXEC}
      SOURCES ${SRC}
      LIBS ${this}
      NOINSTALL
      )
endforeach ()
