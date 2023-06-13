# Set minimum CMake version required to build this project
cmake_minimum_required(VERSION 3.16)

set(TOOLCHAIN_PREFIX /Applications/ARM)

# Include the specific toolchain file which has the configurations for the ARM toolchain
include(toolchain-arm-none-eabi.cmake)

# Declare project name and the languages used in the project
project(picopad-template ASM C CXX)

# Set the path to the SDK. CMAKE_SOURCE_DIR refers to the directory where the top level CMakeLists.txt file is located.
set(SDK_PATH ${CMAKE_SOURCE_DIR}/picopad-sdk/sdk)

set(UF2 ${CMAKE_SOURCE_DIR}/tools/elf2uf2${TOOLS_SUFFIX} ${EXE_FILE_EXT})

set(CRC ${CMAKE_SOURCE_DIR}/tools/loader_crc${TOOLS_SUFFIX} ${EXE_FILE_EXT})

# Specify the linker script file
set(LDSCRIPT ${SDK_PATH}/memmap_app.ld)

# Include additional cmake file with flags configuration
include(picopad-flags.cmake)

# Add the current directory to the list of directories to look for include files
include_directories(.)

# Set the assembly source files
set(ASM_SOURCES
        picopad-sdk/sdk/!LOADER/loader_bin.S
        picopad-sdk/sdk/_lib/src/vga_blitkey.S
        picopad-sdk/sdk/_sdk/crt0.S
        picopad-sdk/sdk/_sdk/src/sdk_divider.S
        picopad-sdk/sdk/_sdk/src/sdk_double_asm.S
        picopad-sdk/sdk/_sdk/src/sdk_float_asm.S
        picopad-sdk/sdk/_sdk/src/sdk_memops_asm.S
        )

# Set the C source files, looking for .c files in various directories under the SDK path
file(GLOB_RECURSE C_SOURCES
        picopad-sdk/sdk/_sdk/src/*.c
        picopad-sdk/sdk/_lib/src/*.c
        picopad-sdk/sdk/_font/*.c
        picopad-sdk/sdk/_lib/bigint/bigint.c
        picopad-sdk/sdk/_lib/bigint/bernoulli.c
        picopad-sdk/sdk/_lib/real/real.c
        picopad-sdk/sdk/_display/st7789/st7789.c
        picopad-sdk/sdk/_devices/picopad/*.c
        )

# Set the C++ source files, looking for .cpp files in the 'src' directory
file(GLOB_RECURSE SOURCES "src/*.cpp" "multiset/*.o")

# Create an executable file from the source files
add_executable(${PROJECT_NAME}.elf ${ASM_SOURCES} ${C_SOURCES} ${SOURCES})

# Create a 'build' directory
file(MAKE_DIRECTORY ${CMAKE_BINARY_DIR}/../build)

# Generate a .bin file from the .elf file after the build
# Generate a .hex file from the .elf file after the build
add_custom_command(TARGET ${PROJECT_NAME}.elf
        POST_BUILD
        COMMENT "Generating bin file from .elf file"
        COMMAND ${CMAKE_OBJCOPY} -O binary $<TARGET_FILE:${PROJECT_NAME}.elf> ${PROJECT_NAME}.bin
        COMMENT "Generating hex file from .elf file"
        COMMAND ${CMAKE_OBJCOPY} -O ihex $<TARGET_FILE:${PROJECT_NAME}.elf> ${PROJECT_NAME}.hex
        )

add_custom_command(TARGET ${PROJECT_NAME}.elf POST_BUILD
        COMMENT "UF2 ${PROJECT_NAME}.uf2"
        COMMAND ${UF2} $<TARGET_FILE:${PROJECT_NAME}.elf> ${PROJECT_NAME}.uf2)

add_custom_command(TARGET ${PROJECT_NAME}.elf POST_BUILD
        COMMENT "CRC ${PROJECT_NAME}.bin ${PROJECT_NAME}.uf2"
        COMMAND ${CRC} ${PROJECT_NAME}.bin ${PROJECT_NAME}.uf2)

# Copy the .elf, .bin, and .hex files to the 'build' directory after the build
add_custom_command(TARGET ${PROJECT_NAME}.elf
        POST_BUILD
        COMMENT "Copying elf file to build directory"
        COMMAND ${CMAKE_COMMAND} -E copy ${PROJECT_NAME}.elf ${CMAKE_BINARY_DIR}/../build/
        COMMENT "Copying bin file to build directory"
        COMMAND ${CMAKE_COMMAND} -E copy ${PROJECT_NAME}.bin ${CMAKE_BINARY_DIR}/../build/
        COMMENT "Copying hex file to build directory"
        COMMAND ${CMAKE_COMMAND} -E copy ${PROJECT_NAME}.hex ${CMAKE_BINARY_DIR}/../build/
        COMMENT "Copying uf2 file to build directory"
        COMMAND ${CMAKE_COMMAND} -E copy ${PROJECT_NAME}.uf2 ${CMAKE_BINARY_DIR}/../build/
        )

# Print the size of the binary file after the build is complete
add_custom_command(TARGET ${PROJECT_NAME}.elf POST_BUILD
        COMMAND ${CMAKE_SIZE} $<TARGET_FILE:${PROJECT_NAME}.elf>)
