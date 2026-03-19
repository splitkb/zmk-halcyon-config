# Porting a Keyboard for Halcyon Module Support

This guide outlines the steps to enable **Halcyon + VIK** or **Halcyon → Pro Micro adapter** support for your ZMK keyboard.

> NOTE: If you do not plan on using Halcyon modules, you can skip these steps and use the standard `halcyon_to_promicro` shield definition as described in the main [readme](README.md). If you only want to add RGB support, go to the [RGB section](#rgb-support) below.

Please follow the [Initial Setup & Prerequisites](README.md) from the main readme first before starting this guide.

Examples of "Halcyon-ified" keyboards can be found in the [Aurora branch](https://github.com/splitkb/zmk-halcyon-config/tree/aurora).

## 1. Directory Setup
Navigate to your `zmk-halcyon-config/config` directory. All files created in this guide should reside there unless otherwise noted.


## 2. Preparing the Layout Transform
Create a `<my_keyboard>.dtsi` file. You need to extend the existing keymap transform to accommodate the Halcyon module's extra row.

1. Locate the original `keymap_transform` in the ZMK repository or your keyboard's source `.dtsi`.
2. Copy it into your `.dtsi` and rename it to `halcyon_transform`. Make sure to place it within the `/ { <...> };` brackets. 
3. Increment the `rows` count by 1 (e.g., change `4` to `5`).
4. Add the Halcyon row to the map:

### For Split Keyboards
Add a row at the bottom of the map. For a 4-row keyboard (like the Kyria), the index starts at `4`. Add 5 columns on the left and 5 on the right:
```c
// Example for a split 4-row keyboard (now 5 rows)
        RC(4,0) RC(4,1) RC(4,2) RC(4,3) RC(4,4)     RC(4,9) RC(4,10) RC(4,11) RC(4,12) RC(4,13)
```

### For Non-Split Keyboards
Add 5 columns to a new row at the bottom:
```c
        RC(4,0) RC(4,1) RC(4,2) RC(4,3) RC(4,4)
```


## 3. Updating Physical Layouts

For both ZMK Studio and non-studio builds you need to update the chose physical layout.

```c
/{
    chosen {
        zmk,physical-layout = &halcyon_layout; 
    };
};
```

### Option A: Non-studio
Use this if you don't need ZMK Studio support. It simply links the transform to the layout name.

```c
/ {
    halcyon_layout: halcyon_layout {
        compatible = "zmk,physical-layout";
        display-name = "Halcyon Layout";
        transform = <&halcyon_transform>;
    };
};
```

### Option B: ZMK Studio support
To ensure the Halcyon modules appear correctly in ZMK Studio, you must define their physical positions.

1. Copy the original layout, rename it to `halcyon_layout`, and update the transform to `&halcyon_transform`.  Make sure to place it within the `/ { <...> };` brackets. 
2. Add the physical attributes for the 10 new Halcyon "keys" at the end of the `keys` list.

Placement Tip: Set the `y` coordinate higher than your existing rows and center the `x` coordinates.

```c
            // Add to the end of your keys = <...>; list
            , <&key_physical_attrs 100 100    0  600       0     0     0> // Halcyon button left 1
            , <&key_physical_attrs 100 100  100  600       0     0     0> // Halcyon button left 2
            , <&key_physical_attrs 100 100  200  600       0     0     0> // Halcyon button left 3
            , <&key_physical_attrs 100 100  300  600       0     0     0> // Halcyon button left 4
            , <&key_physical_attrs 100 100  400  600       0     0     0> // Halcyon button left 5
            , <&key_physical_attrs 100 100 1200  600       0     0     0> // Halcyon button right 5
            , <&key_physical_attrs 100 100 1300  600       0     0     0> // Halcyon button right 4
            , <&key_physical_attrs 100 100 1400  600       0     0     0> // Halcyon button right 3
            , <&key_physical_attrs 100 100 1500  600       0     0     0> // Halcyon button right 2
            , <&key_physical_attrs 100 100 1600  600       0     0     0> // Halcyon button right 1

```

Make sure to add the following include at the top of the file: `#include <physical_layouts.dtsi>`


## 4. Configuring the Composite Kscan
Since Halcyon uses a extra scan for its modules, you must combine it with your keyboard's primary matrix using a composite driver.

Update your `.overlay` with the following:
```c
/ {
    chosen {
        zmk,kscan = &kscan_composite;
    };

    kscan_composite: kscan_composite {
        compatible = "zmk,kscan-composite";
        columns = <14>; // Match your keyboard's column count
        rows = <5>;    // Total rows including the Halcyon row
        wakeup-source;

        matrix {
            kscan = <&kscan0>; // Match your keyboard's existing kscan
        };

        halcyon {
            kscan = <&kscan_halcyon>;
            row-offset = <4>; // Number of existing rows (0-indexed)
        };
    };
};
```


## 5. Adding Halcyon Encoders
Halcyon modules often include extra encoders. You need to inject these into your sensor list.

```c
/ {
    sensors: sensors {
        compatible = "zmk,keymap-sensors";
        // Place Halcyon encoders before your physical board encoders
        sensors = <&left_halcyon_encoder &right_halcyon_encoder &left_encoder &right_encoder>;
        triggers-per-rotation = <20>;
    };
};
```

## 6. Creating split overlays

Due to how the build system works you will now also have to create two additional files.

### <my_keyboard>_left.overlay

Within this file you will only need to include the previously created `.dtsi.`

```c
#include "<my_keyboard>.dtsi"
```

### <my_keyboard>_right.overlay

Next to adding the include, you also need to add a `col-offset` to the `halcyon_transform`. The col-offset can be copied from the original keyboard's source `_right` file.

```c
#include "<my_keyboard>.dtsi"

&halcyon_transform {
    col-offset = <7>;
};
```

## 7. Keymap Integration
Finally, update your `.keymap` file. Every layer must now include the extra row defined in your transform.

```c
// Example bottom row addition for each layer
&kp RGUI &none &none &none &none    &none &none &none &none &kp RGUI
```

Similarly, expand the `sensor-bindings` to the number of encoders you have on the keyboard. If you do not have any encoders on your keyboard and only plan on using a Halcyon encoder, you will not have to update this most likely, unless your keyboard did not have any encoder support before.

```c
// Example sensor row for a keyboard with now 4 encoder options (2 on keyboard and 2 halcyon encoders)
sensor-bindings = <&inc_dec_kp C_VOL_UP C_VOL_DN &inc_dec_kp PG_UP PG_DN &inc_dec_kp C_VOL_UP C_VOL_DN &inc_dec_kp PG_UP PG_DN>;
```


## 8. Dongle Support (Optional)
If you are building a dongle-based setup, refer to the [ZMK Dongle Integration Guide](https://zmk.dev/docs/development/hardware-integration/dongle). Ensure your dongle's overlay also uses the `halcyon_transform` and `halcyon_layout`. You can copy these from the newly created `.dtsi` from your config folder.

In the dongle configuration you also need to make sure to define all encoders available on the keyboard shields. These can be set as `disabled`. The dongle just needs to know that they exist. For the Aurora Corne, an example is shown below.

```c
/ {
    left_halcyon_encoder: halcyon_encoder_left {
        status = "disabled";
    };

    right_halcyon_encoder: halcyon_encoder_right {
        status = "disabled";
    };
    
    left_encoder: left_encoder {
        status = "disabled";
    };

    right_encoder: right_encoder {
        status = "disabled";
    };

    sensors: sensors {
        compatible = "zmk,keymap-sensors";
        sensors = <&left_halcyon_encoder &right_halcyon_encoder &left_encoder &right_encoder>;
        triggers-per-rotation = <20>;
    };
};
```

# RGB Support
If your keyboard shield also has RGB LEDs installed, you will have to configure this for the new board.

## 1. Creating files
You can skip this step if you already created these during the previous steps.

Navigate to your `zmk-halcyon-config/config` directory. All files created in this guide should reside there unless otherwise noted.

Create the following files:
- `<my_keyboard>.dtsi`
- `<my_keyboard>_left.overlay`
- `<my_keyboard>_right.overlay`

Within the `<my_keyboard>_left.overlay` and `<my_keyboard>_right.overlay` you only have to add a include for the main `<my_keyboards>.dtsi`

```c
#include "<my_keyboard>.dtsi"
```

## 2. Adding RGB devicetree config
Within the `<my_keyboard>.dtsi` add the following:

```c
#include <dt-bindings/led/led.h>

/ {
    chosen {
        zmk,underglow = &led_strip;
    };
};

&pinctrl {
    spi2_default: spi2_default {
        group1 {
            psels = <NRF_PSEL(SPIM_SCK, 1, 4)>, /* Not connected */
                    <NRF_PSEL(SPIM_MOSI, 0, 27)>; /* CHANGE */
        };
    };

    spi2_sleep: spi2_sleep {
        group1 {
            psels = <NRF_PSEL(SPIM_SCK, 1, 4)>, /* Not connected */
                    <NRF_PSEL(SPIM_MOSI, 0, 27)>; /* CHANGE */
            low-power-enable;
        };
    };
};

&spi2 {
    compatible = "nordic,nrf-spim";
    status = "okay";

    pinctrl-0 = <&spi2_default>;
    pinctrl-1 = <&spi2_sleep>;
    pinctrl-names = "default", "sleep";

    led_strip: ws2812@0 {
        compatible = "worldsemi,ws2812-spi";

        /* SPI */
        reg = <0>; /* ignored, but necessary for SPI bindings */
        spi-max-frequency = <4000000>;

        /* WS2812 */
        chain-length = <31>; /* CHANGE */
        spi-one-frame = <0x70>;
        spi-zero-frame = <0x40>;

        color-mapping = <LED_COLOR_ID_GREEN 
                LED_COLOR_ID_RED 
                LED_COLOR_ID_BLUE>;
    };
};
```

In this file you need to change the `<NRF_PSEL(SPIM_MOSI, 0, 27)>;` pin to the pin matching your RGB data pin. Also change the `chain-length` to the number of LEDs on your keyboard shield.

## 3. Enabling RGB
Create the `<my_keyboard>.conf` file.

Within this file add the following configuration:
```makefile
# RGB settings
CONFIG_ZMK_RGB_UNDERGLOW=y
CONFIG_ZMK_RGB_UNDERGLOW_ON_START=n
CONFIG_ZMK_RGB_UNDERGLOW_BRT_MAX=20
```

You can of course tune these settings to your liking. We recommend setting the max brightness not too high to limit the current and increase battery life when using the keyboard.
