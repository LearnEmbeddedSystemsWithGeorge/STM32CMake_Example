# STM32CMake_Example
Example of project using CMake

If you are using Windows, you could install MinGW as I used it on my side for testing. You can find a good package here: https://jmeubank.github.io/tdm-gcc/

##Steps:
1. Create out folder inside project root folder where CMakeLists.txt is located.

###Windows:
2. Open GitBash from out folder and run following commands:

3. cmake -S .. -B . -G "MinGW Makefiles"

4. mingw32-make -j8

###Linux:
2. Open a Terminal inside out folder and run following commands:

3. cmake -S .. -B .

4. make -j8

Output will be somehting like this:

Scanning dependencies of target STM32_Template_Demo

[  4%] Building C object CMakeFiles/STM32_Template_Demo.dir/Core/Src/main.c.obj

[  9%] Building C object CMakeFiles/STM32_Template_Demo.dir/Core/Src/stm32f4xx_hal_msp.c.obj

...

[ 95%] Building ASM object CMakeFiles/STM32_Template_Demo.dir/startup_stm32f446xx.s.obj

[100%] Linking C executable STM32_Template_Demo.elf

Memory region         Used Size  Region Size  %age Used

             RAM:        1648 B       128 KB      1.26%

           FLASH:        7816 B       512 KB      1.49%

   text    data     bss     dec     hex filename

   7796      20    1636    9452    24ec E:/Youtube/TestSTM32CMake/Demo/STM32CMake_Example/out/STM32_Template_Demo.elf

[100%] Built target STM32_Template_Demo


Use ST-Link and flash the generated binary file or the hex file.