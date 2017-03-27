## Blinky, your "Hello World!", on Arduino Primo

<br>

### Objective

Learn how to use packages from a default application repository of Mynewt to build your first *Hello World* application (Blinky) on a target board. Once built using the *newt* tool, this application will blink the LED lights on the target board.

Create a project with a simple app that blinks an LED on the Arduino Primo board.  Download the application to the target and watch it blink!

Note that the Mynewt OS will run on the nRF52 chip in the Arduino Primo board. However, the board support package for the Arduino Primo is different from the nRF52 dev kit board support package.

<br>

### Prerequisites
Ensure that you have met the following prerequisites before continuing with this tutorial:

* Have an Arduino Primo
* Have Internet connectivity to fetch remote Mynewt components.
* Have a computer to build a Mynewt application and connect to the` board over USB.
* Have a Micro-USB cable to connect the board and the computer.
* Install the Newt tool and toolchains (See [Basic Setup](/os/get_started/get_started.md)).
* Create a project space (directory structure) and populated it with the core code repository (apache-mynewt-core) or know how to as explained in [Creating Your First Project](/os/get_started/project_create).
* Read the Mynewt OS [Concepts](/os/get_started/vocabulary.md) section.
* Install a debugger - choose one of the two options below. Option 1 requires additional hardware but very easy to set up. Option 2 is free software install but not as simple as Option 1.

<br>

##### Option 1

* [Segger J-Link Debug Probe](https://www.segger.com/jlink-debug-probes.html) - any model (this tutorial has been tested with J-Link EDU and J-Link Pro)
* [J-Link 9 pin Cortex-M Adapter](https://www.segger.com/jlink-adapters.html#CM_9pin) that allows JTAG, SWD and SWO connections between J-Link and Cortex M based target hardware systems

##### Option 2

 No additional hardware is required but a version of OpenOCD 0.10.0 that is currently in development needs to be installed. A patch for the nRF52 has been applied to the OpenOCD code in development and a tarball has been made available for download [here](downloads/openocd-wnrf52.tgz). Untar it. From the top of the directory tree ("openocd-code-89bf96ffe6ac66c80407af8383b9d5adc0dc35f4"), build it using the following configuration:

```
$./configure --enable-cmsis-dap --enable-openjtag_ftdi --enable-jlink --enable-stlink
```

Then run `make` and `sudo make install`. This step takes minutes, so be patient.

```
$ openocd -v
Open On-Chip Debugger 0.10.0-dev-snapshot (2016-05-20-10:43)
Licensed under GNU GPL v2
For bug reports, read
    http://openocd.org/doc/doxygen/bugs.html
```
Next, make sure that you have checked out the newt develop branch and rebuilt newt.
```
$ cd $GOPATH/src/mynewt.apache.org/newt
$ git checkout develop
$ git pull
$ cd newt
$ go install
```
**Note:** This step can be removed once the changes have been pushed to master.

You can now use openocd to upload to Arduino Primo board via the USB port itself.



<br>


### Install jlinkEXE 

In order to be able to communicate with the SEGGER J-Link debugger on the dev board, you have to download and install the J-Link GDB Server software on to your laptop. You may download the "Software and documentation pack for Mac OS X" from [https://www.segger.com/jlink-software.html](https://www.segger.com/jlink-software.html). 

<br>

### Create a Project  
Create a new project if you do not have an existing one.  You can skip this step and proceed to [create the targets](#create_targets) if you already created a project.

Run the following commands to create a new project:

```no-highlight
    $ mkdir ~/dev
    $ cd ~/dev
    $ newt new myproj
    Downloading project skeleton from apache/incubator-mynewt-blinky...
    Installing skeleton in myproj...
    Project myproj successfully created.
    $ cd myproj
    $ newt install
    apache-mynewt-core
    $
```

<br>
### <a name="create_targets"></a>Create the Targets

Create two targets for the Arduino Primo board - one for the bootloader and one for the Blinky application.

Run the following `newt target` commands, from your project directory, to create a bootloader target. We name the target `primo_boot`.

```no-highlight
$ newt target create primo_boot
$ newt target set primo_boot app=@apache-mynewt-core/apps/boot bsp=@apache-mynewt-core/hw/bsp/arduino_primo_nrf52 build_profile=optimized
```
<br>
Run the following `newt target` commands to create a target for the Blinky application. We name the target `primoblinky`.
```no-highlight
$ newt target create primoblinky
$ newt target set primoblinky app=apps/blinky bsp=@apache-mynewt-core/hw/bsp/arduino_primo_nrf52 build_profile=debug
```
<br>
If you are using openocd, run the following `newt target set` commands:

```no-highlight
$ newt target set primoblinky syscfg=OPENOCD_DEBUG=1
$ newt target set primo_boot syscfg=OPENOCD_DEBUG=1
```

<br>
You can run the `newt target show` command to verify the target settings:

```no-highlight
$ newt target show
targets/my_blinky_sim
    app=apps/blinky
    bsp=@apache-mynewt-core/hw/bsp/native
    build_profile=debug
targets/primo_boot
    app=@apache-mynewt-core/apps/boot
    bsp=@apache-mynewt-core/hw/bsp/arduino_primo_nrf52
    build_profile=optimized
targets/primoblinky
    app=@apache-mynewt-core/apps/blinky
    bsp=@apache-mynewt-core/hw/bsp/arduino_primo_nrf52
    build_profile=optimized
```


<br>

### Build the Target Executables 
Run the `newt build primo_boot` command to build the bootloader:
```no-highlight
$ newt build primo_boot
Building target targets/primo_boot
Compiling repos/apache-mynewt-core/boot/bootutil/src/image_rsa.c
Compiling repos/apache-mynewt-core/boot/bootutil/src/image_ec256.c
Compiling repos/apache-mynewt-core/crypto/mbedtls/src/aes.c
Compiling repos/apache-mynewt-core/apps/boot/src/boot.c
Compiling repos/apache-mynewt-core/boot/bootutil/src/image_ec.c
Compiling repos/apache-mynewt-core/boot/bootutil/src/loader.c
Compiling repos/apache-mynewt-core/boot/bootutil/src/bootutil_misc.c

      ...

Archiving sys_mfg.a
Archiving sys_sysinit.a
Archiving util_mem.a
Linking ~/dev/myproj/bin/targets/primo_boot/app/apps/boot/boot.elf
Target successfully built: targets/primo_boot
```
<br>
Run the `newt build primoblinky` command to build the Blinky application:

```no-highlight
$ newt build primoblinky
Building target targets/primoblinky
Compiling repos/apache-mynewt-core/hw/drivers/uart/src/uart.c
Assembling repos/apache-mynewt-core/hw/bsp/arduino_primo_nrf52/src/arch/cortex_m4/gcc_startup_nrf52.s
Compiling repos/apache-mynewt-core/hw/bsp/arduino_primo_nrf52/src/sbrk.c
Compiling repos/apache-mynewt-core/hw/cmsis-core/src/cmsis_nvic.c
Assembling repos/apache-mynewt-core/hw/bsp/arduino_primo_nrf52/src/arch/cortex_m4/gcc_startup_nrf52_split.s
Compiling apps/blinky/src/main.c
Compiling repos/apache-mynewt-core/hw/drivers/uart/uart_bitbang/src/uart_bitbang.c
Compiling repos/apache-mynewt-core/hw/bsp/arduino_primo_nrf52/src/hal_bsp.c


Archiving sys_mfg.a
Archiving sys_sysinit.a
Archiving util_mem.a
Linking ~/dev/myproj/bin/targets/primoblinky/app/apps/blinky/blinky.elf
Target successfully built: targets/primoblinky
```

<br>

### Sign and Create the Blinky Application Image 

Run the `newt create-image primoblinky 1.0.0` command to create and sign the application image. You may assign an arbitrary version (e.g. 1.0.0) to the image.

```no-highlight
$ newt create-image primoblinky 1.0.0
App image succesfully generated: ~/dev/myproj/bin/targets/primoblinky/app/apps/blinky/blinky.img

```

<br>

### Connect to the Board
* Connect a micro USB cable to the Arduino Primo board and to your computer's USB port.
* If you are using the Segger J-Link debug probe, connect the debug probe to the JTAG port on the Primo board using the Jlink 9-pin adapter and cable. Note that there are two JTAG ports on the board. Use the one nearest to the reset button as shown in the picture. 

![J-Link debug probe to Arduino](pics/primo-jlink.jpg "Connecting J-Link debug probe to Arduino Primo")

**Note:** If you are using the OpenOCD debugger,  you do not need to attach this connector. 

### Load the Bootloader and the Blinky Application Image
Run the `newt load primo_boot` command to load the bootloader onto the board:

```no-highlight
$ newt load primo_boot
Loading bootloader
$
```
<br>
Run the `newt load primoblinky` command to load the Blinky application image onto the board.

```no-highlight
$ newt  load primoblinky 
Loading app image into slot 1
$
```

You should see the LED on the board blink!

Note: If the LED does not blink, try resetting the board.


<br>

**Note:** If you want to erase the flash and load the image again, you can use JLinkExe to issue an `erase` command.

```
$ JLinkExe -device nRF52 -speed 4000 -if SWD
SEGGER J-Link Commander V5.12c (Compiled Apr 21 2016 16:05:51)
DLL version V5.12c, compiled Apr 21 2016 16:05:45

Connecting to J-Link via USB...O.K.
Firmware: J-Link OB-SAM3U128-V2-NordicSemi compiled Mar 15 2016 18:03:17
Hardware version: V1.00
S/N: 682863966
VTref = 3.300V


Type "connect" to establish a target connection, '?' for help
J-Link>erase
Cortex-M4 identified.
Erasing device (0;?i?)...
Comparing flash   [100%] Done.
Erasing flash     [100%] Done.
Verifying flash   [100%] Done.
J-Link: Flash download: Total time needed: 0.363s (Prepare: 0.093s, Compare: 0.000s, Erase: 0.262s, Program: 0.000s, Verify: 0.000s, Restore: 0.008s)
Erasing done.
J-Link>exit
$
```

<br>


### Conclusion

You have created, setup, compiled, loaded, and ran your first mynewt application
for an Arduino Primo board.

We have more fun tutorials for you to get your hands dirty. Be bold and work on the OS with tutorials on [writing a test suite](unit_test.md) or try enabling additional functionality such as [remote comms](project-target-slinky.md) or [Bluetooth Low Energy](bletiny_project.md) on your current board.

If you see anything missing or want to send us feedback, please do so by signing up for appropriate mailing lists on our [Community Page](../../community.md).

Keep on hacking and blinking!




