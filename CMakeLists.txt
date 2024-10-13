cmake_minimum_required(VERSION 3.20)
set(PROJECT_NAME m328p_freertos)
set(CMAKE_TOOLCHAIN_FILE avr-gcc.toolchain.cmake)

project(${PROJECT_NAME} C CXX ASM)

## AVR Chip Configuration
# 8Mhz, this should match the crystal on your board,
# I use 8Mhz and 3.3V for the lowest power consumption
set(F_CPU 8000000UL)

# CPU, you can find the list here:
# https://gcc.gnu.org/onlinedocs/gcc/AVR-Options.html
set(MCU atmega328p)

# Default Baudrate for UART, read avr include/util/setbaud.h for usage
set(BAUD 9600)


include(FetchContent)
FetchContent_Declare(
    freertos_kernel
    GIT_REPOSITORY https://github.com/FreeRTOS/FreeRTOS-Kernel.git
    GIT_TAG        V11.1.0
    EXCLUDE_FROM_ALL
)

add_library(freertos_config INTERFACE)
target_include_directories(freertos_config SYSTEM INTERFACE include)
target_compile_definitions(freertos_config INTERFACE projCOVERAGE_TEST=0)

set(FREERTOS_HEAP "4" CACHE STRING "" FORCE)
# Select the native compile PORT
set(FREERTOS_PORT "GCC_POSIX" CACHE STRING "" FORCE)
# Select the cross-compile PORT
if (CMAKE_CROSSCOMPILING)
  set(FREERTOS_PORT "GCC_ATMEGA" CACHE STRING "" FORCE)
endif()

FetchContent_MakeAvailable(freertos_kernel)

target_compile_options(freertos_config INTERFACE
    -mmcu=${MCU}
    -g
    -Os
    -w
    -std=gnu++11
    -fno-exceptions
    -ffunction-sections
    -fdata-sections
    -fno-threadsafe-statics
    -gdwarf-2
    -funsigned-char
    -funsigned-bitfields
    -fpack-struct
    -fshort-enums
    -ffunction-sections
    -fdata-sections
    -fno-split-wide-types
    -fno-tree-scev-cprop
    -Wall
    -Wno-main
    -Wundef
    -pedantic
    -Wstrict-prototypes
    -Werror
    -Wfatal-errors
    -Wl,--relax,--gc-sections
)

target_compile_definitions(freertos_config INTERFACE
    -DF_CPU=${F_CPU}
    -DBAUD=${BAUD}
)

# # Load all files in src folder
# file(GLOB SOURCES
#     src/main.c
#     src/ParTest/ParTest.c
#     src/serial/serial.c
#     src/regtest.c
#     ${freertos_kernel_SOURCE_DIR}/tasks.c
#     ${freertos_kernel_SOURCE_DIR}/queue.c
#     ${freertos_kernel_SOURCE_DIR}/list.c
#     ${freertos_kernel_SOURCE_DIR}/croutine.c
#     ${freertos_kernel_SOURCE_DIR}/portable/MemMang/heap_1.c
#     ${freertos_kernel_SOURCE_DIR}/portable/GCC/ATMega323/port.c
#     src/Minimal/crflash.c
#     src/Minimal/integer.c
#     src/Minimal/PollQ.c
#     src/Minimal/comtest.c
# )

# add_avr_executable(${PROJECT_NAME} ${SOURCES})

# target_include_directories(${PROJECT_NAME} PRIVATE
#     include
#     include/Minimal
#     ${freertos_kernel_SOURCE_DIR}/include
#     ${freertos_kernel_SOURCE_DIR}/portable/GCC/ATMega323
# )

# # Compilation flags
# target_compile_options(${PROJECT_NAME} PRIVATE
#     -mmcu=${MCU}
#     -g
#     -Os
#     -w
#     -std=gnu++11
#     -fno-exceptions
#     -ffunction-sections
#     -fdata-sections
#     -fno-threadsafe-statics
#     -gdwarf-2
#     -funsigned-char
#     -funsigned-bitfields
#     -fpack-struct
#     -fshort-enums
#     -ffunction-sections
#     -fdata-sections
#     -fno-split-wide-types
#     -fno-tree-scev-cprop
#     -Wall
#     -Wno-main
#     -Wundef
#     -pedantic
#     -Wstrict-prototypes
#     -Werror
#     -Wfatal-errors
#     -Wl,--relax,--gc-sections
# )

# # Pass defines to compiler
# target_compile_definitions(${PROJECT_NAME} PRIVATE
#     -DF_CPU=${F_CPU}
#     -DBAUD=${BAUD}
# )

# # Linker flags
# target_link_options(${PROJECT_NAME} PRIVATE
#     -mmcu=${MCU}
#     -lc
#     -lm
#     -lgcc
#     -Wl,-lprintf_flt
#     -Wl,-u,vfprintf
# )


# The programmer to use, read avrdude manual for list
set(PROG_TYPE usbasp)
set(MCU_PART_NO m328p)

# AVR Fuses, must be in concordance with your hardware and F_CPU
# http://eleccelerator.com/fusecalc/fusecalc.php?chip=atmega328p
set(E_FUSE 0xfd)
set(H_FUSE 0xda)
set(L_FUSE 0xfd)
set(LOCK_BIT 0xff)

# Create one target
# add_executable(${PROJECT_NAME} ${SOURCES})

# Rename the output to .elf as we will create multiple files
# set_target_properties(${PROJECT_NAME} PROPERTIES OUTPUT_NAME ${PROJECT_NAME}.elf)

# Strip binary for upload
# add_custom_target(strip ALL avr-strip ${PROJECT_NAME}.elf DEPENDS ${PROJECT_NAME})

# Transform binary into hex file, we ignore the eeprom segments in the step
# add_custom_target(hex ALL avr-objcopy -R .eeprom -O ihex ${PROJECT_NAME}.elf ${PROJECT_NAME}.hex DEPENDS strip)
# Transform binary into hex file, this is the eeprom part (empty if you don't
# use eeprom static variables)
# add_custom_target(eeprom avr-objcopy -j .eeprom  --set-section-flags=.eeprom="alloc,load"  --change-section-lma .eeprom=0 -O ihex ${PROJECT_NAME}.elf ${PROJECT_NAME}.eep DEPENDS strip)

# Upload the firmware with avrdude
add_custom_target(upload avrdude -c ${PROG_TYPE} -p ${MCU_PART_NO} -U flash:w:${PROJECT_NAME}.hex DEPENDS ${PROJECT_NAME}_flash_files)

# Upload the eeprom with avrdude
# add_custom_target(upload_eeprom avrdude -c ${PROG_TYPE} -p ${MCU_PART_NO} -U eeprom:w:${PROJECT_NAME}.eep DEPENDS eeprom)

# Burn fuses
# add_custom_target(fuses avrdude -c ${PROG_TYPE} -p ${MCU_PART_NO}  -U lfuse:w:${L_FUSE}:m -U hfuse:w:${H_FUSE}:m -U efuse:w:${E_FUSE}:m -U lock:w:${LOCK_BIT}:m )

# Clean extra files
# set_directory_properties(PROPERTIES ADDITIONAL_MAKE_CLEAN_FILES "${PROJECT_NAME}.hex;${PROJECT_NAME}.eeprom;${PROJECT_NAME}.lst")
