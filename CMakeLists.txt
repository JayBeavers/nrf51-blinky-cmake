cmake_minimum_required(VERSION 3.5)

set(CMAKE_SYSTEM_NAME Generic)
set(CMAKE_SYSTEM_PROCESSOR cortex-m0plus)
set(CMAKE_INCLUDE_PATH "")

set(tools               /usr/local/gcc-arm-none-eabi-5_2-2015q4)
set(CMAKE_CXX_COMPILER  ${tools}/bin/arm-none-eabi-g++)
set(CMAKE_C_COMPILER    ${tools}/bin/arm-none-eabi-gcc)
set(CMAKE_ASM_COMPILER  ${tools}/bin/arm-none-eabi-gcc)
set(SIZE_COMMAND        ${tools}/bin/arm-none-eabi-size)
set(OBJCOPY_COMMAND     ${tools}/bin/arm-none-eabi-objcopy)
set(CMAKE_C_COMPILER_FORCED 1)
set(CMAKE_CXX_COMPILER_FORCED 1)

project(blinky)
enable_language(ASM)

set(nrftools            /Users/jay/nRF5x-Command-Line-Tools_8_4_0_OSX)
set(NRFJPROG_COMMAND    ${nrftools}/nrfjprog/nrfjprog)
set(MERGEHEX_COMMAND    ${nrftools}/mergehex/mergehex)

set(CHIP NRF51)
set(SOFTDEVICE S130)
set(BOARD PCA10031)

set(BUILD_SHARED_LIBS OFF)

set(NRF_SDK_ROOT            $ENV{HOME}/bluepark/external/nRF5_SDK_11.0.0_89a8197)

set(MAIN_TARGET ${PROJECT_NAME}.elf)

set(LINKER_SCRIPT ${PROJECT_SOURCE_DIR}/blinky_gcc_nrf51.ld)

set(CXX_DEFAULT_FLAGS "-Wall -Wfatal-errors -Werror")
set(CXX_SPECIFIC_FLAGS "-std=c++14 -ftemplate-backtrace-limit=0 -fvisibility-inlines-hidden -fno-rtti -fno-exceptions")
set(C_SPECIFIC_FLAGS "--std=gnu99 -fno-strict-aliasing -fno-builtin --short-enums -L ${NRF_SDK_ROOT}/components/toolchain/gcc -lc -lnosys -T${LINKER_SCRIPT}")
set(ARM_FLAGS "-mthumb -mcpu=cortex-m0 -ffunction-sections -fdata-sections")
set(CXX_ARC_FLAGS "${ARM_FLAGS} -mabi=aapcs -mfloat-abi=soft")
set(LINK_ARC_FLAGS "${ARM_FLAGS} --specs=nosys.specs --specs=nano.specs -static -lm -lstdc++ -lsupc++")
set(LINK_DEFAULT_FLAGS "-Xlinker -Map=${PROJECT_NAME}.map -Wl,--gc-sections")

add_definitions(-D_LITTLE_ENDIAN=1 -D${CHIP} -DBOARD_${BOARD} -DDEBUG -D__HEAP_SIZE=0 -DBSP_DEFINES_ONLY -DSOFTDEVICE_PRESENT -DBLE_STACK_SUPPORT_REQD -DS130)

# debugging
set(LINK_BUILD_TYPE_FLAGS "-g")
set(CXX_BUILD_TYPE_FLAGS "-O0 -ggdb")

set(CMAKE_ASM_FLAGS "-x assembler-with-cpp")
set(CMAKE_CXX_FLAGS "${CXX_DEFAULT_FLAGS} ${CXX_SPECIFIC_FLAGS} ${CXX_ARC_FLAGS} ${CXX_BUILD_TYPE_FLAGS} ${CXX_DISABLED_WARNINGS}")
set(CMAKE_C_FLAGS   "${CXX_DEFAULT_FLAGS} ${C_SPECIFIC_FLAGS} ${CXX_ARC_FLAGS} ${CXX_BUILD_TYPE_FLAGS} ${CXX_DISABLED_WARNINGS}")
set(CMAKE_EXE_LINKER_FLAGS "${LINK_DEFAULT_FLAGS} ${LINK_ARC_FLAGS} ${LINK_BUILD_TYPE_FLAGS} -L ${CMAKE_SOURCE_DIR}")

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
    ${NRF_SDK_ROOT}/components/toolchain/gcc/gcc_startup_nrf51.s
    ${NRF_SDK_ROOT}/components/toolchain/system_nrf51.c
    ${NRF_SDK_ROOT}/components/drivers_nrf/delay/nrf_delay.c
    main.c
)

add_executable(${PROJECT_NAME}.elf ${SOURCE_FILES})

add_custom_command(
        TARGET ${PROJECT_NAME}.elf
        COMMAND ${SIZE_COMMAND} ${MAIN_TARGET}
        COMMAND ${OBJCOPY_COMMAND} -O ihex ${MAIN_TARGET} ${PROJECT_NAME}.hex
        COMMAND ${OBJCOPY_COMMAND} -O binary ${MAIN_TARGET} ${PROJECT_NAME}.bin
        #COMMAND ${MERGEHEX_COMMAND} --merge ${NRF_SDK_ROOT}/components/softdevice/s130/hex/s130_nrf51_2.0.0_softdevice.hex ${PROJECT_NAME}.hex --output combined.hex
)

add_custom_target(${PROJECT_NAME}.flash
        COMMAND ${NRFJPROG_COMMAND} --sectorerase --program ${PROJECT_NAME}.hex --verify
        COMMAND ${NRFJPROG_COMMAND} --reset
        )

add_custom_target(${PROJECT_NAME}.flashall
        COMMAND ${NRFJPROG_COMMAND} --chiperase --program ${NRF_SDK_ROOT}/components/softdevice/s130/hex/s130_nrf51_2.0.0_softdevice.hex
        COMMAND ${NRFJPROG_COMMAND} --sectorerase --program ${PROJECT_NAME}.hex --verify
        COMMAND ${NRFJPROG_COMMAND} --reset
        )

add_dependencies(${PROJECT_NAME}.flash ${PROJECT_NAME}.elf)
add_dependencies(${PROJECT_NAME}.flashall ${PROJECT_NAME}.elf)
