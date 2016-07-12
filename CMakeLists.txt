cmake_minimum_required(VERSION 3.5)

include (CMakeForceCompiler)
set(CMAKE_SYSTEM_NAME Generic)
set(CMAKE_SYSTEM_PROCESSOR cortex-m0plus)
SET(CMAKE_CROSSCOMPILING 1)
set(CMAKE_INCLUDE_PATH "")

set(tools               /usr/local/gcc-arm-none-eabi-5_2-2015q4)
set(CMAKE_CXX_COMPILER  ${tools}/bin/arm-none-eabi-g++)
set(CMAKE_C_COMPILER    ${tools}/bin/arm-none-eabi-gcc)
set(CMAKE_ASM_COMPILER  ${tools}/bin/arm-none-eabi-gcc)
set(SIZE_COMMAND        ${tools}/bin/arm-none-eabi-size)
set(OBJCOPY_COMMAND     ${tools}/bin/arm-none-eabi-objcopy)
set(CMAKE_C_COMPILER_FORCED 1)
set(CMAKE_CXX_COMPILER_FORCED 1)

project(nrf51822_xxac)
enable_language(ASM)

set(nrftools            /Users/jay/nRF5x-Command-Line-Tools_8_4_0_OSX)
set(MERGEHEX_COMMAND    ${nrftools}/mergehex/mergehex)

set(CHIP NRF51)
set(SOFTDEVICE S130)
set(BOARD PCA10031)

set(BUILD_SHARED_LIBS OFF)

# NRF related
set(NRF_SDK_ROOT            $ENV{HOME}/bluepark/external/nRF5_SDK_11.0.0_89a8197)
set(SDK_COMPONENTS          ${NRF_SDK_ROOT}/components)
set(NRF_TOOLCHAIN           ${SDK_COMPONENTS}/toolchain)
set(DEVICE_INCLUDES         ${SDK_COMPONENTS}/device)
set(NRF_GCC                 ${NRF_TOOLCHAIN}/gcc)

set(MAIN_TARGET ${PROJECT_NAME}.out)

set(LINKER_SCRIPT ${PROJECT_SOURCE_DIR}/blinky_gcc_nrf51.ld)

set(CMAKE_C_FLAGS "-T${LINKER_SCRIPT} -D${CHIP} -DBOARD_${BOARD} -DBSP_DEFINES_ONLY -mcpu=cortex-m0 -mthumb -mabi=aapcs --std=gnu99 -Wall -Werror -O3 -g3 -mfloat-abi=soft -ffunction-sections -fdata-sections -fno-strict-aliasing -fno-builtin --short-enums -L ${SDK_COMPONENTS}/toolchain/gcc -lc -lnosys")
set(CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} -Xlinker -Map=${PROJECT_NAME}.map -mthumb -mabi=aapcs -mcpu=cortex-m0")
set(CMAKE_ASM_FLAGS "${CMAKE_ASM_FLAGS} -x assembler-with-cpp -D${CHIP} -DBOARD_${BOARD} -DBSP_DEFINES_ONLY")

set(CMAKE_CXX_FLAGS "-std=c++11")
set(CXX_DEFAULT_FLAGS "-Wall")
set(CXX_SPECIFIC_FLAGS "-std=c++11")

include_directories(
    ${NRF_SDK_ROOT}/components/toolchain
    ${NRF_SDK_ROOT}/components/toolchain/gcc
    ${NRF_SDK_ROOT}/components/toolchain/CMSIS/include
    ${NRF_SDK_ROOT}/components/softdevice/${SOFTDEVICE}/headers
    ${NRF_SDK_ROOT}/examples/bsp
    ${NRF_SDK_ROOT}/components/device
    ${NRF_SDK_ROOT}/components/drivers_nrf/delay
    ${NRF_SDK_ROOT}/components/drivers_nrf/hal
)

set(SOURCE_FILES
    ${NRF_GCC}/gcc_startup_nrf51.s
    ${NRF_SDK_ROOT}/components/toolchain/system_nrf51.c
    ${SDK_COMPONENTS}/drivers_nrf/delay/nrf_delay.c
    main.c)

add_executable(${MAIN_TARGET} ${SOURCE_FILES})

add_custom_command(
        TARGET ${MAIN_TARGET}
        COMMAND ${SIZE_COMMAND} ${MAIN_TARGET}
        COMMAND ${OBJCOPY_COMMAND} -O ihex ${MAIN_TARGET} ${PROJECT_NAME}.hex
        COMMAND ${OBJCOPY_COMMAND} -O binary ${MAIN_TARGET} ${PROJECT_NAME}.bin
        #COMMAND ${MERGEHEX_COMMAND} --merge ${NRF_SDK_ROOT}/components/softdevice/s130/hex/s130_nrf51_2.0.0_softdevice.hex ${PROJECT_NAME}.hex --output combined.hex
)
