# Official Splitkb.com Halcyon ZMK config

This is the Splitkb Halcyon ZMK config repository. It allows for an external set of ZMK keymaps with or without Halcyon modules to be defined and compiled. If you want to add support to your existing keyboard, please look at the [porting guide](PORTING.md).

##### Table of Contents  
[Supported keyboards](#supported-keyboards)  
[Initial Setup](#initial-setup--prerequisites)  
[Configuring Build targets](#how-to-configure-your-build-targets)  
[Build target examples](#examples)  
[Local build](#local-builds)  
[Default config options](#config-options)  
[Customizing module behavior](#customizing-module-behavior) 

## Supported Keyboards

Supported boards:

| Board name | Board variable |
| :--- | :--- |
| Halcyon Wireless controller | `halcyon_wireless//zmk` |
| Halcyon Dongle | `halcyon_dongle//zmk` |
| Halcyon Wired controller* | `halcyon_wired//zmk` |

*While the wired controller is supported, full testing has not been done. Some configurations may not work as expected.


Supported keyboard shields:

| Keyboard name | Shield variable |
| :--- | :--- |
| Halcyon Kyria (rev4) | `halcyon_kyria` |
| Halcyon Elora (rev2) | `halcyon_elora` |
| Halcyon Corne (rev2) | `halcyon_corne` |
| Halcyon Ferris (rev1) | `halcyon_ferris` |
| Halcyon Lily58 (rev2) | `halcyon_lily58` |
| Aurora Sweep (rev1)* | `splitkb_aurora_sweep` |
| Aurora Lily58 (rev1)* | `splitkb_aurora_lily58` |
| Aurora Corne (rev1)* | `splitkb_aurora_corne` |
| Aurora Helix (rev1)* | `splitkb_aurora_helix` |
| Aurora Sofle v2 (rev1)* | `splitkb_aurora_sofle` |
| Kyria (rev3)* | `kyria_rev3` |

*The files for the Aurora and Kyria rev3 shields can be found in the [Aurora branch](https://github.com/splitkb/zmk-halcyon-config/tree/aurora). If you want to customize your keyboard behavior and keep the Halcyon module functionality, make sure to use that branch. You can delete any files for the keyboards you are not using.


Supported converters:

| Converter name | Shield variable |
| :--- | :--- |
| Halcyon to Promicro adapter | `halcyon_to_promicro` |


Supported battery boards:
| Battery board name | Shield variable |
| :--- | :--- |
| LiPo Battery board | `mod_battery_lipo` |
| Coincell Battery board | `mod_battery_coincell` |


Supported module shields:

| Module name | Shield variable |
| :--- | :--- |
| [Halcyon TFT LCD Display Module](https://splitkb.com/products/halcyon-tft-lcd-display-module) | `mod_display_tft` |
| [Halcyon Rotary Encoder Module Revision 2](https://splitkb.com/products/halcyon-rotary-encoder-module) | `mod_encoder_left` and `mod_encoder_right` |
| [Halcyon Cirque Touchpad Module](https://splitkb.com/products/halcyon-cirque-touchpad-module) | `mod_cirque_central_hw`, `mod_cirque_central`, `mod_cirque_hw_left` and `mod_cirque_hw_right` |
| Halcyon Epaper Display Module | `mod_display_epaper_mountain`, `mod_display_epaper_forest` and `mod_display_epaper_cityscape` |
| Halcyon MIP Display Module (TBA) | `mod_display_mip` |

Additionally, the nice!view is also supported when using the Halcyon to Promicro adapter by adding the following shields `halcyon_nice_view_adapter nice_view` to your build. OLED can also be enabled by setting the marked config options in the keyboards `.conf` file.

## Initial Setup & Prerequisites

Create a fork of this repository using the `Use this template` button and select `Create a new repository`. 

Follow the steps from the [ZMK documentation](https://zmk.dev/docs/user-setup). When you arrive at [Config Repo Setup](https://zmk.dev/docs/user-setup#config-repo-setup) step, follow the steps where you already have a ZMK config repo on github where you point to the newly cloned fork.

> The aurora branch will have already setup the config. You can clear the `build.yaml` file and follow the instructions and examples below to set up your specific build.

# How to Configure Your Build Targets

By default the ZMK CLI will create build targets for the left and right half. The `build.yaml` file will need to be updated to support the various modules as shown above.

The following section explains how to correctly compose the `shield` values.

## 1. Dongle Builds

Dongles use a fixed composition:

```yaml
shield: <keyboard>_dongle mod_cirque_central
```

* `<keyboard>_dongle` is your keyboard’s dongle shield
* `mod_cirque_central` is required when one of the halfs includes a cirque trackpad

No other modules are added to dongles.

## 2. Keyboard Shields

A keyboard build generally looks like this:

```yaml
shield: <converter_shield> <keyboard>_<side> <battery> <module>
cmake-args: -DCONFIG_ZMK_SPLIT_ROLE_CENTRAL=n
```

* `<converter_shield>` = optional, only needed when using the Halcyon to Promicro adapter
* `<keyboard>` = one of the supported keyboards
* `<side>` = `left` or `right`
* `<battery>` = your selected battery module
* `<module>` = your selected Halcyon module, which will be explained in the next section
* The `cmake-args` is only needed for the left side or on dongle builds 

At the bottom there are some examples.

## 3. Adding Modules

### Encoder Modules

Match the module to the correct side:

* Left encoder:

  ```yaml
  shield: <keyboard>_left <battery> mod_encoder_left
  ```

* Right encoder:

  ```yaml
  shield: <keyboard>_right <battery> mod_encoder_right
  ```


---


### Pointing Modules

#### Dongle builds

Match the module to the correct side:

* Left-side pointing:

  ```yaml
  shield: <keyboard>_left <battery> mod_cirque_hw_left
  ```

* Right-side pointing:

  ```yaml
  shield: <keyboard>_right <battery> mod_cirque_hw_right
  ```

#### Non-dongle builds

* Left central pointing device:

  ```yaml
  shield: <keyboard>_left <battery> mod_cirque_central_hw
  ```

* Right-side pointing:

  ```yaml
  shield: <keyboard>_right <battery> mod_cirque_hw_right
  ```

> When NOT using a dongle but you are using a Cirque trackpad on the right half, the left side will need `mod_cirque_central` appended to the build.

---

### Display Modules

For the display module you can choose between three images: `forest`, `mountain` and `cityscape`.

* Left half with image:

  ```yaml
  shield: <keyboard>_left <battery> mod_display_epaper_<image>
  ```

* Right half with image:

  ```yaml
  shield: <keyboard>_right <battery> mod_display_epaper_<image>
  ```

### nice!view on the Halcyon to Promicro adapter

* Left half:

  ```yaml
  shield: halcyon_to_promicro <keyboard>_left <battery> halcyon_nice_view_adapter nice_view
  ```

* Right half:

  ```yaml
  shield: halcyon_to_promicro <keyboard>_right <battery> halcyon_nice_view_adapter nice_view
  ```

## 4. Studio Builds

If building a Studio build, append:

```yaml
cmake-args: -DCONFIG_ZMK_STUDIO=y
snippet: studio-rpc-usb-uart
```

To the central half. This will be either the dongle or the left side when **NOT** using a dongle.

## Examples:

Halcyon Kyria with a dongle with ZMK Studio enabled, a epaper display with the forest image and coincell battery on the left half, and a rotary encoder and coincell battery on the right half.

```
---
include:
  - board: halcyon_dongle//zmk
    shield: halcyon_kyria_dongle
    cmake-args: -DCONFIG_ZMK_STUDIO=y
    snippet: studio-rpc-usb-uart
  - board: halcyon_wireless//zmk
    shield: halcyon_kyria_left mod_battery_coincell mod_display_epaper_forest
    cmake-args: -DCONFIG_ZMK_SPLIT_ROLE_CENTRAL=n
  - board: halcyon_wireless//zmk
    shield: halcyon_kyria_right mod_battery_coincell mod_encoder_right
```

Aurora corne without dongle, nice!view and LiPo battery on the left half, and a a cirque touchpad and LiPo battery on the right half.

```
---
include:
  - board: halcyon_wireless//zmk
    shield: halcyon_to_promicro splitkb_aurora_corne_left mod_battery_lipo mod_cirque_central halcyon_nice_view_adapter nice_view
  - board: halcyon_wireless//zmk
    shield: halcyon_to_promicro splitkb_aurora_corne_right mod_battery_lipo mod_cirque_hw_right
```


# Local builds

Building the Halcyon boards and shields is of course also possible locally. For this you will just need to follow the [ZMK guide](https://zmk.dev/docs/development/local-toolchain/setup).

The only thing of note is that you will need to add the following repository as a module.
- [splitkb/zmk-halcyon-module](https://github.com/splitkb/zmk-halcyon-module)


# Config options

ZMK allows for a lot of different configuration options. On our Halcyon keyboards we enable a couple of these options to improve the user experience. It is good to know which options are enabled if you ever plan to customize the firmware.

| Config | Description | Board/shield |
| :--- | :--- | :--- |
| CONFIG_ZMK_SLEEP | Enable sleep to increase battery life | `halcyon_wireless` |
| CONFIG_BT_CTLR_TX_PWR_PLUS_8 | Increase TX on dongle | `halcyon_dongle` |
| CONFIG_ZMK_IDLE_TIMEOUT=900000 | Increase idle timeout to decrease the number of updates the epaper display does | `mod_display_epaper_*` |
| CONFIG_ZMK_BATTERY_REPORT_INTERVAL=600 | Battery life is weeks to months so updating less makes sense | `mod_battery_*` |
| CONFIG_ZMK_RGB_UNDERGLOW_AUTO_OFF_USB | Makes sure that the EXT_POWER turns off when you unplug the keyboard with coin cell battery to increase battery life | `mod_battery_coincell` |
| CONFIG_ZMK_RGB_UNDERGLOW=y | Enable RGB | `keyboard shield` |
| CONFIG_ZMK_RGB_UNDERGLOW_ON_START=n | Disable RGB on start | `keyboard shield` |
| CONFIG_ZMK_RGB_UNDERGLOW_BRT_MAX=20 | Set a limit to the brightness, so your battery won't drain quickly | `keyboard shield` |


# Customizing module behavior

Within the [ZMK Halcyon module repository](https://github.com/splitkb/zmk-halcyon-module) we define all keyboard shields, controller boards, halcyon module shields and drivers. 

With Zephyr, what ZMK is based on, it's quite easy to override any functions in the keymap file.

As an example, overriding the number of times the LED will blink and at which percentage, can be done like the following in a `.keymap` file.

```c
&battery_empty_led {
    battery_threshold_percentage = <40>;
    num_blinks = <10>;
};
```

## Display

Please look at the [ZMK documentation](https://zmk.dev/docs/config/displays) if you want to change the behavior of the display. You can also look at the [README from the Halcyon Epaper shield](https://github.com/splitkb/zmk-halcyon-module/blob/main/boards/shields/mod_display_epaper/README.md) to see how you can add a custom image.

## Encoder

Keycodes can be changed in the `sensor-bindings` of your keymap where the order is as following; left halcyon encoder, right halcyon encoder, left soldered encoder and right soldered encoder.

## Cirque

Behavior can be changed by adding input processors. You can look at the [ZMK documentation](https://zmk.dev/docs/keymaps/input-processors) for the available options.
