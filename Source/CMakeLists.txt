cmake_minimum_required(VERSION 3.13)

set(PICO_SDK_FETCH_FROM_GIT on)
set(PICO_SDK_FETCH_FROM_GIT_TAG "2.2.0")
set(PICO_EXTRAS_FETCH_FROM_GIT on)
set(PICO_EXTRAS_FETCH_FROM_GIT_TAG "sdk-2.2.0")

include(pico_sdk_import.cmake)
include(pico_extras_import.cmake)

project(SKpico C CXX ASM)
set(CMAKE_C_STANDARD 11)
set(CMAKE_CXX_STANDARD 17)

option(USE_RGB_LED "Enable RGB LED support" off)
option(FLASH_LED "Enable flash LED support" off)
option(U64BOARD "Enable U64BOARD support" off)
option(USE_DAC "Enable DAC output" off)
option(OUTPUT_VIA_PWM "Enable PWM output" off)
option(SKPICO_2350CR "Enable SKPICO2350CR" off)
option(SKPICO_2040DAC "Enable SKPICO2040DAC" off)
set(PROGRAM_VERSION "0.1" CACHE STRING "SKpico program version")

project(SKpico)

set(PICO_COPY_TO_RAM 1)

pico_sdk_init()

add_compile_definitions(PICO_NO_FPGA_CHECK=1)
add_compile_definitions(PICO_BARE_METAL=1)
add_compile_definitions(PICO_CXX_ENABLE_EXCEPTIONS=0)
add_compile_definitions(PICO_STDIO_UART=0)

add_executable(SKpico
    SKpico.c
    exodecr.c
    fmopl.c
    prgslots.cc
    reSID16/envelope.cc
    reSID16/extfilt.cc
    reSID16/pot.cc
    reSID16/filter.cc
    reSID16/sid.cc
    reSID16/voice.cc
    reSID16/wave.cc
    reSIDWrapper.cc
)

pico_generate_pio_header(SKpico ${CMAKE_CURRENT_LIST_DIR}/ws2812.pio)

target_compile_definitions(SKpico PUBLIC  PICO PICO_STACK_SIZE=0x100)
target_compile_definitions(SKpico PRIVATE PICO_MALLOC_PANIC=0)
target_compile_definitions(SKpico PRIVATE PICO_USE_MALLOC_MUTEX=0)
target_compile_definitions(SKpico PRIVATE PICO_DEBUG_MALLOC=0)
if(USE_RGB_LED)
    target_compile_definitions(SKpico PRIVATE USE_RGB_LED=1)
endif()
if(U64BOARD)
    target_compile_definitions(SKpico PRIVATE U64BOARD=1)
endif()
if(USE_DAC)
    target_compile_definitions(SKpico PRIVATE USE_DAC=1)
endif()
if(FLASH_LED)
    target_compile_definitions(SKpico PRIVATE FLASH_LED=1)
endif()
if(SKPICO_2040DAC)
    target_compile_definitions(SKpico PRIVATE SKPICO_2040DAC=1)
endif()
if(OUTPUT_VIA_PWM)
    target_compile_definitions(SKpico PRIVATE OUTPUT_VIA_PWM=1)
endif()
target_compile_options(SKpico PRIVATE -save-temps -fverbose-asm)

if(SKPICO_2350CR)
    set(PICO_CUSTOM_LINKER_SCRIPT ${CMAKE_CURRENT_LIST_DIR}/memmap_copy_to_ram_2350.ld)
    target_compile_definitions(SKpico PRIVATE SKPICO_2350CR=1)
else()
    set(PICO_CUSTOM_LINKER_SCRIPT ${CMAKE_CURRENT_LIST_DIR}/memmap_copy_to_ram_skpico.ld)
endif()

set_target_properties(SKpico PROPERTIES PICO_TARGET_LINKER_SCRIPT ${PICO_CUSTOM_LINKER_SCRIPT})

target_link_libraries(SKpico pico_stdlib pico_multicore hardware_dma hardware_interp hardware_pwm pico_audio_i2s hardware_flash hardware_adc)

pico_set_program_name(SKpico "SKpico")
pico_set_program_version(SKpico ${PROGRAM_VERSION})

# create map/bin/hex/uf2 file in addition to ELF.
pico_add_extra_outputs(SKpico)

