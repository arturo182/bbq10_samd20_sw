
# BB Q10 Keyboard-to-I2C Software

This software is designed to interface a BB Q10 keyboard matrix as a I2C slave.

The key presses are put on a FIFO, which can be queried over I2C.

The software also allows for basic configuration of the behaviour.

The software also controls the backlight of the keyboard.

See [Protocol](#protocol) for details.

## Building

See the `targets` directory for a list of available targets.

	git clone https://github.com/arturo182/bbq10kbd_i2c_sw.git
	cd bbq10kbd_i2c_sw
	make TARGET=<target>

## Implementations

Here are libraries that allow interaction with the boards running this software:

- [Arduino](https://github.com/arturo182/arduino_bbq10kbd)
- [CircuitPython](https://github.com/arturo182/arturo182_CircuitPython_BBQ10Keyboard)
- [Rust (Embedded-HAL)](https://crates.io/crates/bbq10kbd)

## Protocol

The device uses I2C slave interface to communicate, the address can be configured in `app/config/conf_app.h`, the default is `0x1F`.

You can read the values of all the registers, the number of returned values depends on the register.
It's also possible to write to the registers, to do that, apply the write mask `0x80` to the register ID (for example, the backlight registred `0x05` becomes `0x85`.

### The FW Version register (REG_VER = 0x01)

Data written to this register is discarded. Reading this register returns 1 byte, the first nibble contains the major version and the second nibble contains the minor version of the firmware.

### The configuration register (REG_CFG = 0x02)

This register can be read and written to, it's 1 byte in size.

This register is a bit map of various settings that can be changed to customize the behaviour of the firmware.

| Bit    | Name             | Description                                                        |
| ------ |:----------------:| ------------------------------------------------------------------:|
| 7      | CFG_USE_MODS     | Should Alt, Sym and the Shift keys modify the keys being reported. |
| 6      | CFG_REPORT_MODS  | Should Alt, Sym and the Shift keys be reported as well.            |
| 5      | CFG_PANIC_INT    | Currently not implemented.                                         |
| 4      | CFG_KEY_INT      | Should an interrupt be generated when a key is pressed.            |
| 3      | CFG_NUMLOCK_INT  | Should an interrupt be generated when Num Lock is toggled.         |
| 2      | CFG_CAPSLOCK_INT | Should an interrupt be generated when Caps Lock is toggled.        |
| 1      | CFG_OVERFLOW_INT | Should an interrupt be generated when a FIFO overflow happens.     |
| 0      | CFG_OVERFLOW_ON  | When a FIFO overflow happens, should the new entry still be pushed, overwriting the oldest one. If 0 then new entry is lost. |

Defaut value:
`CFG_OVERFLOW_INT | CFG_KEY_INT | CFG_USE_MODS`

### Interrupt status register (REG_INT = 0x03)

When an interrupt happens, the register can be read to check what caused the interrupt. It's 1 byte in size.

| Bit    | Name             | Description                                   |
| ------ |:----------------:| ---------------------------------------------:|
| 7      | N/A              | Currently not implemented.                    |
| 6      | N/A              | Currently not implemented.                    |
| 5      | N/A              | Currently not implemented.                    |
| 4      | INT_PANIC        | Currently not implemented.                    |
| 3      | INT_KEY          | The interrupt was generated by a key press.   |
| 2      | INT_NUMLOCK      | The interrupt was generated by Num Lock.      |
| 1      | INT_CAPSLOCK     | The interrupt was generated by Caps Lock.     |
| 0      | INT_OVERFLOW     | The interrupt was generated by FIFO overflow. |

After reading the register, it has to manually be reset to `0x00`.

### Key status register (REG_KEY = 0x04)

This register contains information about the state of the fifo as well as modified keys. It is 1 byte in size.

| Bit    | Name             | Description                                     |
| ------ |:----------------:| -----------------------------------------------:|
| 7      | N/A              | Currently not implemented.                      |
| 6      | KEY_NUMLOCK      | Is Num Lock on at the moment.                   |
| 5      | KEY_CAPSLOCK     | Is Caps Lock on at the moment.                  |
| 0-4    | KEY_COUNT        | Number of items in the FIFO waiting to be read. |

### Backlight control register (REG_BKL = 0x05)

Internally a PWM signal is generated to control the keyboard backlight, this register allows changing the brightness of the backlight. It is 1 byte in size, `0x00` being off and `0xFF` being the brightest.

Default value: `0xFF`.

### Debounce configuration register (REG_DEB = 0x06)

Currently not implemented.

Default value: 10

### Poll frequency configuration register (REG_FRQ = 0x07)

Currently not implemented.

Default value: 5

### Chip reset register (REG_RST = 0x08)

Reading or writing to this register will cause a SW reset of the chip.

### FIFO access register (REG_FIF = 0x09)

This register can be used to read the top of the key FIFO. It returns two bytes, a key state and a key code.

Possible key states:

| Value  | State                   |
| ------ |:-----------------------:|
| 1      | Pressed                 |
| 2      | Pressed and Held        |
| 3      | Released                |
