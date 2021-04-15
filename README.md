# Espressif IoT Development Framework


## Sean Mollet's M1 build:

### To install and use my builds:

1. Replace your tools.json and idf_tools.py with versions that know what macos-arm64 is.

```
cd ~/esp/esp-idf/tools
wget https://github.com/SeanMollet/esp-idf/raw/esp-2020r3-aarch64/tools/tools.json
wget https://github.com/SeanMollet/esp-idf/raw/esp-2020r3-aarch64/tools/idf_tools.py
```
2. Run esp install.sh again

```
~/esp/esp-idf/install.sh
```

3. Enjoy your crazy fast build performance.


Note: I did not go to the trouble of building openocd or the ulp binutils for M1. They're both low cpu utilities and work just fine with rosetta2. The gcc toolchain though was worth the effort. It's MUCH faster running native.


### For VSCode users, download and install the following extension:

https://github.com/SeanMollet/vscode-esp-idf-extension/releases/tag/macos_arm64


### Duplicating my build:
Follow espressif's instructions for installing homebrew items.
Then, clone my repo of "fixes" for the currently broken arm binutils:

git clone https://github.com/SeanMollet/aarch64_binutils_fixes
Copy the shell scripts from there to /opt/homebrew/opt/binutils/bin
Replacing the binaries. You can back them up first if you like, although they don't work anyway and the cross toolchain will build its own.


Follow espressif's instructions to set up a case sensitive disk image on which to perform the building.


Clone my repo and checkout the proper branch:

```
git clone https://github.com/SeanMollet/crosstool-NG
cd crosstool-NG
git checkout esp-2020r3-aarch64
```

Clone the submodule:
```
git submodule update --init --recursive
```

Set the path so it can find the "binutils"
```
export PATH=/opt/homebrew/opt/binutils/bin:$PATH
```

Needed for building cross-tool
```
export LDFLAGS="-L/opt/homebrew/opt/ncurses/lib -L/opt/homebrew/opt/gettext/lib"
export CPPFLAGS="-I/opt/homebrew/opt/ncurses/include -I/opt/homebrew/opt/gettext/include"
export PKG_CONFIG_PATH="/opt/homebrew/opt/ncurses/lib/pkgconfig"
```

Build crosstool-ng:
```
./bootstrap
./configure --enable-local
make
```

Configure for xtensa
```
./ct-ng xtensa-esp32-elf
```

Edit .config and change the following setting:
```
CT_GDB_CROSS_EXTRA_CONFIG_ARRAY="--disable-tui"
```

Build it.. takes about 15 minutes
```
./ct-ng build
```


* [中文版](./README_CN.md)

ESP-IDF is the official development framework for the **ESP32** and **ESP32-S** Series SoCs provided for Windows, Linux and macOS.

# Developing With ESP-IDF

## Setting Up ESP-IDF

See setup guides for detailed instructions to set up the ESP-IDF:

| Chip | Getting Started Guides for ESP-IDF |
|:----:|:----|
| <img src="docs/_static/chip-esp32.svg" height="85" alt="ESP32"> |  <ul><li>[stable](https://docs.espressif.com/projects/esp-idf/en/stable/get-started/) version</li><li>[latest (master branch)](https://docs.espressif.com/projects/esp-idf/en/latest/get-started/) version</li></ul> |
| <img src="docs/_static/chip-esp32-s2.svg" height="100" alt="ESP32-S2"> | <ul><li>[latest (master branch)](https://docs.espressif.com/projects/esp-idf/en/latest/esp32s2/get-started/) version</li></ul> |

**Note:** Each ESP-IDF release has its own documentation. Please see Section [Versions](https://docs.espressif.com/projects/esp-idf/en/latest/esp32/versions.html) how to find documentation and how to checkout specific release of ESP-IDF.

### Non-GitHub forks

ESP-IDF uses relative locations as its submodules URLs ([.gitmodules](.gitmodules)). So they link to GitHub.
If ESP-IDF is forked to a Git repository which is not on GitHub, you will need to run the script
[tools/set-submodules-to-github.sh](tools/set-submodules-to-github.sh) after git clone.
The script sets absolute URLs for all submodules, allowing `git submodule update --init --recursive` to complete.
If cloning ESP-IDF from GitHub, this step is not needed.

## Finding a Project

As well as the [esp-idf-template](https://github.com/espressif/esp-idf-template) project mentioned in Getting Started, ESP-IDF comes with some example projects in the [examples](examples) directory.

Once you've found the project you want to work with, change to its directory and you can configure and build it.

To start your own project based on an example, copy the example project directory outside of the ESP-IDF directory.

# Quick Reference

See the Getting Started guide links above for a detailed setup guide. This is a quick reference for common commands when working with ESP-IDF projects:

## Setup Build Environment

(See the Getting Started guide listed above for a full list of required steps with more details.)

* Install host build dependencies mentioned in the Getting Started guide.
* Run the install script to set up the build environment. The options include `install.bat` or `install.ps1` for Windows, and `install.sh` or `install.fish` for Unix shells.
* Run the export script on Windows (`export.bat`) or source it on Unix (`source export.sh`) in every shell environment before using ESP-IDF.

## Configuring the Project

* `idf.py set-target <chip_name>` sets the target of the project to `<chip_name>`. Run `idf.py set-target` without any arguments to see a list of supported targets.
* `idf.py menuconfig` opens a text-based configuration menu where you can configure the project.

## Compiling the Project

`idf.py build`

... will compile app, bootloader and generate a partition table based on the config.

## Flashing the Project

When the build finishes, it will print a command line to use esptool.py to flash the chip. However you can also do this automatically by running:

`idf.py -p PORT flash`

Replace PORT with the name of your serial port (like `COM3` on Windows, `/dev/ttyUSB0` on Linux, or `/dev/cu.usbserial-X` on MacOS. If the `-p` option is left out, `idf.py flash` will try to flash the first available serial port.

This will flash the entire project (app, bootloader and partition table) to a new chip. The settings for serial port flashing can be configured with `idf.py menuconfig`.

You don't need to run `idf.py build` before running `idf.py flash`, `idf.py flash` will automatically rebuild anything which needs it.

## Viewing Serial Output

The `idf.py monitor` target uses the [idf_monitor tool](https://docs.espressif.com/projects/esp-idf/en/latest/get-started/idf-monitor.html) to display serial output from ESP32 or ESP32-S Series SoCs. idf_monitor also has a range of features to decode crash output and interact with the device. [Check the documentation page for details](https://docs.espressif.com/projects/esp-idf/en/latest/get-started/idf-monitor.html).

Exit the monitor by typing Ctrl-].

To build, flash and monitor output in one pass, you can run:

`idf.py flash monitor`

## Compiling & Flashing Only the App

After the initial flash, you may just want to build and flash just your app, not the bootloader and partition table:

* `idf.py app` - build just the app.
* `idf.py app-flash` - flash just the app.

`idf.py app-flash` will automatically rebuild the app if any source files have changed.

(In normal development there's no downside to reflashing the bootloader and partition table each time, if they haven't changed.)

## Erasing Flash

The `idf.py flash` target does not erase the entire flash contents. However it is sometimes useful to set the device back to a totally erased state, particularly when making partition table changes or OTA app updates. To erase the entire flash, run `idf.py erase_flash`.

This can be combined with other targets, ie `idf.py -p PORT erase_flash flash` will erase everything and then re-flash the new app, bootloader and partition table.

# Resources

* Documentation for the latest version: https://docs.espressif.com/projects/esp-idf/. This documentation is built from the [docs directory](docs) of this repository.

* The [esp32.com forum](https://esp32.com/) is a place to ask questions and find community resources.

* [Check the Issues section on github](https://github.com/espressif/esp-idf/issues) if you find a bug or have a feature request. Please check existing Issues before opening a new one.

* If you're interested in contributing to ESP-IDF, please check the [Contributions Guide](https://docs.espressif.com/projects/esp-idf/en/latest/contribute/index.html).


