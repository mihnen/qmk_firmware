# QMK Keyboard Guidelines

Since starting, QMK has grown by leaps and bounds thanks to people like you who contribute to creating and maintaining our community keyboards. As we've grown we've discovered some patterns that work well, and ask that you conform to them to make it easier for other people to benefit from your hard work.


## Use QMK Lint

We have provided a tool, `qmk lint`, which will let you check over your keyboard for problems. We suggest using it frequently while working on your keyboard and keymap. 

Example passing check:

```
$ qmk lint -kb rominronin/katana60/rev2
Ψ Lint check passed!
```

Example failing check:

```
$ qmk lint -kb clueboard/66/rev3
☒ Missing keyboards/clueboard/66/rev3/readme.md
☒ Lint check failed!
```

## Naming Your Keyboard/Project

All keyboard names are in lower case, consisting only of letters, numbers, and underscore (`_`). Names may not begin with an underscore. Forward slash (`/`) is used as a sub-folder separation character.

The names `test`, `keyboard`, and `all` are reserved for make commands and may not be used as a keyboard or subfolder name.

Valid Examples:

* `412_64`
* `chimera_ortho`
* `clueboard/66/rev3`
* `planck`
* `v60_type_r`

## Sub-folders

QMK uses sub-folders both for organization and to share code between revisions of the same keyboard. You can nest folders up to 4 levels deep:

```
qmk_firmware/keyboards/top_folder/sub_1/sub_2/sub_3/sub_4
```

If a sub-folder has a `keyboard.json` file it will be considered a compilable keyboard. It will be available in QMK Configurator and tested with `make all`. If you are using a folder to organize several keyboards from the same maker you should not have a `keyboard.json` file.

::: tip
When configuring a keyboard with multiple revisions (like the `clueboard/66` example below), an `info.json` file at the top keyboard level (eg. `clueboard/66`) should be used for configuration shared between revisions. Then `keyboard.json` in each revision directory containing revision-specific configuration, and indicating a buildable keyboard.
:::

Example:

Clueboard uses sub-folders for both purposes, organization and keyboard revisions.

* [`qmk_firmware`](https://github.com/qmk/qmk_firmware/tree/master)
  * [`keyboards`](https://github.com/qmk/qmk_firmware/tree/master/keyboards)
    * [`clueboard`](https://github.com/qmk/qmk_firmware/tree/master/keyboards/clueboard)  &larr; This is the organization folder, there's no `keyboard.json` file
      * [`60`](https://github.com/qmk/qmk_firmware/tree/master/keyboards/clueboard/60)  &larr; This is a compilable keyboard - it has a `keyboard.json` file
      * [`66`](https://github.com/qmk/qmk_firmware/tree/master/keyboards/clueboard/66) &larr; This is not a compilable keyboard - a revision must be specified
        * [`rev1`](https://github.com/qmk/qmk_firmware/tree/master/keyboards/clueboard/66/rev1) &larr; compilable: `make clueboard/66/rev1`
        * [`rev2`](https://github.com/qmk/qmk_firmware/tree/master/keyboards/clueboard/66/rev2) &larr; compilable: `make clueboard/66/rev2`
        * [`rev3`](https://github.com/qmk/qmk_firmware/tree/master/keyboards/clueboard/66/rev3) &larr; compilable: `make clueboard/66/rev3`

## Keyboard Folder Structure

Your keyboard should be located in `qmk_firmware/keyboards/` and the folder name should be your keyboard's name as described in the previous section. Inside this folder should be several files, some of which are optional:

* `readme.md`
* `keyboard.json` (or `info.json`)
* `config.h`
* `rules.mk`
* `<keyboard_name>.c`
* `<keyboard_name>.h`

### `readme.md`

All projects need to have a `readme.md` file that explains what the keyboard is, who made it and where it's available. If applicable, it should also contain links to more information, such as the maker's website. Please follow the [published template](documentation_templates#keyboard-readmemd-template).

### `keyboard.json`/`info.json`

The `keyboard.json` file is necessary for your keyboard (or keyboard revision) to be considered a buildable keyboard. The same content is valid in both `info.json` and `keyboard.json`. For the available configuration options of this file, see the [reference page](reference_info_json). This file is also used by the [QMK API](https://github.com/qmk/qmk_api), and by the [QMK Configurator](https://config.qmk.fm/) to display a representation of the available layouts of your keyboard.

Additionally, this is where layouts available on your keyboard are defined. If you only have a single layout, it should be named `LAYOUT`. When defining multiple layouts, you should have a base layout, named `LAYOUT_all`, that supports all possible switch positions in your matrix, even if that layout is impossible to build physically. This is the layout that should be used in the `default` keymap. You should then have additional keymaps named `default_<layout>` that configure keymaps for the other layouts. Layout macro names are entirely lowercase, except for the prefix of `LAYOUT`.

As an example, if you have a 60% PCB that supports ANSI and ISO, you might define the following layouts and keymaps:

| Layout Name | Keymap Name  | Description                              |
|-------------|--------------|------------------------------------------|
| LAYOUT_all  | default      | A layout that supports both ISO and ANSI |
| LAYOUT_ansi | default_ansi | An ANSI layout                           |
| LAYOUT_iso  | default_iso  | An ISO layout                            |

::: tip
Providing only `LAYOUT_all` is invalid, as is providing a `LAYOUT` when multiple layouts are present.
:::

### `config.h`

Some projects will need to have a `config.h` that configures parameters that are not possible to be set in `keyboard.json`. This is not a required file.

The `config.h` files can also be placed in sub-folders, and the order in which they are read is as follows:

* `keyboards/top_folder/config.h`
  * `keyboards/top_folder/sub_1/config.h`
    * `keyboards/top_folder/sub_1/sub_2/config.h`
      * `keyboards/top_folder/sub_1/sub_2/sub_3/config.h`
        * `keyboards/top_folder/sub_1/sub_2/sub_3/sub_4/config.h`
          * [`.build/objs_<keyboard>/src/info_config.h`](data_driven_config#add-code-to-generate-it) see [Data Driven Configuration](data_driven_config)
          * `users/a_user_folder/config.h`
          * `keyboards/top_folder/keymaps/a_keymap/config.h`
        * `keyboards/top_folder/sub_1/sub_2/sub_3/sub_4/post_config.h`
      * `keyboards/top_folder/sub_1/sub_2/sub_3/post_config.h`
    * `keyboards/top_folder/sub_1/sub_2/post_config.h`
  * `keyboards/top_folder/sub_1/post_config.h`
* `keyboards/top_folder/post_config.h`

The `post_config.h` file can be used for additional post-processing, depending on what is specified in the `config.h` file. For example, if you define the `IOS_DEVICE_ENABLE` macro in your keymap-level `config.h` file as follows, you can configure more detailed settings accordingly in the `post_config.h` file:

* `keyboards/top_folder/keymaps/a_keymap/config.h`
  ```c
  #define IOS_DEVICE_ENABLE
  ```
* `keyboards/top_folder/post_config.h`
  ```c
  #ifndef IOS_DEVICE_ENABLE
    // USB_MAX_POWER_CONSUMPTION value for this keyboard
    #define USB_MAX_POWER_CONSUMPTION 400
  #else
    // fix iPhone and iPad power adapter issue
    // iOS devices need less than 100
    #define USB_MAX_POWER_CONSUMPTION 100
  #endif
  
  #ifdef RGBLIGHT_ENABLE
    #ifndef IOS_DEVICE_ENABLE
      #define RGBLIGHT_LIMIT_VAL 200
      #define RGBLIGHT_VAL_STEP 17
    #else
      #define RGBLIGHT_LIMIT_VAL 35
      #define RGBLIGHT_VAL_STEP 4
    #endif
    #ifndef RGBLIGHT_HUE_STEP
      #define RGBLIGHT_HUE_STEP 10
    #endif
    #ifndef RGBLIGHT_SAT_STEP
      #define RGBLIGHT_SAT_STEP 17
    #endif
  #endif
  ```

::: tip
If you define options using `post_config.h` as in the above example, you should not define the same options in the keyboard- or user-level `config.h`.
:::

### `rules.mk`

This file is typically used to configure hardware drivers (eg. pointing device), or to include additional C files in compilation. This is not a required file.

The `rules.mk` file can also be placed in a sub-folder, and its reading order is as follows:

* `keyboards/top_folder/rules.mk`
  * `keyboards/top_folder/sub_1/rules.mk`
    * `keyboards/top_folder/sub_1/sub_2/rules.mk`
      * `keyboards/top_folder/sub_1/sub_2/sub_3/rules.mk`
        * `keyboards/top_folder/sub_1/sub_2/sub_3/sub_4/rules.mk`
          * `keyboards/top_folder/keymaps/a_keymap/rules.mk`
          * `users/a_user_folder/rules.mk`
        * `keyboards/top_folder/sub_1/sub_2/sub_3/sub_4/post_rules.mk`
      * `keyboards/top_folder/sub_1/sub_2/sub_3/post_rules.mk`
    * `keyboards/top_folder/sub_1/sub_2/post_rules.mk`
  * `keyboards/top_folder/sub_1/post_rules.mk`
* `keyboards/top_folder/post_rules.mk`
* `common_features.mk`

Many of the settings written in the `rules.mk` file are interpreted by `common_features.mk`, which sets the necessary source files and compiler options.

The `post_rules.mk` file can interpret `features` of a keyboard-level before `common_features.mk`.  For example, when your designed keyboard has the option to implement backlighting or underglow using rgblight.c, writing the following in the `post_rules.mk` makes it easier for the user to configure the `rules.mk`.

* `keyboards/top_folder/keymaps/a_keymap/rules.mk`
  ```make
  # Please set the following according to the selection of the hardware implementation option.
  RGBLED_OPTION_TYPE = backlight   ## none, backlight or underglow
  ```
* `keyboards/top_folder/post_rules.mk`
  ```make
  ifeq ($(filter $(strip $(RGBLED_OPTION_TYPE))x, nonex backlightx underglowx x),)
     $(error unknown RGBLED_OPTION_TYPE value "$(RGBLED_OPTION_TYPE)")
  endif

  ifeq ($(strip $(RGBLED_OPTION_TYPE)),backlight)
    RGBLIGHT_ENABLE = yes
    OPT_DEFS += -DRGBLIGHT_LED_COUNT=30
  endif
  ifeq ($(strip $(RGBLED_OPTION_TYPE)),underglow)
    RGBLIGHT_ENABLE = yes
    OPT_DEFS += -DRGBLIGHT_LED_COUNT=6
  endif
  ```

::: tip
See `build_keyboard.mk` and `common_features.mk` for more details.
:::

### `<keyboard_name.c>`

This file should contain C code required for the functionality of your keyboard, for example hardware initialisation code, OLED display code, and so on. This file should only contain code necessary for the keyboard to work, and *not* things that should be left to the end user to configure in their keymap. This file is automatically included in compilation if it exists. This is not a required file.

The following functions are typically defined in this file:

* `void matrix_init_kb(void)`
* `void matrix_scan_kb(void)`
* `bool process_record_kb(uint16_t keycode, keyrecord_t *record)`
* `bool led_update_kb(led_t led_state)`

### `<keyboard_name.h>`

This file can contain function prototypes for custom functions and other header file code utilised by `<keyboard_name>.c`. The `<keyboard_name>.c` file should include this file. This is not a required file.

## Image/Hardware Files

In an effort to keep the repo size down we do not accept binary files of any format, with few exceptions. Hosting them elsewhere (such as <https://imgur.com>) and linking them in the `readme.md` is preferred. Hardware files such as plates, cases, and PCBs can be published in a personal repository or elsewhere, and linked to by your keyboard's `readme.md` file.

## Keyboard Defaults

Given the amount of functionality that QMK exposes it's very easy to confuse new users. When putting together the default firmware for your keyboard we recommend limiting your enabled features and options to the minimal set needed to support your hardware. Recommendations for specific features follow.

### Magic Keycodes and Command

[Magic Keycodes](keycodes_magic) and [Command](features/command) are two related features that allow a user to control their keyboard in non-obvious ways. We recommend you think long and hard about if you're going to enable either feature, and how you will expose this functionality. Keep in mind that users who want this functionality can enable it in their personal keymaps without affecting all the novice users who may be using your keyboard as their first programmable board.

If your keyboard does not have 2 shift keys you should provide a working default for `IS_COMMAND`, even when you have set `COMMAND_ENABLE = no`. This will give your users a default to conform to if they do enable Command.

## Custom Keyboard Programming

As documented on [Customizing Functionality](custom_quantum_functions) you can define custom functions for your keyboard. Please keep in mind that your users may want to customize that behavior as well, and make it possible for them to do that. If you are providing a custom function, for example `process_record_kb()`, make sure that your function calls the `_user()` version of the call too. You should also take into account the return value of the `_user()` version, and only run your custom code if the user returns `true`.

## Non-Production/Handwired Projects

We're happy to accept any project that uses QMK, including handwired ones, but we have a separate `/keyboards/handwired/` folder for them, so the main `/keyboards/` folder doesn't get overcrowded. If a prototype project becomes a production project at some point in the future, we'd be happy to move it to the main `/keyboards/` folder!

## Warnings as Errors

When developing your keyboard, keep in mind that all warnings will be treated as errors - these small warnings can build-up and cause larger errors down the road (and keeping them is generally a bad practice).

## Copyright Blurb

If you're adapting your keyboard's setup from another project, but not using the same code, be sure to update the copyright header at the top of the files to show your name, in this format:

```c
Copyright 2017 Your Name <your@email.com>
```

If you are modifying someone else's code and have made only trivial changes you should leave their name in the copyright statement. If you have done significant work on the file you should add your name to theirs, like so:

```c
Copyright 2017 Their Name <original_author@example.com> Your Name <you@example.com>
```

The year should be the first year the file is created. If work was done to that file in later years you can reflect that by appending the second year to the first, like so:

```c
Copyright 2015-2017 Your Name <you@example.com>
```

## License

The core of QMK is licensed under the [GNU General Public License](https://www.gnu.org/licenses/licenses.en.html). If you are shipping binaries for AVR processors you may choose either [GPLv2](https://www.gnu.org/licenses/old-licenses/gpl-2.0.html) or [GPLv3](https://www.gnu.org/licenses/gpl.html). If you are shipping binaries for ARM processors you must choose [GPL Version 3](https://www.gnu.org/licenses/gpl.html) to comply with the [ChibiOS](https://www.chibios.org) GPLv3 license.
