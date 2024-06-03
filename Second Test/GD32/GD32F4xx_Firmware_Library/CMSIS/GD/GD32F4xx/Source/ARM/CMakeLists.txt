enable_language(ASM)

add_library(arm_cortex_m4_startup
startup_gd32f407_427.S
)

set(LIBRARY_OUTPUT_PATH ${PROJECT_SOURCE_DIR}/lib)


target_compile_options(arm_cortex_m4_startup PRIVATE 
-x
assembler-with-cpp
-mcpu=cortex-m4
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
