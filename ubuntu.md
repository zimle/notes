# Ubuntu

## Trivia

- If scroll lock key is always pressed though not available with laptop keyboard, using LibreOffice might get impossible as navigating with the arrows is disabled.
    To disable this, the following reconfiguration might help (from [SO](https://superuser.com/questions/1514717/how-to-disable-scroll-lock-key-in-linux-permanently))

    ```bash
    # read the key binding
    dconf read /org/gnome/desktop/input-sources/xkb-options
    ['grp_led:scroll']
    # change it
    dconf write /org/gnome/desktop/input-sources/xkb-options "['grp:alt_shift_toggle']"
    ```
