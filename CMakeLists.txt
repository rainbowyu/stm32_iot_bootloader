SET(CMAKE_SYSTEM_NAME Generic)
SET(CMAKE_SYSTEM_VERSION 1)
cmake_minimum_required(VERSION 3.7)
project(stm32_iot_bootloader C ASM)

#set(CMAKE_C_STANDARD 99)

# specify cross compilers and tools
SET(CMAKE_C_COMPILER_WORKS 1)
SET(CMAKE_C_COMPILER arm-none-eabi-gcc)
SET(CMAKE_CXX_COMPILER_WORKS 1)
SET(CMAKE_CXX_COMPILER arm-none-eabi-g++)
set(CMAKE_ASM_COMPILER  arm-none-eabi-gcc)
set(CMAKE_AR arm-none-eabi-ar)
set(CMAKE_OBJCOPY arm-none-eabi-objcopy)
set(CMAKE_OBJDUMP arm-none-eabi-objdump)
set(SIZE arm-none-eabi-size)

#Uncomment for hardware floating point
SET(FPU_FLAGS "-mfloat-abi=hard -mfpu=fpv4-sp-d16")
add_definitions(-DARM_MATH_CM4 -DARM_MATH_MATRIX_CHECK -DARM_MATH_ROUNDING)

#-D__FPU_PRESENT=1
add_definitions(-DRT_USING_NEWLIB -DHAVE_CCONFIG_H)
add_definitions(-D__weak=__attribute__\(\(weak\)\) -D__packed=__attribute__\(\(__packed__\)\) -DUSE_HAL_DRIVER -DSTM32F407xx)

#Uncomment for software floating point
#SET(FPU_FLAGS "-mfloat-abi=soft")

# set link script
SET(LINKER_SCRIPT ${CMAKE_SOURCE_DIR}/STM32F407ZETx_FLASH.ld)
SET(COMMON_FLAGS
        "-mcpu=cortex-m4 ${FPU_FLAGS} -mthumb -mthumb-interwork -ffunction-sections -fdata-sections \
        -g -fno-common -fmessage-length=0 -specs=nosys.specs -specs=nano.specs")
SET(CMAKE_CXX_FLAGS_INIT "${COMMON_FLAGS} -std=c++11")
SET(CMAKE_C_FLAGS_INIT "${COMMON_FLAGS} -std=gnu99")
SET(CMAKE_EXE_LINKER_FLAGS_INIT "-Wl,-gc-sections,--print-memory-usage  -T ${LINKER_SCRIPT} ")

# set include dir
# hal lib
SET(STM32_HAL_INCLUDE
        Libraries
        Libraries/STM32F4xx_HAL_Driver/Inc
        Libraries/STM32F4xx_HAL_Driver/Inc/Legacy)

# cmsis lib
SET(CMSIS_INCLUDE
        Libraries/CMSIS/Include
        Libraries/CMSIS/Device/ST/STM32F4xx/Include)

set(STM32_COMMON
        Stm32Common/inc)

SET(APP_INCLUDE

)

# all include
SET(ALL_INCLUDE
        ${STM32_HAL_INCLUDE}
        ${CMSIS_INCLUDE}
        ${STM32_COMMON}
        ${APP_INCLUDE}
        )
include_directories( ${ALL_INCLUDE} )

# set source code
file(GLOB_RECURSE STM32_HAL_SOURCES "Libraries/STM32F4xx_HAL_Driver/Src/*.c")
file(GLOB_RECURSE STM32_CMSIS_SOURCES "Libraries/CMSIS/Device/ST/STM32F4xx/Source/Templates/*.c")
file(GLOB_RECURSE STM32_COMMON "Stm32Common/src/*.c")
set(STM32_STARTUP "Libraries/CMSIS/Device/ST/STM32F4xx/Source/Templates/gcc")

set(ALL_SOURCE
        ${STM32_HAL_SOURCES}
        ${STM32_CMSIS_SOURCES}
        ${STM32_STARTUP}
        ${STM32_COMMON}
        ${LINKER_SCRIPT}
        main.c
)

add_executable(${PROJECT_NAME}.elf ${ALL_SOURCE})

set(CMAKE_EXE_LINKER_FLAGS
        "${CMAKE_EXE_LINKER_FLAGS} -Wl,-Map=${PROJECT_BINARY_DIR}/${PROJECT_NAME}.map")

set(HEX_FILE ${PROJECT_BINARY_DIR}/${PROJECT_NAME}.hex)
set(BIN_FILE ${PROJECT_BINARY_DIR}/${PROJECT_NAME}.bin)
add_custom_command(TARGET ${PROJECT_NAME}.elf POST_BUILD
        COMMAND ${CMAKE_OBJCOPY} -Oihex $<TARGET_FILE:${PROJECT_NAME}.elf> ${HEX_FILE}
        COMMAND ${CMAKE_OBJCOPY} -Obinary $<TARGET_FILE:${PROJECT_NAME}.elf> ${BIN_FILE}
        COMMENT "Building ${HEX_FILE}
Building ${BIN_FILE}")