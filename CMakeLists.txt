project(FreeRTOS_STM32F7 C CXX ASM)

set(CMAKE_SYSTEM_PROCESSOR arm)

set(ATFE_HOME "/pkg/ATfE-20.1.0-Linux-x86_64")
set(CMAKE_C_COMPILER "${ATFE_HOME}/bin/clang")
set(CMAKE_CXX_COMPILER "${ATFE_HOME}/bin/clang++")
set(CMAKE_ASM_COMPILER "${ATFE_HOME}/bin/clang")
set(CMAKE_OBJCOPY "${ATFE_HOME}/bin/llvm-objcopy")

set(ARM_FLAGS "--target=arm-none-eabi -mcpu=cortex-m7 -mthumb -mfpu=fpv5-d16 -mfloat-abi=hard")
set(CMAKE_C_FLAGS "${ARM_FLAGS} -ffunction-sections -fdata-sections -g -Wall -Og")
set(CMAKE_CXX_FLAGS "${CMAKE_C_FLAGS} -fno-exceptions -fno-rtti")
set(CMAKE_ASM_FLAGS "${CMAKE_C_FLAGS}")

add_compile_options(${CPP_FLAGS} ${CPP_COMMON_FLAGS})
add_link_options(${CPP_FLAGS} -T${CMAKE_SOURCE_DIR}/board/stm32f769i-discovery.ld -Wl,--gc-sections -nostartfiles)

include_directories(
	cmsis_core/CMSIS/Core/Include
	cmsis_f7/Include
	rtos/include
	rtos/portable/GCC/ARM_CM7/r0p1
	app
)

add_executable(app
	cmsis_f7/Source/Templates/system_stm32f7xx.c
	cmsis_f7/Source/Templates/gcc/startup_stm32f769xx.s
	
	rtos/portable/MemMang/heap_4.c
	rtos/portable/GCC/ARM_CM7/r0p1/port.c

	rtos/tasks.c
	rtos/queue.c
	rtos/list.c
	rtos/timers.c

	app/main.cpp
)

add_compile_definitions(app
	PRIVATE
		STM32F769xx
)

add_custom_command(
	TARGET app POST_BUILD
	COMMAND ${CMAKE_OBJCOPY} -O ihex app ${PROJECT_BINARY_DIR}/app.hex
	COMMENT "Generating app.hex"
)