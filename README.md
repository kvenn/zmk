# Zephyr™ Mechanical Keyboard (ZMK) Firmware

[![Discord](https://img.shields.io/discord/719497620560543766)](https://zmk.dev/community/discord/invite)
[![Build](https://github.com/zmkfirmware/zmk/workflows/Build/badge.svg)](https://github.com/zmkfirmware/zmk/actions)
[![Contributor Covenant](https://img.shields.io/badge/Contributor%20Covenant-v2.0%20adopted-ff69b4.svg)](CODE_OF_CONDUCT.md)

[ZMK Firmware](https://zmk.dev/) is an open source (MIT) keyboard firmware built on the [Zephyr™ Project](https://www.zephyrproject.org/) Real Time Operating System (RTOS). ZMK's goal is to provide a modern, wireless, and powerful firmware free of licensing issues.

Check out the website to learn more: https://zmk.dev/

You can also come join our [ZMK Discord Server](https://zmk.dev/community/discord/invite)

To review features, check out the [feature overview](https://zmk.dev/docs/). ZMK is under active development, and new features are listed with the [enhancement label](https://github.com/zmkfirmware/zmk/issues?q=is%3Aissue+is%3Aopen+label%3Aenhancement) in GitHub. Please feel free to add 👍 to the issue description of any requests to upvote the feature.


# Setup
* Download docker for mac
* download remote containers
* Open directory in container
* run west update (creates volumes)
* close vscode (kills container)
* delete the container
* delete zmk-config (docker volume) - docker volume rm zmk-config
* create own zmk config volume pointed at correct board (inside /app/boards/sheild/<boardname>)
docker volume create --driver local -o o=bind -o type=none -o \
    device="/Users/kylevenn/Development/zmk/app/boards/shields/sp64" zmk-config
* Open vscode again and reopen in container
* Then run build
west build -b nice_nano -- -DSHIELD=sp64_left -DZMK_CONFIG="/workspaces/zmk-config/config"

If this works, it'll output the file to `/workspaces/zmk/app/build/zephyr/zmk.uf2`



## Setup
This repo contains the keymap and conf file for your keyboard inside `zmk/app/boards/shields/<your_keyboard>`.

Fork this repo to begin keymap modifications.

## Setup west
`west` is the pip extension for building your uf2 file to be flashed to the keyboard

### Installation
[Docs](https://zmk.dev/docs/development/setup/#prerequisites)
* Run all these
```bash
asdf plugin-add python
asdf install # asdf install python 3.10.4
brew install cmake ninja ccache dtc git wget dfu-util
asdf reshim python
pip install --user -U west
brew install --cask gcc-arm-embedded
```
* Add the following to your zshrc
```
# zephyr build vars
export ZEPHYR_TOOLCHAIN_VARIANT=gnuarmemb
export GNUARMEMB_TOOLCHAIN_PATH=$(brew --prefix)

# pip
export PATH="$HOME/.local/bin:$PATH"
``` 
* Then run `asdf reshim python` one last time
* Confirm it worked with `python -m pip show west`

### Init west
[Docs](https://zmk.dev/docs/development/setup/#initialize-west)

```bash
west init -l app/
west update
west zephyr-export
pip install --user -r zephyr/scripts/requirements-base.txt
```

### Install zephyr sdk (Not necessary??)
[Docs](https://docs.zephyrproject.org/latest/getting_started/index.html#install-a-toolchain)
* NOTE: Read docs if not on m1 mac
```bash
cd ~
wget https://github.com/zephyrproject-rtos/sdk-ng/releases/download/v0.14.0/zephyr-sdk-0.14.0_macos-aarch64.tar.gz
wget -O - https://github.com/zephyrproject-rtos/sdk-ng/releases/download/v0.14.0/sha256.sum | shasum --check --ignore-missing
tar xvf zephyr-sdk-0.14.0_macos-aarch64.tar.gz
rm zephyr-sdk-0.14.0_macos-aarch64.tar.gz
# Only need to do this once
./zephyr-sdk-0.14.0/setup.sh
```

## Modifying keymap
* 

## Building
[Docs](https://zmk.dev/docs/development/build-flash/#building-for-split-keyboards)
```bash
cd app
## nuke build folder
rm -r build/
west build -d build/sp64/left -b nice_nano -- -DSHIELD=sp64_left
west build -d build/sp64/right -b nice_nano -- -DSHIELD=sp64_right
```

This will output .uf2 files -> /build/sp64/left/zephyr/zmk.uf2

## Flashing uf2 files
[Docs](https://zmk.dev/docs/user-setup/#flashing-uf2-files)
[Docs](https://zmk.dev/docs/development/build-flash/#flashing)
[Docs](https://zmk.dev/docs/customization#flashing-your-changes)

> For split keyboards, only the central (left) side will need to be reflashed if you are just updating your keymap.

* First put your board into bootloader mode by double clicking the reset button (either on the MCU board itself, or the one that is part of your keyboard). 
* The controller should appear in your OS as a new USB storage device.
* Copy the correct UF2 file (e.g. left or right if working on a split), and paste it onto the root of that USB mass storage device. 
   * Should be called NICENANO
* Once the flash is complete, the controller should automatically restart, and load your newly flashed firmware.
   * It will unmount when it finishes copying
   * Ignore warning that you need to eject before unmounting.  It will always do that.

## Connecting split halves
https://zmk.dev/docs/user-setup/#connecting-split-keyboard-halves
For split keyboards, after flashing each half individually you can connect them together by resetting them at the same time. Within a few seconds of resetting, both halves should automatically connect to each other.