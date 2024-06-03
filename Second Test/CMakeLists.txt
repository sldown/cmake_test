cmake_minimum_required(VERSION 3.9)
# 目标平台系统名称，裸机嵌入式一般就写 Generic
# 目标平台处理器架构，如 arm mips riscv x86
set(CMAKE_SYSTEM_NAME Generic)
# 目标平台的体系结构
set(CMAKE_SYSTEM_PROCESSOR arm)

# set(CMAKE_C_COMPILE_WORKS TRUE)
SET(CMAKE_CXX_COMPILE_WORKS TRUE)
SET(CMAKE_C_COMPILE_WORKS TRUE)

set(CMAKE_C_COMPILER arm-none-eabi-gcc)
set(CMAKE_CXX_COMPILER arm-none-eabi-g++)
set(CMAKE_OBJCOPY  arm-none-eabi-objcopy)
set(CMAKE_OBJDUMP  arm-none-eabi-objdump)
set(CMAKE_ASM_COMPILER  arm-none-eabi-as)
set(CMAKE_AR   arm-none-eabi-ar)

set(CMAKE_TRY_COMPILE_TARGET_TYPE STATIC_LIBRARY)

project(myproj)

enable_language(ASM C)
set(CMAKE_C_STANDARD 11)
set(CMAKE_C_STANDARD_REQUIRED OFF)
set(CMAKE_C_EXTENSIONS OFF)

set(ELF_TARGET ${CMAKE_PROJECT_NAME}.elf)

# 预定义参数
add_definitions(
    -D GD32F450 
)

# CPU 架构
set(CPU_FLAGS
    -mcpu=cortex-m4
)

# 链接脚本
set(LINK_SCRIPT
    ${CMAKE_CURRENT_SOURCE_DIR}/gd32f4xx_flash.ld
)

# 启动文件
# set(STARTUP_SCRIPT
#     # ${CMAKE_CURRENT_SOURCE_DIR}/GD32/GD32F4xx_Firmware_Library/CMSIS/GD/D32F4xx/Source/ARM/startup_gd32f450_470.s
#     ${CMAKE_CURRENT_SOURCE_DIR}/startup_gd32f450_470.S
# )                           
add_subdirectory(GD32/GD32F4xx_Firmware_Library/CMSIS/GD/GD32F4xx/Source/ARM)  


# CMSIS 内核文件
set(CMSIS_CORE
    ${CMAKE_CURRENT_SOURCE_DIR}/GD32/GD32F4xx_Firmware_Library/CMSIS/core_cm4.h
    # ${CMAKE_CURRENT_SOURCE_DIR}/GD32/GD32F4xx_Firmware_Library/CMSIS/GD/GD32F4xx/Source/ARM/startup_gd32f450_470.s
    ${CMAKE_CURRENT_SOURCE_DIR}/GD32/GD32F4xx_Firmware_Library/CMSIS/GD/GD32F4xx/Source/system_gd32f4xx.c
)

# 标准库外设源文件
file(GLOB STDPERIPH_DRIVER
    ${CMAKE_CURRENT_SOURCE_DIR}/GD32/GD32F4xx_Firmware_Library/GD32F4xx_standard_peripheral/Source/*.c
)

# 用户程序的源文件
file(GLOB_RECURSE USER_SRC
    ${CMAKE_CURRENT_SOURCE_DIR}/GD32/Project/*.c
)


# 目标程序
add_executable(${ELF_TARGET}
    ${CMSIS_CORE}
    # ${STARTUP_SCRIPT}
    ${STDPERIPH_DRIVER}
    ${USER_SRC}
)
# target_link_libraries(${ELF_TARGET} PRIVATE gd32f4xx_startup)
target_link_libraries(${ELF_TARGET} PRIVATE arm_cortex_m4_startup)

# 编译选项
target_compile_definitions(${ELF_TARGET} PRIVATE
    ${CMAKE_CURRENT_SOURCE_DIR}/GD32/Project/gd32f4xx_it.h
)


# # 链接选项
# target_link_options(${ELF_TARGET} PRIVATE
#                             ${LINK_SCRIPT}
# )

# 头文件路径
target_include_directories(${ELF_TARGET} PRIVATE
    ${CMAKE_CURRENT_SOURCE_DIR}/GD32/GD32F4xx_Firmware_Library/CMSIS/
    ${CMAKE_CURRENT_SOURCE_DIR}/GD32/GD32F4xx_Firmware_Library/CMSIS/GD/GD32F4xx/Include/
    ${CMAKE_CURRENT_SOURCE_DIR}/GD32/GD32F4xx_Firmware_Library/GD32F4xx_standard_peripheral/Include/
    ${CMAKE_CURRENT_SOURCE_DIR}/GD32/Project/
    ${CMAKE_CURRENT_SOURCE_DIR}/GD32/Utilities/
    # ${CMAKE_CURRENT_SOURCE_DIR}/User/led/
)



#  编译参数
target_compile_options(${ELF_TARGET} PRIVATE
    ${CPU_FLAGS}
    -Wall
    -Wextra
    -g 
    -gdwarf-2 
    -mthumb 
    -Os 
    -Wall 
    -fdata-sections 
    -ffunction-sections
    -fmessage-length=0
)


# 链接参数
target_link_options(${ELF_TARGET} PRIVATE
    -T${LINK_SCRIPT}
    ${CPU_FLAGS}
    --specs=nano.specs
    -Wl,--gc-sections
    -Wl,-Map=${CMAKE_PROJECT_NAME}.map
    -Wl,--cref
    -Wl,--print-memory-usage
)


# 添加自定义 makefile 规则，用于生成 hex bin 格式文件
add_custom_command(TARGET ${ELF_TARGET} POST_BUILD
    COMMAND ${CMAKE_OBJCOPY} -O ihex ${ELF_TARGET} ${CMAKE_PROJECT_NAME}.hex
    COMMAND ${CMAKE_OBJCOPY} -O binary -S ${ELF_TARGET} ${CMAKE_PROJECT_NAME}.bin
)


