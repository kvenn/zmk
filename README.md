# Zephyr™ Mechanical Keyboard (ZMK) Firmware

[![Discord](https://img.shields.io/discord/719497620560543766)](https://zmk.dev/community/discord/invite)
[![Build](https://github.com/zmkfirmware/zmk/workflows/Build/badge.svg)](https://github.com/zmkfirmware/zmk/actions)

[ZMK Firmware](https://zmk.dev/) is an open source (MIT) keyboard firmware built on the [Zephyr™ Project](https://www.zephyrproject.org/) Real Time Operating System (RTOS). ZMK's goal is to provide a modern, wireless, and powerful firmware free of licensing issues.

Check out the website to learn more: https://zmk.dev/

You can also come join our [ZMK Discord Server](https://zmk.dev/community/discord/invite)

To review features, check out the [feature overview](https://zmk.dev/docs/). ZMK is under active development, and new features are listed with the [enhancement label](https://github.com/zmkfirmware/zmk/issues?q=is%3Aissue+is%3Aopen+label%3Aenhancement) in GitHub. Please feel free to add 👍 to the issue description of any requests to upvote the feature.


## Setup
This repo contains the keymap and conf file for your keyboard inside `zmk/app/boards/shields/<your_keyboard>`.

Fork this repo to begin keymap modifications.

### Steps
You're going to run the build of this repo in a Docker instance that's configured with python, west, and a bunch of other gnarly stuff.
You're going to create a volume called `zmk-config` that this Docker knows to look for that will have a link to the board you want build.

1. Install [Docker for Mac](https://docs.docker.com/desktop/mac/install/)
   - Wait for it to be fully started (icon in system tray will stop animating)
1. Open this repo in VSCode
1. Download [Remote Containers VSCode extension](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-containers)
1. Select "Open directory in container" (from command palette or from VSCode popup post install)
1. In the VSCode terminal, run `west update` (this creates volumes in Docker)
1. Close VSCode (this stops the container) 
1. Open Docker for Mac and delete the container (trash icon)
1. Delete the zmk-config (docker volume) 
   - `docker volume rm zmk-config` or click trashcan in Docker for Mac
1. Create your own zmk config volume pointed at the correct board (inside `/app/boards/sheild/<your_keyboard>`)
   - Replace the url below with the location of your board
   ```bash
   docker volume create --driver local -o o=bind -o type=none -o \
      device="/Users/kylevenn/Development/zmk/app/boards/shields/sp64" zmk-config 
   ```
1. Open VSCode again and reopen the directory in a container (command palette)
1. Install the ZMK extension for VSCode (gets syntax highlighting)

## Editing keymap
[Full list of codes](https://zmk.dev/docs/codes/)

* Open up the keymap for your board located at `zmk/app/boards/sheild/<your_keyboard>/<keyboard>.keymap`
* Make whatever modifications you want 

### Cheatsheet
* lt = layer tap <long press layer to activate> <key press action>
* kp = key press
* mo = momentary layer (only activates layers while pressed)

### Layers
Layers are represented by numbers. Here is a standard config. So if you see &mo 1, that means pressing that key toggles the lower layer.
```
#define DEFAULT 0
#define LOWER   1
#define RAISE   2
```

## Building and deploying
You'll generate a .uf2 file that you'll flash to the keyboard. You only need to flash left for a split that's already set up.

Inside the VSCode terminal, build the left side with
`west build -b nice_nano -- -DSHIELD=sp64_left -DZMK_CONFIG="/workspaces/zmk-config/config"`
And right side with:
`west build -b nice_nano -- -DSHIELD=sp64_right -DZMK_CONFIG="/workspaces/zmk-config/config"`

If this works, it'll output the file to `/workspaces/zmk/app/build/zephyr/zmk.uf2`

## Flashing 
[Docs](https://zmk.dev/docs/user-setup/#flashing-uf2-files)
[Docs](https://zmk.dev/docs/development/build-flash/#flashing)
[Docs](https://zmk.dev/docs/customization#flashing-your-changes)

> For split keyboards, only the central (left) side will need to be reflashed if you are just updating your keymap.

* First put your board into bootloader mode by double clicking the reset button (either on the MCU board itself, or the one that is part of your keyboard). 
* The controller should appear in your OS as a new USB storage device.
* Copy the correct UF2 file (e.g. left or right if working on a split), and paste it onto the root of that USB mass storage device. 
   * Should be called something like NICENANO
* Once the flash is complete, the controller should automatically restart, and load your newly flashed firmware.
   * It will unmount when it finishes copying
   * Ignore warning that you need to eject before unmounting.  It will always do that.

## Connecting split halves
https://zmk.dev/docs/user-setup/#connecting-split-keyboard-halves
For split keyboards, after flashing each half individually you can connect them together by resetting them at the same time. Within a few seconds of resetting, both halves should automatically connect to each other.

-----------

# Manual setup (if you didn't do docker above)
Warning, this doesn't work on M1 Macs

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
