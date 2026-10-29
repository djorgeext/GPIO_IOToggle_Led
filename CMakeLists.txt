# Set the minimum version of CMake that can be used
cmake_minimum_required(VERSION 3.20)

# Enable compile command to ease indexing with e.g. clangd
set(CMAKE_EXPORT_COMPILE_COMMANDS TRUE)
set(CMAKE_C_COMPILER_FORCED TRUE)
set(CMAKE_CXX_COMPILER_FORCED TRUE)

# Set target definition variables required by the toolchain file
include(${CMAKE_CURRENT_SOURCE_DIR}/cmake/target.cmake)

# Set the toolchain file for cross-compilation
include(${CMAKE_CURRENT_SOURCE_DIR}/cmake/gcc-arm-none-eabi.cmake)

# Project definition
project(GPIO_IOToggle)

# Enable CMake support for ASM and C/CPP languages
enable_language(C ASM)

set(PROJECT_NAME GPIO_IOToggle)

# Create an executable object type
add_executable(${PROJECT_NAME})

# Include additional CMake files for flags, components, and files
include(${CMAKE_CURRENT_SOURCE_DIR}/cmake/flags.cmake)
include(${CMAKE_CURRENT_SOURCE_DIR}/cmake/components.cmake)
include(${CMAKE_CURRENT_SOURCE_DIR}/cmake/files.cmake)

# WeAct MiniSTM32H7xx (STM32H750VBT6) build type: code runs straight from the
# 128KB internal flash at 0x08000000, no external-memory bootloader needed.
# The settings in cmake/flags.cmake only cover the STM32H750B-DK build types.
set(WEACT_LINKER_SCRIPT "${CMAKE_CURRENT_SOURCE_DIR}/STM32H750VBTX_FLASH.ld")

target_compile_options(${PROJECT_NAME} PRIVATE
  $<$<CONFIG:WeAct_InternalFlash_Debug>:
    $<$<COMPILE_LANGUAGE:C>:
      --specs=nano.specs
      -std=gnu11
      -g3
      -O0
      -Wall
      -fstack-usage
    >
    $<$<COMPILE_LANGUAGE:ASM>:
      --specs=nano.specs
      -g3
      -x
      assembler-with-cpp
    >
  >
)

target_compile_definitions(${PROJECT_NAME} PRIVATE
  $<$<CONFIG:WeAct_InternalFlash_Debug>:
    $<$<COMPILE_LANGUAGE:C>:
      DEBUG
      STM32H750xx
      USE_HAL_DRIVER
      USE_PWR_LDO_SUPPLY
      APPLICATION_ADDRESS=0x08000000U
    >
    $<$<COMPILE_LANGUAGE:ASM>:
      DEBUG
    >
  >
)

target_link_options(${PROJECT_NAME} PRIVATE
  $<$<CONFIG:WeAct_InternalFlash_Debug>:
    --specs=nano.specs
    -T${WEACT_LINKER_SCRIPT}
    --specs=nosys.specs
    -Wl,-Map=${CMAKE_CURRENT_BINARY_DIR}/GPIO_IOToggle.map
    -static
  >
)

if(CMAKE_BUILD_TYPE STREQUAL "WeAct_InternalFlash_Debug")
  set_target_properties(${PROJECT_NAME} PROPERTIES LINK_DEPENDS "${WEACT_LINKER_SCRIPT}")
endif()

# Add sources to executable/library
target_sources(${PROJECT_NAME} PRIVATE
  # User defined sources
)

# Add include directories
target_include_directories(${PROJECT_NAME} PRIVATE
  # User defined include directories
)

# Compiler options
target_compile_options(${PROJECT_NAME} PRIVATE
  # User defined compiler options
)

# Add additional defined symbols
target_compile_definitions(${PROJECT_NAME} PRIVATE
  # Add additional defined symbols here
)

# Linker options
target_link_options(${PROJECT_NAME} PRIVATE 
  # User defined linker options
)

# Add additional linker search directories
target_link_directories(${PROJECT_NAME} PRIVATE
  # Add additional linker search directories here
)

# Add additional linker libraries
target_link_libraries(${PROJECT_NAME} PRIVATE
  # Add additional linker libraries here
)
