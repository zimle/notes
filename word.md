# Microsoft Word

## Math formulas

[Microsoft Word](https://en.wikipedia.org/wiki/Microsoft_Word) supports math formulas, even in tex form. But after fiddling 20 minutes because it did not work, I found a much cleaner and better solution than typing in Word:

1. Type the tex code in an editor of your choice, e.g.

    ```tex
    $$
    \omega_{X} = f^\ast(\mathcal{L}^\vee \otimes \omega_S) \otimes \mathcal{O}_X\Big(\sum_{s \neq \eta} a_s F_s\Big),
    $$
    ```

    which should be rendered as

    $$
    \omega_{X} = f^\ast(\mathcal{L}^\vee \otimes \omega_S) \otimes \mathcal{O}_X\Big(\sum_{s \neq \eta} a_s F_s\Big).
    $$

2. Copy the code into your clipboard and convert it via [pandoc](https://pandoc.org/demos.html) into a word file `temp;:

    ```bash
    # git bash
    cat /dev/clipboard | pandoc -f markdown -o temp.docx
    # example linux
    xclip -o -selection c | pandoc -f markdown -o temp.docx
    ```

3. Open the newly written file `temp.docx` and copy the math formula to your target document
