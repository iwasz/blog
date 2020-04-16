---
ID: 469
post_title: 'Cross compilation with GCC &#038; QtCreator for ARM Cortex M'
author: admin
post_excerpt: ""
layout: post
permalink: >
  http://www.iwasz.pl/uncategorized/cross-compilation-with-gcc-qtcreator-for-arm-cortex-m/
published: true
post_date: 2016-06-21 10:16:27
---
<strong>This post is outdated!!</strong>. Please see http://www.iwasz.pl/electronics/stm32-on-ubuntu-linux-step-by-step/ for more up to date instructions.

I used Eclipse CDT for years for C/C++ and was disappointed by its bulkiness, slowness and memory usage. I did mostly embedded, and sometimes GTK+ desktop apps (even OpenGL once or twice). I looked for a replacement, tried dozen or more IDEs and editors, and finally found out the QtCreator (my main concerns were : great code navigation - eclipse often get confused wit serious, templated C++ code), great code completion, and CMake integration. I am satisfied for now (It's been a year now), and I use it for all but Qt. But all of a sudden, wen a new version appeared, I came across a minor flaw in CMake builder, which reported an error like "Can't link a test program". Obviously he cannot, because he used host compiler instead of ARM one.

So in version prior to 4.0.0 i used to configure my project with cmake like:
<pre>cd build
cmake ..
make</pre>
Then, from Qt, i simply compiled the project, and it worked flawlessly. But since QtCreator 4.0.0 it started to invoke cmake in every possible situation. Be it a IDE startup, or saving a CMakeLists.txt file. And he did it with <em>-DCMAKE_CXX_COMPILER=xyz</em> where "xyz" was a path configured in <em>Tools -&gt; Options -&gt; Build &amp; Run -&gt; Compilers</em>. If I run cmake manually, without this CMAKE_CXX_COMPILER variable set, everything was OK. I saw in the Internet, that many people had the same problem, and used QtCreator for embedded like I do (<a href="https://blog.qt.io/blog/2016/05/11/qt-creator-4-0-0-released/">see comments here</a>).

So I decided, that instead of forcing QtCreator to stop invoking cmake, or invoking it with different parameters, I should fix my CMakeLists.txt so it would run as QtCreator want it. Solution I found was <a href="https://cmake.org/Wiki/CMake_Cross_Compiling#The_toolchain_file">CMAKE_FORCE_C_COMPILER and CMAKE_FORCE_CXX_COMPILER documented here</a>. My CMakeLists.txt looks like this:
<pre>CMAKE_MINIMUM_REQUIRED(VERSION 2.8)

SET (CMAKE_VERBOSE_MAKEFILE OFF)
include (stm32.cmake)

PROJECT (robot1)

SET(USB_LIB "" CACHE STRING "")

INCLUDE_DIRECTORIES("src/")
LIST (APPEND APP_SOURCES "src/main.c")
LIST (APPEND APP_SOURCES "src/stm32f4xx_it.c")
LIST (APPEND APP_SOURCES "src/syscalls.c")
LIST (APPEND APP_SOURCES "src/system_stm32f0xx.c")
LIST (APPEND APP_SOURCES "src/config.h")
LIST (APPEND APP_SOURCES "src/stm32f0xx_hal_conf.h")

AUX_SOURCE_DIRECTORY ("${CUBE_ROOT}/Drivers/STM32F0xx_HAL_Driver/Src/" APP_SOURCES)
LIST (APPEND APP_SOURCES "${STARTUP_CODE}")

ADD_EXECUTABLE(${CMAKE_PROJECT_NAME}.elf ${APP_SOURCES})
ADD_CUSTOM_TARGET(${CMAKE_PROJECT_NAME}.bin ALL DEPENDS ${CMAKE_PROJECT_NAME}.elf COMMAND ${CMAKE_OBJCOPY} -Obinary ${CMAKE_PROJECT_NAME}.elf ${CMAKE_PROJECT_NAME}.bin)

FIND_PROGRAM (ST_FLASH st-flash)
ADD_CUSTOM_TARGET("upload" DEPENDS ${CMAKE_PROJECT_NAME}.elf COMMAND ${ST_FLASH} --reset write ${CMAKE_PROJECT_NAME}.bin 0x8000000)
</pre>
And the "toolchain file" is like this:
<pre>SET (TOOLCHAIN_PREFIX "/home/iwasz/local/share/armcortexm0-unknown-eabi" CACHE STRING "")
SET (TARGET_TRIPLET "armcortexm0-unknown-eabi" CACHE STRING "")
SET (DEVICE "STM32F072xB")
SET (CUBE_ROOT "/home/iwasz/workspace/stm32cubef0")
SET (CRYSTAL_HZ 16000000)
SET (STARTUP_CODE "src/startup_stm32f072xb.s")
SET (LINKER_SCRIPT "${CMAKE_SOURCE_DIR}/src/STM32F072RB_FLASH.ld")
SET (TOOLCHAIN_BIN_DIR ${TOOLCHAIN_PREFIX}/bin)
SET (TOOLCHAIN_INC_DIR ${TOOLCHAIN_PREFIX}/${TARGET_TRIPLET}/include)
SET (TOOLCHAIN_LIB_DIR ${TOOLCHAIN_PREFIX}/${TARGET_TRIPLET}/lib)

INCLUDE(CMakeForceCompiler)
SET (CMAKE_SYSTEM_NAME Generic)
SET (CMAKE_SYSTEM_PROCESSOR arm)

CMAKE_FORCE_C_COMPILER ("${TOOLCHAIN_BIN_DIR}/${TARGET_TRIPLET}-gcc" GNU)
CMAKE_FORCE_CXX_COMPILER ("${TOOLCHAIN_BIN_DIR}/${TARGET_TRIPLET}-gcc" GNU)
SET (CMAKE_ASM_COMPILER "${TOOLCHAIN_BIN_DIR}/${TARGET_TRIPLET}-as")
SET (CMAKE_ASM-ATT_COMPILER "${TOOLCHAIN_BIN_DIR}/${TARGET_TRIPLET}-as")
SET (CMAKE_OBJCOPY ${TOOLCHAIN_BIN_DIR}/${TARGET_TRIPLET}-objcopy)
SET (CMAKE_OBJDUMP ${TOOLCHAIN_BIN_DIR}/${TARGET_TRIPLET}-objdump)
SET (CMAKE_C_FLAGS "-std=gnu99 -fdata-sections -ffunction-sections -Wall" CACHE INTERNAL "c compiler flags")
SET (CMAKE_CXX_FLAGS "-std=c++11 -Wall -fdata-sections -ffunction-sections -MD -Wall" CACHE INTERNAL "cxx compiler flags")
SET (CMAKE_EXE_LINKER_FLAGS "-T ${LINKER_SCRIPT} -Wl,--gc-sections" CACHE INTERNAL "exe link flags")
SET (CMAKE_SHARED_LIBRARY_LINK_C_FLAGS "")
SET (CMAKE_SHARED_LIBRARY_LINK_CXX_FLAGS "")

INCLUDE_DIRECTORIES(${SUPPORT_FILES})
LINK_DIRECTORIES(${SUPPORT_FILES})
ADD_DEFINITIONS(-D${DEVICE})
INCLUDE_DIRECTORIES("${CUBE_ROOT}/Drivers/STM32F0xx_HAL_Driver/Inc/")
INCLUDE_DIRECTORIES("${CUBE_ROOT}/Drivers/CMSIS/Device/ST/STM32F0xx/Include/")
INCLUDE_DIRECTORIES("${CUBE_ROOT}/Drivers/CMSIS/Include/")

ENABLE_LANGUAGE (ASM-ATT)
</pre>
Two most important changes were:
<ul>
 	<li>Using CMAKE_FORCE_CXX_COMPILER macro instead of simply setting CMAKE_CXX_COMPILER var.</li>
 	<li>Including the toolchain file (and thus setting / forcing the compiler) before PROJECT macro.</li>
</ul>
My configuration as of writing this:
<ul>
 	<li>Qt Creator 4.0.2, Based on Qt 5.7.0 (GCC 4.9.1 20140922 (Red Hat 4.9.1-10), 64 bit), Built on Jun 13 2016 01:05:36, From revision 47b4f2c738</li>
 	<li>Host system : Ubuntu 15.10, Linux ingram 4.2.0-38-generic #45-Ubuntu SMP Wed Jun 8 21:21:49 UTC 2016 x86_64 x86_64 x86_64 GNU/Linux</li>
 	<li>Host GCC : gcc (Ubuntu 5.2.1-22ubuntu2) 5.2.1 20151010</li>
 	<li>Target GCC : armcortexm0-unknown-eabi-gcc (crosstool-NG crosstool-ng-1.21.0-74-g6ac93ed - iwasz) 5.2.0</li>
</ul>