---
ID: 531
post_title: STM32 on Ubuntu Linux step by step
author: admin
post_excerpt: ""
layout: post
permalink: >
  http://www.iwasz.pl/electronics/stm32-on-ubuntu-linux-step-by-step/
published: true
post_date: 2018-11-06 00:26:35
---
This is a step by step tutorial on using STM32 (<strong>stm32f407vg</strong> to be precise) under Linux (Ubuntu 18.10) and other tools (<strong>QtCreator</strong> and <strong>CMake</strong>) that I use in my everyday work. In this article we will compile simple LED blinking program and run it on the <a href="https://www.st.com/en/evaluation-tools/stm32f4discovery.html">STM32F4-DISCOVERY</a>. There may be easier ways of accomplishing this though. Paragraphs ending with asterisk are optional.

First we will try to prepare a project which compiles simply in a terminal using make or ninja. This is very important for me because nowadays every µC vendor seems to provide its own shitty IDE, and their examples tend to compile only under those IDEs in non standar ways. This forces you to install and use stuff that you (or is it only me?) don't like not to mention that most of those tools runs only under Windows. Then I'll show you my favorite IDE which happens to be QtCreator, but any other IDE which can cope with CMake build system should do.

Source code for this article is here : <a href="https://github.com/iwasz/blinky01">https://github.com/iwasz/blinky01</a>

<strong>Note</strong> : do not confuse [Stm32]CubeMX (desktop configuration tool) and [Stm32]CubeF4 which is a SDK package.

<strong>Note to myself</strong> : virtualBox creates way to small disk images by default. I went with 10GB which proved to be to small after only 1 hour of tweaking. Resizing the VB disk is done via vboxmanage, reviving dead distro is done as usual by using LiveCD, and then you delete the partition using fdisk, create new in the place of the old with all options set to defaults, and then you use resize2fs.

<strong>Note to myself 2</strong> : install virtual box guest extensions in the guest system for seamless clipboard operation. Install virtualbox-ext-pack in the host system for USB 2.0 forwarding, add yourself to the vboxusers group, <a href="https://forums.virtualbox.org/viewtopic.php?f=35&amp;t=82639">read this</a>.
<h2>Installing the tools</h2>
<h3>Stm32CubeMX</h3>
<a href="http://www.iwasz.pl/wp-content/uploads/2018/10/VirtualBox_Ubuntu-18.png"><img class="alignright size-medium wp-image-534" src="http://www.iwasz.pl/wp-content/uploads/2018/10/VirtualBox_Ubuntu-18-300x194.png" alt="" width="300" height="194" /></a>Install <a href="http://Ubuntu 18.10">Ubuntu 18.10</a>. I used VirtualBox running on Ubuntu 18.04 for this.

Make sure everything is up to date. Run apt update + upgrade, or use GUI Ubuntu provides.

We are going to use an excellent program <a href="https://www.st.com/en/development-tools/stm32cubemx.html">ST provides called STM32CubeMX</a> which lets us configure pins, clock sources and more. But above all it can generate some startup and config code which we'll take as a base for our LED blinking application. So lets download it and remember to install Java first. I usually install Oracle's Java JDK, but JRE of course will do as well (in this case it was <strong>jdk-11.0.1_linux-x64_bin.deb</strong>). I have no experience with other Java implementations like OpenJDK but I suspect, that it also will work as expected.

After jdk was installed I added java to the PATH in ~/.profile : PATH="/usr/lib/jvm/jdk-11.0.1/bin:$PATH"

Download <a href="https://www.st.com/en/development-tools/stm32cubemx.html">Stm32CubeMX</a> and unpack it (login required unfortunately). Run ./SetupSTM32CubeMX-4.27.0.linux . In my case (fresh Ubuntu) it said :
<pre>./SetupSTM32CubeMX-4.27.0.linux 
bash: ./SetupSTM32CubeMX-4.27.0.linux: No such file or directory</pre>
It is very non intuitive message, but it is because of 32 bit libraries missing. <a href="https://blog.teststation.org/ubuntu/2016/05/12/installing-32-bit-software-on-ubuntu-16.04/">This post tells us what</a> to do, and to my surprise it discourages from installing ia32-libs which I would normally do:
<pre>file SetupSTM32CubeMX-4.27.0.linux
SetupSTM32CubeMX-4.27.0.linux: ELF 32-bit LSB executable, Intel 80386, ......
sudo dpkg --add-architecture i386
sudo apt update
sudo apt-get install libc6:i386 libstdc++6:i386</pre>
Run the installer and verify, that CubeMX works.
<h3>QtCreator*</h3>
Everyone has his/her favorite IDE, but mine is QtCreator for various reasons which I'm not going to dive into, but Qt libraries are not one of them. I do not use Qt, I simply tried many IDE's and QtCreator suits me the best. First lets grab an installer.
<ul>
 	<li><a href="https://download.qt.io/official_releases/qtcreator/">https://download.qt.io/official_releases/qtcreator/</a> - those are the official releases.</li>
 	<li><a href="https://download.qt.io/snapshots/qtcreator/">https://download.qt.io/snapshots/qtcreator/</a> - and here are nightly builds.</li>
</ul>
For this article I picked <a href="https://download.qt.io/official_releases/qtcreator/4.7/4.7.2/qt-creator-opensource-linux-x86_64-4.7.2.run">qt-creator-opensource-linux-x86_64-4.7.2.run</a> and run it in the terminal and that's it (login required).
<h3>Toolchain</h3>
Toolchain can be easily installed from Launchpad PPA, or can be compiled using <a href="https://crosstool-ng.github.io/">excellent tool called crosstool-ng</a>. Detailed instructions are in <a href="http://www.iwasz.pl/electronics/toolchain-for-cortex-m4/">one of my previous posts</a>. But for now lets use the easier way:
<pre>sudo add-apt-repository ppa:team-<wbr />gcc-arm-<wbr />embedded/<wbr />ppa
sudo apt-get update
sudo apt install gcc-arm-none-eabi binutils-arm-none-eabi libnewlib-arm-none-eabi libstdc++-arm-none-eabi-newlib
sudo apt install cmake ninja</pre>
<h3>Other tools</h3>
<pre>sudo apt install mc openocd dos2unix gdb-multiarch</pre>
<h2>The project</h2>
<a href="http://www.iwasz.pl/wp-content/uploads/2018/11/Screenshot-from-2018-11-02-12-42-39.png"><img class="alignright size-medium wp-image-549" src="http://www.iwasz.pl/wp-content/uploads/2018/11/Screenshot-from-2018-11-02-12-42-39-300x206.png" alt="" width="300" height="206" /></a>Run Stm32CubeMX and start a new project. In the "Part Number Search" in the top left corner insert "stm32f407vg". This is the model of a µC we will be using and which is mounted in the STM32F4-DISCOVERY, a popular evaluation board (you can get it from all major distributors, though <a href="https://www.findchips.com/search/stm32f4-discovery">findchips.com shows</a>, that availability is not at its best right now).

Click on the blue link-like label in the search result table and start the project. The next thing you'll see is a view of your microcontroller with all the peripherals and GPIOs initialized according to the boards specs. This is because the board itself contains some neat stuff like accelerometer, digital to analog audio chip with amplifier and so on. We are focused on PD12 - PD15 which are connected directly to LEDs.

<a href="http://www.iwasz.pl/wp-content/uploads/2018/11/Screenshot-from-2018-11-05-13-32-46.png"><img class="alignright size-medium wp-image-551" src="http://www.iwasz.pl/wp-content/uploads/2018/11/Screenshot-from-2018-11-05-13-32-46-300x139.png" alt="" width="300" height="139" /></a>Turn off USB support. It uses some additional files which makes compilation a little bit harder. Make sure that USB_OTG_FS Mode is set to "Disable" in the left pane. Also on the "Configuration" tab make sure, that USB middleware is not used.

Turn off other unnecessary peripherals like SPI, I2C, USARTs. The more the peripherals, the more source files from SDK we will have to compile, so in my project only SYS, RCC and of course GPIOs are configured. Please check my <a href="https://github.com/iwasz/blinky01/blob/master/blinky.ioc">blinky.ioc</a> in case of trouble or simply experiment with CubeMx's output.

Then use "project -&gt; Generate Code" in the main menu to generate the code. When asked abut downloading the SDK called StmCubeF4 click YES, and appropriate SDK will be placed in the ~/STM32Cube/Repository. This is important as our code will depend on it (although the SDK can be separately <a href="https://www.st.com/en/embedded-software/stm32cubef4.html">downloaded from here</a>). Here's what the generated directory structure looks like on my computer:
<pre>./EWARM : <em>some stuff for some proprietary IDE i guess.</em>
./blinky.ioc : <em>CubeMX project file.</em>
./Middlewares
./Middlewares/ST : <em>USB library which won't be of interest for us since we only want to blink LEDs.</em>

./Drivers : <em>Parts of the CubeF4 gets copied here.</em> 
./Drivers/STM32F4xx_HAL_Driver : <em>Peripheral library.</em>
./Drivers/CMSIS : <em>CMSIS Library is a low level code for interfacing with the CPU.</em>

./Inc 
./Inc/stm32f4xx_it.h : IRQ handlers declarations used by our app. Not particularly useful.
./Inc/stm32f4xx_hal_conf.h : <em>low level peripheral configuration based on CubeMX options.</em>
./Inc/main.h : <em>another unnecessary file.</em>
./Src
./Src/system_stm32f4xx.c : <em>Low level init routines run from the startup code before main.</em>
./Src/stm32f4xx_it.c : <em>IRQ handler definitions.</em>
./Src/stm32f4xx_hal_msp.c : <em>hi level peripheral configuration based on CubeMX options (GPIOs etc).</em>
./Src/main.c</pre>
So as you can see pretty verbose output gets produced from CubeMX, but we are interested only in the *.h and *.c files. My favorite directory structure at the other hand looks like this one below (for completeness sake, the directory resides in ~/workspace/blinky but of course dir names are up to you). Create it, and copy generated sources into src directory as so:
<pre>./build : <em>Compiled files goes here. Never commit this dir as it's contents are generated.</em>
./stm32f407xx.cmake : <em>The toolchain file, more on it later.</em>
./src : <em>Sources generated by the CubeMX. *.h and *.c together, but it's up to you.</em>
./src/stm32f4xx_it.h
./src/system_stm32f4xx.c
./src/stm32f4xx_hal_conf.h
./src/stm32f4xx_it.c
./src/stm32f4xx_hal_msp.c
./src/main.h
./src/main.c
./CMakeLists.txt : <em>The CMake file.</em></pre>
One can argue whether STM32F4xx_HAL_Driver and CMSIS should be inside our project tree or not. I prefer having all parts of SDK in some external directories simply because copying them into dozens of projects would be a waste and a pollution of the source tree. At the other hand though packaging all the necessary code into our project makes it self contained and easier to compile (no external deps. your choice). For this article I assume that SDK resides in the ~/STM32Cube/Repository/STM32Cube_FW_F4_V1.21.0 (the default for CubeMX). So now you have two files missing : <strong>stm32f407xx.cmake</strong> and <strong>CMakeLists.txt</strong>. The former looks like this:
<pre lang="cmake"># This variable is used later to set a C/C++ macro which tells CubeF4 which µC to use. ST
# drivers, and other code is bloated with all sorts of macros, #defines and #ifdefs.
SET (DEVICE "STM32F407xx")

# This is a variable which is later used here and in the CMakeLists.txt. It simply tells
# where to find the SDK (CubeF4). Please change it accordingly if you have other 
# version of CubeF4 installed.
SET (CUBE_ROOT "$ENV{HOME}/STM32Cube/Repository/STM32Cube_FW_F4_V1.21.0")

# Startup code and linker script - more on it later.
SET (STARTUP_CODE "${CUBE_ROOT}/Projects/STM32F4-Discovery/Templates/SW4STM32/startup_stm32f407xx.s")
SET (LINKER_SCRIPT "${CUBE_ROOT}/Projects/STM32F4-Discovery/Templates/SW4STM32/STM32F4-Discovery/STM32F407VGTx_FLASH.ld")

# Magic settings. Without it CMake tries to run test programs on the host platform, which
# fails of course.
SET (CMAKE_SYSTEM_NAME Generic)
SET (CMAKE_SYSTEM_PROCESSOR arm)

# -mcpu tells which CPU to target obviously. -fdata-sections -ffunction-sections Tells GCC to.
# get rid of unused code in the output binary. -Wall produces verbose warnings.
SET(CMAKE_C_FLAGS "-mcpu=cortex-m4 -std=gnu99 -fdata-sections -ffunction-sections -Wall" CACHE INTERNAL "c compiler flags")

# Flags for g++ are used only when compliing C++ sources (*.cc, *.cpp etc). -std=c++17 Turns
# on all the C++17 goodies, -fno-rtti -fno-exceptions turns off rtti and exceptions.
SET(CMAKE_CXX_FLAGS "-mcpu=cortex-m4 -std=c++17 -fno-rtti -fno-exceptions -Wall -fdata-sections -ffunction-sections -MD -Wall" CACHE INTERNAL "cxx compiler flags")

# Those flags gets passed into the linker which is run by the GCC at he end of the process..
# -T tells the linker which LD script to use, -specs=nosys.specs sets the specs which most 
# notably tells the compiler to use libnosys.a which contains all the syscalls like _sbrk,
# _exit and much more. They are more like an interface between our program and operating system /
# bare metal system we are running it on. You can use rdimon.specs instead or write syscalls 
# yourself which for bare-metal isn't difficult. --gc-sections strips out unused code from 
#binaries I think.
SET (CMAKE_EXE_LINKER_FLAGS "-T ${LINKER_SCRIPT} -specs=nosys.specs -Wl,--gc-sections" CACHE INTERNAL "exe link flags")

# Some directories in the GCC tree.
INCLUDE_DIRECTORIES(${SUPPORT_FILES})
LINK_DIRECTORIES(${SUPPORT_FILES})

# Macro I wrote about in the first line.
ADD_DEFINITIONS(-D${DEVICE})

# Random include paths for CubeF4 peripheral drivers and CMSIS.
INCLUDE_DIRECTORIES("${CUBE_ROOT}/Drivers/STM32F4xx_HAL_Driver/Inc/")
INCLUDE_DIRECTORIES("${CUBE_ROOT}/Drivers/CMSIS/Device/ST/STM32F4xx/Include/")
INCLUDE_DIRECTORIES("${CUBE_ROOT}/Drivers/CMSIS/Include/")</pre>
Few more words on the linker script (LD script) and the startup code. I won't dive into detail to much, but LD script tells the linker how to assemble the final executable from all the object files that were compiled by a compiler. Basically for every *.c or *cc file one object (*.o or *.obj) file gets created (you can find them later inside build/CMakeFiles dir), and every of them is full of symbols stored in the "input" sections. LD script tells the linker what input symbols from which input sections should be copied into the "output" sections in the final executable (and much more; LD scripts are powerful).

Cortex-M4 CPU after powering up looks at memory address 0 which tells it where the stack starts. Then at the address 0x0000'0004 it checks where the reset IRQ routine is and runs it. Reset IRQ is the first of the many IRQ handlers which addresses reside in the IRQ vector defined in the startup (assembly file). Reset IRQ is called Reset_Handler in ST's assembly startup code and does a few simple things:
<ul>
 	<li>Copies the <strong>.data</strong> section from flash to RAM. .data section contains global initialized variables.</li>
 	<li>Clears the <strong>.bss</strong> RAM section which contains uninitialized static variables.</li>
 	<li>Calls <strong>SystemInit</strong> which I mentioned previously (basic stuff like external memory controller maybe?)</li>
 	<li>Calls <strong>__libc_init_array</strong> which among other things that I'm unaware of calls C++ constructors.</li>
 	<li>Calls <strong>main</strong> function.</li>
</ul>
Writing startup code and LD script is not trivial (although startup code can be written in C), so thankfully ST provided us with their own implementations which we can use. <a href="https://jacobmossberg.se/posts/2018/08/11/run-c-program-bare-metal-on-arm-cortex-m3.html">There is an excellent article on the subject for those who are interested</a>. Go to the ~/STM32Cube/Repository/STM32Cube_FW_F4_V1.21.0 and issue a find :
<pre>find ~/STM32Cube/Repository/STM32Cube_FW_F4_V1.21.0/ -name startup_stm32f407xx.s
/home/iwasz/STM32Cube/Repository/STM32Cube_FW_F4_V1.21.0/Projects/STM32F4-Discovery/Templates/SW4STM32/startup_stm32f407xx.s</pre>
There's a lots of hits because the CubeF4 SDK is made so it can be used with all of STM evaluation boards which there are tons of. This file which we found though, seems to be prepared for our board (Stm32F4-DISCOVERY or sometimes Stm32F4-DISCO for short). Secondly they provide projects for 3 or 4 major (in their opinion) IDEs. Files prepared for TrueStudio and SW4STM32 are the way to go as those are GCC based apparently (never used them).

For finding the LD script issue:
<pre>find ~/STM32Cube/Repository/STM32Cube_FW_F4_V1.21.0/ -name STM32F407*.ld 
/home/iwasz/STM32Cube/Repository/STM32Cube_FW_F4_V1.21.0/Projects/STM32F4-Discovery/Templates/SW4STM32/STM32F4-Discovery/STM32F407VGTx_FLASH.ld</pre>
Modify <strong>stm32f407xx.cmake</strong> according to your findings (STARTUP_CODE and LINKER_SCRIPT variables) though if you followed my instructions directly, you should be fine without modifications.

Next the CMakeLists.txt file. This is like Makefile but on the higher level of abstraction. Mine looks like this:
<pre lang="cmake">CMAKE_MINIMUM_REQUIRED(VERSION 2.8)

PROJECT (blinky)

# Startup code is written by ST in assembly, so without this statement there are errors.
ENABLE_LANGUAGE (ASM-ATT)

INCLUDE_DIRECTORIES("src/")

# Resonator used in this project. Stm32F4-DISCO uses 8MHz crystal. I left this definition here
# in the CMakeLists.txt rather than the toolchain file, because it's project dependent, not
# "platform" dependent, where by platform I mean STM32F4.
ADD_DEFINITIONS (-DHSE_VALUE=8000000)

# All the sources goes here. Adding headers isn't obligatory, but since QtCreator treats CMakeLists.txt as
# its "project configuration" it simply makes header files appear in the source tree pane.
ADD_EXECUTABLE(${CMAKE_PROJECT_NAME}.elf
 "src/main.c"
 "src/main.h"
 "src/stm32f4xx_hal_conf.h"
 "src/stm32f4xx_hal_msp.c"
 "src/stm32f4xx_it.c"
 "src/stm32f4xx_it.h"
 "src/system_stm32f4xx"
)

# Workaround : splitting C and C++ code helps QtCreator parse header files correctly. Without it, QtCreator
# sometimes treats C++ as C and vice versa. EDIT : this comment was written when in the ADD_EXECUTABLE C++
# files were present.
add_library ("stm" STATIC
 "${CUBE_ROOT}/Drivers/STM32F4xx_HAL_Driver/Src/stm32f4xx_hal.c"
 "${CUBE_ROOT}/Drivers/STM32F4xx_HAL_Driver/Src/stm32f4xx_hal_cortex.c"
 "${CUBE_ROOT}/Drivers/STM32F4xx_HAL_Driver/Src/stm32f4xx_hal_gpio.c"
 "${CUBE_ROOT}/Drivers/STM32F4xx_HAL_Driver/Src/stm32f4xx_hal_rcc.c"
 "${CUBE_ROOT}/Drivers/STM32F4xx_HAL_Driver/Src/stm32f4xx_ll_gpio.c"
 "${CUBE_ROOT}/Drivers/STM32F4xx_HAL_Driver/Src/stm32f4xx_ll_rcc.c"
 "${CUBE_ROOT}/Drivers/STM32F4xx_HAL_Driver/Src/stm32f4xx_ll_utils.c"
 "${CUBE_ROOT}/Drivers/STM32F4xx_HAL_Driver/Inc/stm32f4xx_hal_cortex.h"
 "${CUBE_ROOT}/Drivers/STM32F4xx_HAL_Driver/Inc/stm32f4xx_hal_gpio_ex.h"
 "${CUBE_ROOT}/Drivers/STM32F4xx_HAL_Driver/Inc/stm32f4xx_hal_gpio.h"
 "${CUBE_ROOT}/Drivers/STM32F4xx_HAL_Driver/Inc/stm32f4xx_hal.h"
 "${CUBE_ROOT}/Drivers/STM32F4xx_HAL_Driver/Inc/stm32f4xx_hal_rcc_ex.h"
 "${CUBE_ROOT}/Drivers/STM32F4xx_HAL_Driver/Inc/stm32f4xx_hal_rcc.h"
 "${CUBE_ROOT}/Drivers/STM32F4xx_HAL_Driver/Inc/stm32f4xx_ll_cortex.h"
 "${CUBE_ROOT}/Drivers/STM32F4xx_HAL_Driver/Inc/stm32f4xx_ll_gpio.h"
 "${CUBE_ROOT}/Drivers/STM32F4xx_HAL_Driver/Inc/stm32f4xx_ll_rcc.h"
 "${CUBE_ROOT}/Drivers/STM32F4xx_HAL_Driver/Inc/stm32f4xx_ll_system.h"
 "${STARTUP_CODE}"
)

# This links both pieces together.
TARGET_LINK_LIBRARIES (${CMAKE_PROJECT_NAME}.elf -Wl,--whole-archive stm -Wl,--no-whole-archive)

FIND_PROGRAM (OPENOCD openocd)
ADD_CUSTOM_TARGET("upload" DEPENDS ${CMAKE_PROJECT_NAME}.elf COMMAND ${OPENOCD} -f /usr/share/openocd/scripts/interface/stlink-v2.cfg -f /usr/share/openocd/scripts/target/stm32f4x.cfg -c 'program ${CMAKE_PROJECT_NAME}.elf verify reset exit')</pre>
OK now that we are set, lets create a build directory if it doesn't exist yet, and build our project:
<pre>pwd
/home/iwasz/workspace/blinky
mkdir build
cd build
cmake -DCMAKE_CXX_COMPILER=arm-none-eabi-g++ \
    -DCMAKE_C_COMPILER=arm-none-eabi-gcc \
    -DCMAKE_TOOLCHAIN_FILE=../stm32f407xx.cmake -GNinja ..
ninja</pre>
If all went well, blinky.elf should appear in the build directory. Ninja isn't mandatory (but its faster, more modern and jazzy), so if you omit <strong>-GNinja</strong> part, you will be left with classic Makefile and you would issue make instead of ninja at the end.

Feel free to post a comment in case of any trouble, we will sort it out.
<h2>Uploading</h2>
[caption id="attachment_563" align="alignnone" width="512"]<a href="http://www.iwasz.pl/wp-content/uploads/2018/11/blinky.gif"><img class="wp-image-563 size-full" src="http://www.iwasz.pl/wp-content/uploads/2018/11/blinky.gif" alt="" width="512" height="288" /></a> Wires are for some other project. This board has been through a lot.[/caption]

I don't know why but, most of the Internet and books says "downloading" to describe the process of transmitting a binary firmware from host (PC) to the target (µC). I find it very confusing, because when a file is moved from a PC to some remote server, everybody calls it "uploading" not "downloading".

Lets modify our generated code, so it actually blinks. Locate the main loop in the main.c file (it will be empty), and place this inside, and recompile (simply issue ninja or make inside the build dir):
<pre lang="c"> HAL_GPIO_WritePin(GPIOD, LD4_Pin|LD3_Pin|LD5_Pin|LD6_Pin, GPIO_PIN_RESET);
 HAL_Delay (500);
 HAL_GPIO_WritePin(GPIOD, LD4_Pin|LD3_Pin|LD5_Pin|LD6_Pin, GPIO_PIN_SET);
 HAL_Delay (500);</pre>
In the "other tools" section we installed openocd, so lets use it now. Connect the STM32F4-DISCO, make sure that ST-LINK and JP1 jumpers are closed (the default), and :
<pre>cd build
ninja
openocd -f /usr/share/openocd/scripts/interface/stlink-v2.cfg -f /usr/share/openocd/scripts/target/stm32f4x.cfg -c 'program blinky.elf verify reset exit'</pre>
If everything went OK, you should see like a two dozens of messages like :
<pre>adapter speed: 8000 kHz
** Programming Started **
auto erase enabled
Info : device id = 0x10016413
Info : flash size = 1024kbytes
[...]
** Programming Finished **
** Verify Started **
[...]
verified 7992 bytes in 0.104675s (74.561 KiB/s)</pre>
Let me know in case of any problems, we can work it out probably. Remember that I managed to flash the thing under Ubuntu 18.10 running inside Ubuntu 18.04, so it cannot be that difficult :D. Another way of flashing STM32s under Linux is by using <a href="https://github.com/texane/stlink">Texane's st-link</a>, but I found openocd to be more reliable and universal.

Oh, and I added an "upload" target in the CMakeLists.txt, so you can simply do "ninja upload" instead of running openocd manually.
<h2>The QtCreator IDE*</h2>
Now that our project compiles and runs in a console we can integrate it with QtCreator (or other IDE). Run it, and open <strong>Help -&gt; About plugins</strong>. Make sure, that BareMetal (experimental) plugin is active and restart the IDE when asked.

<a href="http://www.iwasz.pl/wp-content/uploads/2018/11/plugins-qtcreator.png"><img class="size-medium wp-image-567 aligncenter" src="http://www.iwasz.pl/wp-content/uploads/2018/11/plugins-qtcreator-300x202.png" alt="" width="300" height="202" /></a>

Now open <strong>Tools -&gt; Options</strong> from the main menu.

In the <strong>Devices</strong> section go to <strong>Bare Metal</strong> tab and <strong>Add</strong> OpenOCD GDB server provider. Defaults are OK, don't change anything.

<a href="http://www.iwasz.pl/wp-content/uploads/2018/11/bare-metal-gdb-provider.png"><img class="aligncenter size-medium wp-image-571" src="http://www.iwasz.pl/wp-content/uploads/2018/11/bare-metal-gdb-provider-300x187.png" alt="" width="300" height="187" /></a>Apply changes and move to the <strong>Devices</strong> tab. Add new <strong>Bare Metal Device</strong> , name it accordingly (I named it Stm32F4) and pick OpenOCD GDB Server provider we created in the previous step.<a href="http://www.iwasz.pl/wp-content/uploads/2018/11/Screenshot-from-2018-11-05-23-38-54.png"><img class="aligncenter size-medium wp-image-572" src="http://www.iwasz.pl/wp-content/uploads/2018/11/Screenshot-from-2018-11-05-23-38-54-300x187.png" alt="" width="300" height="187" /></a>Go to <strong>Kits</strong> section, <strong>Compilers</strong> tab, and make sure GCC for ARM 32 got auto-detected. If it weren't (because you installed some other GCC based toolchain in some non-standard place) add it there using the <strong>Add</strong> button.<a href="http://www.iwasz.pl/wp-content/uploads/2018/11/Screenshot-from-2018-11-05-23-41-52.png"><img class="aligncenter size-medium wp-image-573" src="http://www.iwasz.pl/wp-content/uploads/2018/11/Screenshot-from-2018-11-05-23-41-52-300x206.png" alt="" width="300" height="206" /></a>Go to the <strong>Debuggers</strong> tab and add gdb-multiarch which we installed previously like so (remember to Apply after each modification):<a href="http://www.iwasz.pl/wp-content/uploads/2018/11/Screenshot-from-2018-11-05-23-55-55.png"><img class="aligncenter size-medium wp-image-574" src="http://www.iwasz.pl/wp-content/uploads/2018/11/Screenshot-from-2018-11-05-23-55-55-300x206.png" alt="" width="300" height="206" /></a>Finally go to the <strong>Kits</strong> tab and add new. Pick a name (you can even add an icon), change <strong>Device type</strong> to Bare Metal Device, and pick proper one in the combo below it. In the <strong>Compiler</strong> combos pick the ones we created / found in the Compilers tab, and do the same with the <strong>Debugger</strong> combo.

Change <strong>CMake Configuration</strong> (last row) so it looks like this (those are the options we passed to the CMake using -D flag):<a href="http://www.iwasz.pl/wp-content/uploads/2018/11/Screenshot-from-2018-11-06-00-12-57.png"><img class="aligncenter size-medium wp-image-576" src="http://www.iwasz.pl/wp-content/uploads/2018/11/Screenshot-from-2018-11-06-00-12-57-300x106.png" alt="" width="300" height="106" /></a>After all this effort you should be left with a new kit looking like that:<a href="http://www.iwasz.pl/wp-content/uploads/2018/11/Screenshot-from-2018-11-06-00-02-25.png"><img class="aligncenter size-medium wp-image-575" src="http://www.iwasz.pl/wp-content/uploads/2018/11/Screenshot-from-2018-11-06-00-02-25-300x229.png" alt="" width="300" height="229" /></a>Now we can open our project. Delete old build directory to be sure old config doesn't break anything and open the project by selecting CMakeLists.txt. If you don't delete our old build directory created by hand in previous paragraphs, new "temporary" kit gets created and it can be used after some modifications, but we already have better one.

You will be presented with <strong>Configure Project</strong> window where you can pick a kit to be used. Uncheck the <strong>Desktop</strong> kit, and check our <strong>Stm32F4</strong> one:

<a href="http://www.iwasz.pl/wp-content/uploads/2018/11/Screenshot-from-2018-11-06-00-56-24.png"><img class="aligncenter size-medium wp-image-579" src="http://www.iwasz.pl/wp-content/uploads/2018/11/Screenshot-from-2018-11-06-00-56-24-300x175.png" alt="" width="300" height="175" /></a>Click <strong>Configure</strong> project and hit Ctrl-B to verify that everything compiles.
<h2>Debugging in QtCreator*</h2>
It's simple. Open a terminal, and run openocd like so:
<pre>openocd -f /usr/share/openocd/scripts/interface/stlink-v2.cfg -f /usr/share/openocd/scripts/target/stm32f4x.cfg</pre>
This way GDB remote protocol server is started (not sure about the name), and "normal" GDB can communicate with it. Switch to QtCreator, hit <strong>F5</strong> and that's it! Hit <strong>Shift-F5</strong> to pause program, and you will be presented with a call stack, variables, and so On. <strong>F10</strong> is for stepping over, <strong>F11</strong> for stepping into, and <strong>Shift-F11</strong> for stepping out. Happy debugging.

<a href="http://www.iwasz.pl/wp-content/uploads/2018/11/Screenshot-from-2018-11-06-01-26-00.png"><img class="aligncenter size-medium wp-image-580" src="http://www.iwasz.pl/wp-content/uploads/2018/11/Screenshot-from-2018-11-06-01-26-00-300x179.png" alt="" width="300" height="179" /></a>