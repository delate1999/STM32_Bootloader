# STM32 Bootloader

Simple bootloader implementation for STM32 (Nucleo-F446RE) using STM32 HAL.

## Overview

This repository contains a minimal bootloader and a main application for the
STM32F446RE (Nucleo-F446RE). The bootloader can receive and launch application
firmware, while the Main_application demonstrates a user application built to
run alongside the bootloader.

## Repository layout

- `Bootloader/STM32_Bootloader_Bootloader` — bootloader project (CMake)
- `Main_application/STM32_Bootloader_Main_application` — main application project (CMake)
- `README.md` — this file

Each project contains its own `CMakeLists.txt`, `startup_stm32f446xx.s`, and
linker script `stm32f446retx_flash.ld` configured for the target memory map.

## Features

- Small bootloader implemented with STM32 HAL
- Example main application demonstrating jump from bootloader
- CMake + Ninja build configuration targeting `arm-none-eabi`

## Prerequisites

- GNU Arm Embedded Toolchain (`arm-none-eabi-gcc`, `arm-none-eabi-objcopy`)
- `CMake` (>= 3.15) and `ninja`
- STM32CubeMX (optional, for regenerating initialization code)
- ST-Link / STM32CubeProgrammer or OpenOCD for flashing/debugging

On Windows, ensure `arm-none-eabi-gcc` and `ninja` are in your `PATH`.

## Build (example)

Below are example commands to build the bootloader and the main application.
Adjust paths and toolchain file locations if needed.

Build the Bootloader:

```bash
cd Bootloader/STM32_Bootloader_Bootloader
mkdir -p build && cd build
cmake -G Ninja .. -DCMAKE_TOOLCHAIN_FILE=../cmake/gcc-arm-none-eabi.cmake -DCMAKE_BUILD_TYPE=Debug
ninja
```

Build the Main Application:

```bash
cd Main_application/STM32_Bootloader_Main_application
mkdir -p build && cd build
cmake -G Ninja .. -DCMAKE_TOOLCHAIN_FILE=../cmake/gcc-arm-none-eabi.cmake -DCMAKE_BUILD_TYPE=Debug
ninja
```

Artifacts are produced in the respective `build` directories (typically
`build/Debug` depending on CMake configuration).

## Flashing

Use STM32CubeProgrammer (GUI or CLI) or `st-flash`/OpenOCD to program the binary.
Check the project's linker script (`stm32f446retx_flash.ld`) for expected
flash addresses before flashing. Example using `STM32_Programmer_CLI`:

```bash
# Example: flash the bootloader (adjust file path and address as needed)
STM32_Programmer_CLI -c port=SWD -w build/Debug/STM32_Bootloader_Bootloader.bin 0x08000000
```

Or using `st-flash` (binary) with default ST-Link:

```bash
st-flash --reset write build/Debug/STM32_Bootloader_Bootloader.bin 0x08000000
```

Note: The correct flash address for the main application is defined by the
linker script and by how you partition flash between bootloader and application.

## Linker & Vector Table Notes

- The bootloader typically occupies the beginning of flash (e.g. `0x08000000`).
- The application should be linked to run from its own start address and must
	have a valid vector table and stack pointer placed at that address.
- Before jumping to the application, the bootloader must relocate the vector
	table (`SCB->VTOR`) to the application's vector table and set the stack
	pointer from the application's initial stack value.

Refer to the code in `Bootloader/STM32_Bootloader_Bootloader/Core/Src` and the
linker script to confirm addresses and alignment.

## Development & Debugging

- Use your debugger (ST-Link) connected to the Nucleo board.
- Load the bootloader first, then load the application to its intended flash
	offset, or use the bootloader's firmware update method if implemented.

## License

MIT

