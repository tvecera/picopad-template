# Picopad Program Template

This repository provides a simple program template for creating software for the Pajenicko Picopad handheld. This
project is based on the Picopad SDK and is set up for development and building using CLion and CMAKE.

### Links

- [CLion](https://www.jetbrains.com/clion/download/)
- [CMake](https://cmake.org/download/)
- Picopad from [Pajenicko e-shop](https://www.pajenicko.cz/)
- Picopad SDK (https://github.com/Pajenicko/Picopad)

### Setting Up the Project

1. Clone this repository to your local machine.
2. Open the project directory in CLion.
3. CLion automatically picks up CMakeLists.txt and configures the project accordingly.
4. You need to change the TOOLCHAIN_PREFIX in the CMake file to match your specific system setup.

### Building and Running the Project

1. To build the project, go to `Build -> Build Project` in the CLion menu.
2. The final step is to upload the generated uf2 file to the SD card or directly to the handheld device.

**Important:** This project only contains macOS builds of some SDK tools. 

