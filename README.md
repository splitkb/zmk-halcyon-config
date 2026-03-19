# Official Splitkb.com Halcyon ZMK config

This is the Splitkb Halcyon ZMK config repository. It allows for an external set of ZMK keymaps with or without Halcyon modules to be defined and compiled. If you want to add support to your existing keyboard, please look at the [porting guide](PORTING.md).


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

*The files for the Aurora and Kyria rev3 shields can be found in the [Aurora branch](https://github.com/splitkb/zmk-halcyon-config/tree/aurora). If you want to customize your keyboard behavior and keep the Halcyon module functionality, make sure to use that branch.


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
| Halcyon Epaper Display Module | `mod_display_epaper` |
| Halcyon MIP Display Module (TBA) | `mod_display_mip` |


## Initial Setup & Prerequisites

Create a fork of this repository using the `Use this template` button and select `Create a new repository`. 

Follow the steps from the [ZMK documentation](https://zmk.dev/docs/user-setup). When you arrive at [Config Repo Setup](https://zmk.dev/docs/user-setup#config-repo-setup) step, follow the steps where you already have a ZMK config repo on github where you point to the newly cloned fork.


# How to Configure Your Build Targets

By default ZMK will create build targets for the left and right half. The `build.yaml` file will need to be updated to support the various modules as shown above.

This explains how to correctly compose `shield` values when using modules with split keyboards and dongles.

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
* `<module>` = your selected Halcyon module
* The `cmake-args` is only needed for the left side or on dongle builds 


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

### Other Modules

For all other module types:

* Left:

  ```yaml
  shield: <keyboard>_left <battery> <module>
  ```

* Right:

  ```yaml
  shield: <keyboard>_right <battery> <module>
  ```


## 4. Studio Builds

If building a Studio build, append:

```yaml
cmake-args: -DCONFIG_ZMK_STUDIO=y
snippet: studio-rpc-usb-uart
```

To the central half. This will be either the dongle or the Left side when NOT using a dongle.

# Local builds

Building the Halcyon boards and shields is of course also possible locally. For this you will just need to follow the [ZMK guide](https://zmk.dev/docs/development/local-toolchain/setup).

The only thing of note is that you will need to add the following repositories as modules.
- [splitkb/zmk-halcyon-module](https://github.com/splitkb/zmk-halcyon-module)
- [splitkb/zmk-driver-charge-led](https://github.com/splitkb/zmk-driver-charge-led)
- [splitkb/zmk-driver-vik-sleep](https://github.com/splitkb/zmk-driver-vik-sleep)
