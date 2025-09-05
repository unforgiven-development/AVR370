#
# CMakeLists.txt for AVR370
#

cmake_minimum_required(VERSION 3.0)


set(AVR_UPLOADTOOL avrdude)

# set avrdude programmer type and port
# TODO: make these options
set(AVR_PROGRAMMER jtagice3_isp)
set(AVR_UPLOADTOOL_PORT usb)


set(AVR_MCU atmega1284p)
set(AVR_L_FUSE 0x00)
set(AVR_H_FUSE 0x00)
set(AVR_E_FUSE 0x00)


set(AVR_CPU_CLK "16000000UL")

#set(WITH_MCU OFF)

include(cmake/generic-gcc-avr.cmake)

# Set project name
project(AVR370 LANGUAGES C ASM)

set(EXE_OUT_NAME mega_1284p_xplained_example)


########################################################################################################################
# Status messages
########################################################################################################################

message(STATUS "Current upload tool is:          ${AVR_UPLOADTOOL}")
message(STATUS "Current programmer is:           ${AVR_PROGRAMMER}")
message(STATUS "Current upload port is:          ${AVR_UPLOADTOOL_PORT}")
message(STATUS "Current upload tool options are: ${AVR_UPLOADTOOL_OPTIONS}")
message(STATUS "Selected microcontroller:        ${AVR_MCU}")
message(STATUS "Lower fuse byte data:            ${AVR_L_FUSE}")
message(STATUS "Upper fuse byte data:            ${AVR_H_FUSE}")
message(STATUS "Extended fuse byte data:         ${AVR_E_FUSE}")
message(STATUS "CPU frequency:                   ${AVR_CPU_CLK}")


if(NOT CMAKE_BUILD_TYPE)
	set(CMAKE_BUILD_TYPE Release)
	message(STATUS "CMAKE_BUILD_TYPE was not specified; defaulting to Release...")
endif(NOT CMAKE_BUILD_TYPE)


########################################################################################################################
# some cmake cross-compile necessities
########################################################################################################################

if(DEFINED ENV{AVR_FIND_ROOT_PATH})
	set(CMAKE_FIND_ROOT_PATH $ENV{AVR_FIND_ROOT_PATH})
else(DEFINED ENV{AVR_FIND_ROOT_PATH})
	if(EXISTS "/opt/local/avr")
		set(CMAKE_FIND_ROOT_PATH "/opt/local/avr")
	elseif(EXISTS "/usr/avr")
		set(CMAKE_FIND_ROOT_PATH "/usr/avr")
	elseif(EXISTS "/usr/lib/avr")
		set(CMAKE_FIND_ROOT_PATH "/usr/lib/avr")
	elseif(EXISTS "/usr/local/CrossPack-AVR")
		set(CMAKE_FIND_ROOT_PATH "/usr/local/CrossPack-AVR")
	else(EXISTS "/opt/local/avr")
		message(FATAL_ERROR "Please set AVR_FIND_ROOT_PATH in your environment.")
	endif(EXISTS "/opt/local/avr")
endif(DEFINED ENV{AVR_FIND_ROOT_PATH})
set(CMAKE_FIND_ROOT_PATH_MODE_PROGRAM NEVER)
set(CMAKE_FIND_ROOT_PATH_MODE_LIBRARY ONLY)
set(CMAKE_FIND_ROOT_PATH_MODE_INCLUDE ONLY)
# not added automatically, since CMAKE_SYSTEM_NAME is "generic"
set(CMAKE_SYSTEM_INCLUDE_PATH "${CMAKE_FIND_ROOT_PATH}/include")
set(CMAKE_SYSTEM_LIBRARY_PATH "${CMAKE_FIND_ROOT_PATH}/lib")


########################################################################################################################
# status messages for generating
########################################################################################################################

message(STATUS "Set CMAKE_FIND_ROOT_PATH to:      ${CMAKE_FIND_ROOT_PATH}")
message(STATUS "Set CMAKE_SYSTEM_INCLUDE_PATH to: ${CMAKE_SYSTEM_INCLUDE_PATH}")
message(STATUS "Set CMAKE_SYSTEM_LIBRARY_PATH to: ${CMAKE_SYSTEM_LIBRARY_PATH}")


########################################################################################################################
# set compiler options for build types
########################################################################################################################

if(CMAKE_BUILD_TYPE MATCHES Release)
	set(CMAKE_C_FLAGS_RELEASE "-Os")
	set(CMAKE_CXX_FLAGS_RELEASE "-Os")
	add_definitions("-DNDEBUG")
	add_definitions("-UDEBUG")
endif(CMAKE_BUILD_TYPE MATCHES Release)

if(CMAKE_BUILD_TYPE MATCHES RelWithDebInfo)
	set(CMAKE_C_FLAGS_RELWITHDEBINFO "-Os -save-temps -g -gdwarf-3 -gstrict-dwarf")
	set(CMAKE_CXX_FLAGS_RELWITHDEBINFO "-Os -save-temps -g -gdwarf-3 -gstrict-dwarf")
endif(CMAKE_BUILD_TYPE MATCHES RelWithDebInfo)

if(CMAKE_BUILD_TYPE MATCHES Debug)
	set(CMAKE_C_FLAGS_DEBUG "-Og -save-temps -g3 -gdwarf-3 -gstrict-dwarf")
	set(CMAKE_CXX_FLAGS_DEBUG "-Og -save-temps -g3 -gdwarf-3 -gstrict-dwarf")
	add_definitions("-DDEBUG")
	add_definitions("-UNDEBUG")
endif(CMAKE_BUILD_TYPE MATCHES Debug)


########################################################################################################################
# compiler options for all build types
########################################################################################################################

add_definitions("-Wall")
add_definitions("-Werror")
# http://gcc.gnu.org/onlinedocs/gcc-4.8.2/gcc/Alternate-Keywords.html#Alternate-Keywords
# [...]-pedantic and other options cause warnings for many GNU C extensions. You can prevent such warnings within
# one expression by writing __extension__ before the expression. __extension__ has no effect aside from this.[...]
#add_definitions("-pedantic")
#add_definitions("-pedantic-errors")
add_definitions("-funsigned-char")
add_definitions("-funsigned-bitfields")
add_definitions("-fpack-struct")
add_definitions("-fshort-enums")
add_definitions("-ffunction-sections")
add_definitions("-fdata-sections")
add_definitions("-c")
add_definitions("-std=gnu11")


add_definitions("-DF_CPU=${AVR_CPU_CLK}")

add_definitions("-D_QTOUCH_")
add_definitions("-DQT_NUM_CHANNELS=4")
add_definitions("-DQTOUCH_STUDIO_MASKS=1")
add_definitions("-D_DEBUG_INTERFACE_")
add_definitions("-DSNS1=D")
add_definitions("-DSNSK1=D")
add_definitions("-DQT_DELAY_CYCLES=1")
add_definitions("-D_SNS1_SNSK1_SAME_PORT_")


##################################################################################
# add AVR executable
##################################################################################
add_avr_executable(
		${EXE_OUT_NAME}
		mega_1284p_xplained_example.c
		init.c
		usart.c
		qt_asm_tiny_mega.S
		avr_compiler.h
		mega_1284p_xplained_example.h
		init.h
		usart.h
		touch_api.h
		touch_qt_config.h
)

#find_library(M_LIB m)
#message(STATUS "avr-libm: ${M_LIB}")

#find_library(C_LIB c)
#message(STATUS "avr-libc: ${C_LIB}")


#################################################################################
# link library to executable
# NOTE: It needs to be the elf target.
##################################################################################
# target_link_libraries(<executable-name>-${AVR_MCU}.elf <library-name>-${AVR_MCU})
# OR easier
#target_link_libraries(<executable-name> <library-name> <some-external-libary>)
avr_target_link_libraries(${EXE_OUT_NAME} ${CMAKE_SOURCE_DIR}/libavr51g1-4qt-k-0rs.a c m) #${C_LIB} ${M_LIB})
