# CMakeLists.txt for STM32

# Minimum required version of CMake
cmake_minimum_required(VERSION 3.14.0)

# Cross compilers and tools
set(CMAKE_SYSTEM_NAME Generic)
set(CMAKE_SYSTEM_VERSION 1)
set(CMAKE_SYSTEM_PROCESSOR arm)

if (UNIX)
  message(STATUS "Unix system")
  set(ARM_TOOLCHAIN_BIN_DIR /opt/gcc-arm-11/bin)
  set(TOOLCHAIN_PREFIX arm-none-eabi-)
  set(EXT "")
else()
  message(STATUS "We suppose that is Windows system")
  set(ARM_TOOLCHAIN_BIN_DIR "c:/Program Files (x86)/GNU Tools Arm Embedded/9 2019-q4-major/bin")
  set(TOOLCHAIN_PREFIX arm-none-eabi-)
  set(EXT ".exe")
endif()

set(FLAGS
    "-fdata-sections -ffunction-sections -fstack-usage \
    --specs=nano.specs -Wl,--gc-sections")
set(CPP_FLAGS
    "-fdata-sections -ffunction-sections -fstack-usage \
    --specs=nano.specs -Wl,--gc-sections \
    -fno-rtti -fno-exceptions \
    -fno-threadsafe-statics")

set(CMAKE_C_COMPILER ${ARM_TOOLCHAIN_BIN_DIR}/${TOOLCHAIN_PREFIX}gcc${EXT} ${FLAGS})
set(CMAKE_CXX_COMPILER ${ARM_TOOLCHAIN_BIN_DIR}/${TOOLCHAIN_PREFIX}g++${EXT} ${CPP_FLAGS})
set(CMAKE_ASM_COMPILER ${ARM_TOOLCHAIN_BIN_DIR}/${TOOLCHAIN_PREFIX}gcc${EXT})
set(CMAKE_AR ${ARM_TOOLCHAIN_BIN_DIR}/${TOOLCHAIN_PREFIX}ar${EXT})
set(CMAKE_OBJCOPY ${ARM_TOOLCHAIN_BIN_DIR}/${TOOLCHAIN_PREFIX}objcopy${EXT} CACHE INTERNAL "objcopy tool")
set(CMAKE_OBJDUMP ${ARM_TOOLCHAIN_BIN_DIR}/${TOOLCHAIN_PREFIX}objdump${EXT} CACHE INTERNAL "objdump tool")
set(CMAKE_SIZE ${ARM_TOOLCHAIN_BIN_DIR}/${TOOLCHAIN_PREFIX}size${EXT} CACHE INTERNAL "size tool")

set(CMAKE_EXECUTABLE_SUFFIX_ASM ".elf")
set(CMAKE_EXECUTABLE_SUFFIX_C ".elf")
set(CMAKE_EXECUTABLE_SUFFIX_CXX ".elf")

set(CMAKE_TRY_COMPILE_TARGET_TYPE STATIC_LIBRARY)
set(CMAKE_FIND_ROOT_PATH_MODE_PROGRAM NEVER)
set(CMAKE_FIND_ROOT_PATH_MODE_LIBRARY ONLY)
set(CMAKE_FIND_ROOT_PATH_MODE_INCLUDE ONLY)
set(CMAKE_FIND_ROOT_PATH_MODE_PACKAGE ONLY)

# Project settings
# Project name will also be set to CMake environment variable PROJECT_NAME
project(STM32_Template_Demo VERSION 1.0.0 DESCRIPTION "Template file" LANGUAGES C CXX ASM)
#set(PROJECT_DIR ${CMAKE_CURRENT_SOURCE_DIR}/Src)

set(MCU_FAMILY STM32F4xx)
set(MCU_MODEL STM32F446xx)
set(CPU_PARAMETERS
    -mcpu=cortex-m4
    -mthumb
    -mfpu=fpv4-sp-d16
    -mfloat-abi=hard)

set(STARTUP_SCRIPT ${CMAKE_CURRENT_SOURCE_DIR}/startup_stm32f446xx.s)
set(MCU_LINKER_SCRIPT ${CMAKE_CURRENT_SOURCE_DIR}/STM32F446RETx_FLASH.ld)

# Common compile settings
# Language standard
set(CMAKE_C_STANDARD 11)
set(CMAKE_C_STANDARD_REQUIRED ON)
set(CMAKE_C_EXTENSIONS ON)
set(CMAKE_CXX_STANDARD 20)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS ON)

set(EXECUTABLE ${CMAKE_PROJECT_NAME})

# Headers
set(CUBEMX_INCLUDE_DIRECTORIES
	${CMAKE_CURRENT_SOURCE_DIR}/Core/Inc
    ${CMAKE_CURRENT_SOURCE_DIR}/Drivers/${MCU_FAMILY}_HAL_Driver/Inc
    ${CMAKE_CURRENT_SOURCE_DIR}/Drivers/${MCU_FAMILY}_HAL_Driver/Inc/Legacy
    ${CMAKE_CURRENT_SOURCE_DIR}/Drivers/CMSIS/Device/ST/${MCU_FAMILY}/Include
    ${CMAKE_CURRENT_SOURCE_DIR}/Drivers/CMSIS/Include)
set(PROJECT_INCLUDE_DIRECTORIES
    ${CMAKE_CURRENT_SOURCE_DIR}
    #${CMAKE_CURRENT_SOURCE_DIR}/${PROJECT_DIR}
)

# Sources
file(GLOB_RECURSE STM32CUBEMX_SOURCES
    ${CMAKE_CURRENT_SOURCE_DIR}/Core/*.c
    ${CMAKE_CURRENT_SOURCE_DIR}/Drivers/*.c)
#file(GLOB_RECURSE PROJECT_SOURCES FOLLOW_SYMLINKS
#    ${PROJECT_DIR}/*.cpp
#    ${PROJECT_DIR}/*.c)

# Executable files
add_executable(${EXECUTABLE}
    ${STM32CUBEMX_SOURCES}
    ${PROJECT_SOURCES}
    ${STARTUP_SCRIPT})

# Embedded macros(defines)
target_compile_definitions(${EXECUTABLE} PRIVATE
    ${MCU_MODEL}
    USE_HAL_DRIVER)

# Add header directories (AFTER add_executable !!)
target_include_directories(${EXECUTABLE} PRIVATE
    ${CUBEMX_INCLUDE_DIRECTORIES}
    ${PROJECT_INCLUDE_DIRECTORIES})

# Compiler and linker options
target_compile_options(${EXECUTABLE} PRIVATE
    ${CPU_PARAMETERS}
    -Wall
    -Wextra
    -Wpedantic
    -Wno-unused-parameter
    $<$<COMPILE_LANGUAGE:CXX>:
        -Wno-volatile
        -Wold-style-cast
        -Wuseless-cast
        -Wsuggest-override>
    $<$<CONFIG:Debug>:-Og -g3 -ggdb>
    $<$<CONFIG:Release>:-Og -g0>)

target_link_options(${EXECUTABLE} PRIVATE
    -T${MCU_LINKER_SCRIPT}
    ${CPU_PARAMETERS}
    -Wl,-Map=${CMAKE_PROJECT_NAME}.map
    --specs=nosys.specs
    -Wl,--start-group
    -lc
    -lm
    -lstdc++
    -Wl,--end-group
    -Wl,--print-memory-usage)

add_custom_command(TARGET ${EXECUTABLE} POST_BUILD
    COMMAND ${CMAKE_SIZE} $<TARGET_FILE:${EXECUTABLE}>)

add_custom_command(TARGET ${EXECUTABLE} POST_BUILD
    COMMAND ${CMAKE_OBJCOPY} -O ihex $<TARGET_FILE:${EXECUTABLE}>
    ${EXECUTABLE}.hex
    COMMAND ${CMAKE_OBJCOPY} -O binary $<TARGET_FILE:${EXECUTABLE}>
    ${EXECUTABLE}.bin)
