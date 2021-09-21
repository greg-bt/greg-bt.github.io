---
layout: writeup
title: "4 - Secret Location Base"
ctf: Google CTF 2021 Beginners Quest
nextWrite: "Google2021Beginners-05"
---
For this challenge we are given a file.
Running the file command we can see the file is a Zip archive.
The archive contains two files: `chal.c` and `pico.uf2`

Inspecting `chal.c` we can see that it contains code written in c that controls some gpio pins on what would appear to be a raspberry pi pico.
```c
#include <stdbool.h>

#include "hardware/gpio.h"
#include "hardware/structs/sio.h"
#include "pico/stdlib.h"

int main(void)
{
	for (int i = 0; i < 8; i++) {
		gpio_init(i);
		gpio_set_dir(i, GPIO_OUT);
	}
	gpio_put_all(0);

	for (;;) {
        gpio_set_mask(67);
		gpio_clr_mask(0);
		sleep_us(100);
		gpio_set_mask(20);
		gpio_clr_mask(3);
		sleep_us(100);
		gpio_set_mask(2);
		gpio_clr_mask(16);
		sleep_us(100);
        etc..
        ect..
```

<h2>Bitmasks Theory</h2>

The gpio pins seem to be set and reset according to bitmasks. A bitmask uses a logical operation to transform an array of bits using another array. For example,
```
Mask:   00110100
Value:  00010101
----- AND ------
Result: 00010100
```

<h2>Reversing time</h2>

We can write a script in python to carry out this task using the values from the c code. Carefully copying over the values used in the c code to a python file we can  write out an algorithm to apply the masks in sequence and then output the resulting bytes as ascii characters. That code looks a little something like this:

```python
maskArray = [67,0,20,3,2,16,57,4,0, etc ...
switch = True
val = 0
flag = ""

for mask in maskArray:
    if switch:
        # Apply next mask with OR operator
        val = val | mask
    else:
        # Apply next mask with XOR operator
        val = val ^ mask
        flag += (chr(val))
    switch = not switch

print(flag)
```

The script will loop through each mask, setting and clearing the bytes in an alternating pattern. The resulting value at the end of each pair of masks in then added to the `flag` string.
Once the script has ran it prints out the flag: `CTF{be65dfa2355e5309808a7720a615bca8c82a13d7}`