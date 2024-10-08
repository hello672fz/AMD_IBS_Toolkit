# Build Kernel modules
set(MODULE_NAME ibs-driver)

## First, find the kernel build directory
execute_process(
        COMMAND uname -r
        OUTPUT_VARIABLE KERNEL_RELEASE
        OUTPUT_STRIP_TRAILING_WHITESPACE)
set(KBUILD_DIR /lib/modules/${KERNEL_RELEASE}/build/)
find_file(KERNEL_MAKEFILE NAMES Makefile PATHS ${KBUILD_DIR} NO_DEFAULT_PATH)
if (NOT KERNEL_MAKEFILE)
    message(FATAL_ERROR "There is no Makefile in ${KBUILD_DIR}!")
endif ()

## Second, gather the source files
set(IBS_DRIVER_SRC_FILES
        ${BASE_DIR}/driver/ibs-core.c
        ${BASE_DIR}/driver/ibs-fops.c
        ${BASE_DIR}/driver/ibs-interrupt.c
        ${BASE_DIR}/driver/ibs-utils.c
        ${BASE_DIR}/driver/ibs-workarounds.c)

## Third, make the src files accessible to the kernel Makefile
set(MODULE_BUILD_DIR ${CMAKE_CURRENT_BINARY_DIR}/)
set(MODULE_KBUILD_FILE "obj-m := ${MODULE_NAME}.o")
foreach (file ${IBS_DRIVER_SRC_FILES})
    file(RELATIVE_PATH file "${CMAKE_CURRENT_SOURCE_DIR}" "${file}")
    configure_file(${file} ${file} COPYONLY)
    string(REPLACE ".c" ".o" file_obj "${file}")
    set(MODULE_KBUILD_FILE "${MODULE_KBUILD_FILE}\n${MODULE_NAME}-y += ${file_obj}")
endforeach ()

set(MODULE_KBUILD_FILE "${MODULE_KBUILD_FILE}\nccflags-y += -I${BASE_DIR}/include")
set(MODULE_KBUILD_FILE "${MODULE_KBUILD_FILE}\nccflags-y += -I${BASE_DIR}/driver")

## Forth, generate a Kbuild file
file(WRITE ${CMAKE_CURRENT_BINARY_DIR}/Kbuild ${MODULE_KBUILD_FILE})

## Fifth, add a custom target to build the module
set(MODULE_CMD ${CMAKE_MAKE_PROGRAM} -C ${KBUILD_DIR} M=${CMAKE_CURRENT_BINARY_DIR})
add_custom_command(OUTPUT ${MODULE_NAME}.ko
        COMMAND ${MODULE_CMD} modules
        WORKING_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}
        DEPENDS ${IBS_DRIVER_SRC_FILES} ${CMAKE_CURRENT_BINARY_DIR}/Kbuild
        VERBATIM)
add_custom_target(${MODULE_NAME} DEPENDS ${MODULE_NAME}.ko)
add_custom_target(${MODULE_NAME}-clean COMMAND ${MODULE_CMD} clean)

# This target helps parsing C files for IDEs like CLion
add_library(dummy-${MODULE_NAME} EXCLUDE_FROM_ALL ${IBS_DRIVER_SRC_FILES})
target_include_directories(dummy-${MODULE_NAME} PRIVATE
        ${KBUILD_DIR}/include
        ${BASE_DIR}/driver)
set_target_properties(dummy-${MODULE_NAME} PROPERTIES DEPRECATION "DO NOT BUILD THIS TARGET")
