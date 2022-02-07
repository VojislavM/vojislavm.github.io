---
layout: post
title: "nRF52 development setup on Linux"
date: 2022-02-07 12:00:00 -0500
---

Setup development environment for nRF5 SDK on Linux host machine 

This blog post was originaly posted on [Irnas blog](https://www.irnas.eu/nrf52-development-setup-on-linux/) writen by me in 2020.


Designing IoT solutions with low power consumption, various types of connectivity (BLE, LoRa etc.), robust casing, and ensuring satisfactory user experience is a wholesome kind of challenge. When starting a new project it is very important to keep all those ends in mind. A big integrating part of every project here at IRNAS is the embedded software. At IRNAS we like to establish tools and methods for ourselves that make our job easier and less prone to errors right from the start. In this blog, we will demonstrate a simple example of Bluetooth LE beacon and how we go about verifying our design decisions in the process.

Embedded software development for IoT devices is as vividly evolving as the industry itself. Often we see that chip vendors provide SDK (Software Development Kit) for their microcontrollers allowing users to choose the desired toolchain. Moreover, continuous integration tools are starting to enter the field of embedded engineers, as well as new test equipment that provides interfaces in some high-level language like Python. All of these are phenomena we are all familiar with and relate them with software development itself. What all this is showing is that it is now time to treat embedded software more as “regular” software and implement all the good practices and knowledge that already exist. Test-driven development, Continuous Integration, Continuous delivery, automated testing, are just some of those practices.

Let us demonstrate a typical workflow on an nRF52, a chip by Nordic Semi, which we use on a regular basis. We will explain how to set up the environment using the GCC compiler on Linux mashing, beginning with the installation of the toolchain and setting up the development environment for the desired microcontroller.

This setup will be done on the Linux machine (Ubuntu 18.04 LTS) and with the nRF52832-DK development board.

## First, Install the Compiler

As mentioned earlier for this task we are using a GCC compiler. It is first necessary to install the correct version for ARM devices:

1. Download arm-gcc version 7.3.1 (gcc-arm-none-eabi-7-2018-q2-update) from ARM website: https://developer.arm.com/tools-and-software/open-source-software/developer-tools/gnu-toolchain/gnu-rm/downloads
This can be done from linux terminal with command:

    ```
    $ wget https://developer.arm.com/-/media/Files/downloads/gnu-rm/7-2018q2/gcc-arm-none-eabi-7-2018-q2-update-linux.tar.bz2
    ```

2. Install the GNU toolchain for ARM: After download, extract it in a folder of your preference **INSTALL_PATH** and then add this path to your **OS PATH** environment variable. Add this line to `.bashrc` file in `/home/$USER` directory

    ```
    $ export PATH=$PATH:/gcc-arm-none-eabi-7-2018-q2-update/bin
    ```

    and then execute:

    ```
    $ source ~/.bashrc
    ```

3. Verify that the path is set correctly, type the following in your terminal and check the output:

    ```
    $ arm-none-eabi-gcc --version
    arm-none-eabi-gcc (GNU Tools for ARM Embedded Processors) 5.4.1 20160919 (release) [ARM/embedded-5-branch revision 240496]
    Copyright (C) 2015 Free Software Foundation, Inc.
    This is free software; see the source for copying conditions. There is NO
    warranty; not even for MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.
    ```

4. Install make (optional): this need to be done if make is not yet present in the system

    ```
    $ sudo apt-get install build-essential checkinstall
    ```

## Now, Clone the Nordic nRF5 SDK

It is necessary to clone Nordic Semi nRF5 SDK from their website. There are a lot of release versions of this SDK in which case our suggestion is to use the latest one and stick with it. We will use version v15.3 from the download link and unpack it in the preferred location.

Next, edit the file named `/components/toolchain/gcc/Makefile.posix` and change the `GNU_INSTALL_ROOT` to point to the location of your GNU Arm Embedded Toolchain installation:

```
GNU_INSTALL_ROOT ?= /gcc-arm-none-eabi-7-2018-q2-update/bin/
GNU_VERSION ?= 7.3.1
GNU_PREFIX ?= arm-none-eabi
```

## Next, Install an IDE or text editor of choice (if you haven’t got one installed already)

Everybody has their preferred IDE and text editor, and we are nobody to get in your way regarding which one you use. One we chose to work with is the Visual Studio Code. It is lightweight and configurable so we can change everything we need to get things done fast and efficient with only a handful of commands. Assuming VS Code is your choice the installation can be done with a snap:

```
$ sudo snap install --classic code # or code-insiders
```

A useful feature of VS Code is the set of available extensions it has to offer. For this example we need two of them:

* CPP TOOLS
* CORTEX DEBUG

Lastly, one optional thing to install is Java run time environment. This is used to get a graphical interface with checkboxes for CMSIS configuration:

```
$ sudo apt-get install default-jre
```

## Add environmental variables to system

Setting the environmental variables in your Linux system can be done in several ways. One way is to open `.bashrc` filed in your favourite text editor and at the end of the file just add:

```
# nRF SDK location
export nRF_SDK=/nRF5_SDK_15.3.0_59ac345
# ARM GCC location
export GNU_GCC=/gcc-arm-none-eabi-7-2018-q2-update 
```

where `<INSTALL_PATH>` needs to be the path to your nRF5 SDK and GCC-ARM. After that save the file and restart the terminal.

## Setup the workplace

We will use ble_app_beacon example, one of many in Nordic Semi nRF5 SDK to demonstrate the build debug and power measurement procedures. So after opening the VS Code, we should add the folder to the workplace. So go to: `File -> Add Folder to Workplace...` and add example application from SDK: `nRF5_SDK_15.3.0_59ac345/examples/ble_peripheral/ble_app_beacon`. Save the workplace file with: `File -> Save Workplace As...`

## C edit configuration

To be able to see the C/CPP properties file in folder structure press `Ctrl + Shift + P` and search for **C/C++: Edit Configuration (JSON)**. After that, a new file will appear in the folder structure `./.vscode/c_cpp_properites.json`. Here we will change all the compile flags and other necessary paths. The file should look like this:

```
{
    "configurations": [
        {
            "name": "nRF52832 DK",
            "includePath": [
                "${workspaceFolder}/**",
                "${env:GNU_GCC}/arm-none-eabi/include",
                "${env:nRF_SDK}/modules/**",
                "${env:nRF_SDK}/components/**"
            ],
            "defines": [
                "BOARD_PCA10040",
                "CONFIG_GPIO_AS_PINRESET",
                "INITIALIZE_USER_SECTIONS",
                "FLOAT_ABI_HARD",
                "NRF52",
                "NRF52832_XXAA",
                "NRF_SD_BLE_API_VERSION=6",
                "S132",
                "SOFTDEVICE_PRESENT",
                "SWI_DISABLE0"
            ],
            "compilerPath": "${env:GNU_GCC}/bin/arm-none-eabi-gcc",
            "cStandard": "c11",
            "cppStandard": "c++17",
            "intelliSenseMode": "clang-x64"
        }
    ],
    "version": 4
}
```

## Task setup

Now it is time to add all necessary tasks to the VS Code to know how to build, clean or flash the project. To create tasks file go to `Terminal -> Configure Tasks...` then select Create `tasks.json` file from template and select Other. A `tasks.json` file will be created and it needs to be configured with the following tasks:

* `build` – build the project
* `clean` – clean the project, delete the build folder
* `flash` – flash the application code
* `flash_softdevice` – flash the softdevice, need to be done before flashing the application firmware
* `sdk_config` – configure CMSIS for the application
* `serial` – open the serial port to the device
* `erase` – erase the application from the target

```
{
    // See https://go.microsoft.com/fwlink/?LinkId=733558
    // for the documentation about the tasks.json format
    "version": "2.0.0",
    "tasks": [
        {
            "label": "build",
            "type": "shell",
            "command": "make",
            "options": {
                "cwd": "${workspaceFolder}/pca10040/s132/armgcc"
            },
            "problemMatcher": [],
            "group": {
                "kind": "build",
                "isDefault": true
            }
        },
        {
            "label": "clean",
            "type": "shell",
            "command": "make clean",
            "options": {
                "cwd": "${workspaceFolder}/pca10040/s132/armgcc"
            },
            "problemMatcher": []
        },
        {
            "label": "flash",
            "type": "shell",
            "command": "make flash",
            "options": {
                "cwd": "${workspaceFolder}/pca10040/s132/armgcc"
            },
            "group": "build",
            "problemMatcher": []
        },
        {
            "label": "flash_softdevice",
            "type": "shell",
            "command": "make flash_softdevice",
            "options": {
                "cwd": "${workspaceFolder}/pca10040/s132/armgcc"
            },
            "problemMatcher": []
        },
        {
            "label": "sdk_config",
            "type": "shell",
            "command": "make sdk_config",
            "options": {
                "cwd": "${workspaceFolder}/pca10040/s132/armgcc"
            },
            "problemMatcher": []
        },
        {
            "label": "serial",
            "type": "shell",
            "command": "screen /dev/ttyACM0 115200",
            "problemMatcher": []
        },
        {
            "label": "erase",
            "type": "shell",
            "command": "make erase",
            "options": {
                "cwd": "${workspaceFolder}/pca10056/s140/armgcc"
            },
            "problemMatcher": []
        }
    ]
}
```

## Finally, Configure debug

Target debug is a feature that most of the professional IDEs have, and it is absolutely crucial when developing advanced firmware applications. With Cortex-debug extension this is also possible in VS code. First, we just need to create debug configuration `launch.json` and configure the file:

```
{
    // Use IntelliSense to learn about possible attributes.
    // Hover to view descriptions of existing attributes.
    // For more information, visit: https://go.microsoft.com/fwlink/?linkid=830387
    "version": "0.2.0",
    "configurations": [
        {
            "name": "Cortex Debug",
            "cwd": "${workspaceRoot}",
            "executable": "${workspaceFolder}/pca10040/s132/armgcc/_build/nrf52832_xxaa.out",
            "request": "launch",
            "type": "cortex-debug",
            "servertype": "jlink",
            "device": "nrf52",
            "interface": "swd",
            "ipAddress": null,
            "serialNumber": null,
            "armToolchainPath": "${env:GNU_GCC}/bin/"
        }
    ]
}
```

To start debugging go to `Run -> Start Debugging` and the debugging session will start.

## Conclusion

After this setup, we can build debug and flash our application to the target device using nRF52832 and see it in action. While we only scratched the surface of what VS Code is capable of, hopefully, this post at least helped you get it all up and running. In the coming posts, we will focus on adding additional tools to the build process and also show you how we develop firmware applications with other exciting test tools.


For more insight, have a scroll through the following links:
* https://www.novelbits.io/nrf52-visual-studio-code/
* https://gustavovelascoh.wordpress.com/2017/01/23/starting-development-with-nordic-nrf5x-and-gcc-on-linux/
* https://electronut.in/visual-studio-code-nrf52-dev/









