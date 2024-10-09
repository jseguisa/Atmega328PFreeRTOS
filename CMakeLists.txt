cmake_minimum_required(VERSION 3.20)
set(PROJECT_NAME free-rtos-demo)
set(CMAKE_TOOLCHAIN_FILE avr-gcc.toolchain.cmake)

project(${PROJECT_NAME} C CXX)

## AVR Chip Configuration
# 8Mhz, this should match the crystal on your board,
# I use 8Mhz and 3.3V for the lowest power consumption
set(F_CPU 16000000UL)

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
)

add_library(freertos_config INTERFACE)
target_include_directories(freertos_config SYSTEM INTERFACE include)
target_compile_definitions(freertos_config INTERFACE projCOVERAGE_TEST=0)

set(FREERTOS_HEAP "4" CACHE STRING "" FORCE)
# Select the native compile PORT
set(FREERTOS_PORT "GCC_POSIX" CACHE STRING "" FORCE)
# Select the cross-compile PORT
if (CMAKE_CROSSCOMPILING)
  set(FREERTOS_PORT "GCC_ATMEGA323" CACHE STRING "" FORCE)
endif()

FetchContent_MakeAvailable(freertos_kernel)

target_compile_definitions(freertos_config INTERFACE ${definitions} -DF_CPU=${F_CPU})
target_compile_options(freertos_config INTERFACE ${options})

# Pass defines to compiler
add_definitions(
    -DF_CPU=${F_CPU}
    -DBAUD=${BAUD}
)

include_directories(include)

file(GLOB_RECURSE SOURCES src/*.c src/*.cpp) # Load all files in src folder
add_avr_executable(${PROJECT_NAME} ${MCU} ${SOURCES})


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
# add_custom_target(upload avrdude -c ${PROG_TYPE} -p ${MCU_PART_NO} -U flash:w:${PROJECT_NAME}.hex DEPENDS hex)

# Upload the eeprom with avrdude
# add_custom_target(upload_eeprom avrdude -c ${PROG_TYPE} -p ${MCU_PART_NO} -U eeprom:w:${PROJECT_NAME}.eep DEPENDS eeprom)

# Burn fuses
# add_custom_target(fuses avrdude -c ${PROG_TYPE} -p ${MCU_PART_NO}  -U lfuse:w:${L_FUSE}:m -U hfuse:w:${H_FUSE}:m -U efuse:w:${E_FUSE}:m -U lock:w:${LOCK_BIT}:m )

# Clean extra files
# set_directory_properties(PROPERTIES ADDITIONAL_MAKE_CLEAN_FILES "${PROJECT_NAME}.hex;${PROJECT_NAME}.eeprom;${PROJECT_NAME}.lst")
