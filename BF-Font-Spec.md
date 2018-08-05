# RFC#0001 - Proposal for a MAX7456 font specification for use with flight controllers

## Purpose

Betaflight (and other Baseflight forks) use the MAX7456 chip to provide OSD functionality. This chip only has the capability to store 256 different glyphs (characters) in its memory. This RFC is a proposal for how we can add versioning metadata to the font to allow the best use of those 256 glyphs - and to allow the choice of for example bigger boot (splash) logo - or more detailed artificial horizon and battery bars.

## MAX7456 Background

The character memory on the MAX7456 chip is capable of storing 256 characters, each of which is 12x18 pixels. Each character requires 54 bytes of storage, but for ease of addressing this is expanded to 64 bytes leaving 10 bytes of unused space. Unused bytes are defaulted to `0x55`. Fonts can be saved\loaded from .mcm files which are just text files consisting of a header line of 'MAX7456' followed by a bit dump of the character memory.

Characters in positions `0x00` and `0xFF` have special uses:

- `0x00` Is used to blank the screen on video initialization.
- `0xFF` Is a byte used to terminate auto-increment mode when writing to the display memory - meaning this character can only be used in single character mode.

## Metadata storage

To allow the firmware to determine the "capabilities" of the font installed it is proposed to store metadata in the font. It is proposed to use `0xFF` for metadata as excluding this character from use will avoid issues with code using the auto-increment display write mode. Also in the currently used BF fonts this character is completely transparent meaning all bytes are set to `0x55`. The proposed layout of the 64 bytes for this "Info Character" is layed out below - the first four bytes are version information and shouldn't change between specification versions, the following bytes are helper information that may vary by specification version.

- Byte 1 - Info Character identifier
- Byte 2 - Font Metadata Version
- Byte 3 - Firmware Type Identifier
- Byte 4 - Font ID

- Byte 5 - ASCII Space Offset
- Byte 6 - Symbol Space Offset
- Byte 7 - Pilot Logo Space Offset
- Byte 8 - Boot Logo Space Offset

- Bytes 9 - 63 Reserved for future use - set to `0x00`
- Byte 64 - CRC or Checksum byte

### Info Character ID

This is to help identify this character as a version character - as the current fonts have this byte as `0x55` it is proposed to set this to `0xFF`

### Font Metadata Version

Specifies the layout of the rest of the metadata within the "Info Character" i.e bytes 3-64. For Version 1 this will be as specified above:

- Byte 3 - Firmware Type Identifier
- Byte 4 - Font ID

- Byte 5 - ASCII Space Offset
- Byte 6 - Symbol Space Offset
- Byte 7 - Pilot Logo Space Offset
- Byte 8 - Boot Logo Space Offset

- Bytes 9 - 63 Reserved for future use - set to `0x00`
- Byte 64 - CRC or Checksum byte

### Firmware Type Identifier

Determines the class of firmware that this font will work with as iNav and Betaflight have different OSD display functions requiring different symbols. A part example would be:

`0x01` - Layout is BF layout e.g. RSSI 1 Character, Home 1 Character, Artificial Horizon 10 Characters, Battery Bar 5 Characters, etc.

`0x02` - Layout is iNav e.g. RSSI 1 character, Direction Symbols 16 Characters, Artificial Horizon 12 characters, RTH 1 Character, etc.

Another example (taken from the BF layout) could be:

| Example Glyph                  | Hex  | Dec | Symbol Name                     |
| -------------------------------|------| --- | ------------------------------- |
| ![096.png](osd_images/096.png) | 0x60 | 096 | SYM_ARROW_SOUTH                 |
| ![097.png](osd_images/097.png) | 0x61 | 097 | SYM_ARROW_2                     |
| ![098.png](osd_images/098.png) | 0x62 | 098 | SYM_ARROW_3                     |
| ![099.png](osd_images/099.png) | 0x63 | 099 | SYM_ARROW_4                     |
| ![100.png](osd_images/100.png) | 0x64 | 100 | SYM_ARROW_EAST                  |
| ![101.png](osd_images/101.png) | 0x65 | 101 | SYM_ARROW_6                     |
| ![102.png](osd_images/102.png) | 0x66 | 102 | SYM_ARROW_7                     |
| ![103.png](osd_images/103.png) | 0x67 | 103 | SYM_ARROW_8                     |
| ![104.png](osd_images/104.png) | 0x68 | 104 | SYM_ARROW_NORTH                 |
| ![105.png](osd_images/105.png) | 0x69 | 105 | SYM_ARROW_10                    |
| ![106.png](osd_images/106.png) | 0x6A | 106 | SYM_ARROW_11                    |
| ![107.png](osd_images/107.png) | 0x6B | 107 | SYM_ARROW_12                    |
| ![108.png](osd_images/108.png) | 0x6C | 108 | SYM_ARROW_WEST                  |
| ![109.png](osd_images/109.png) | 0x6D | 109 | SYM_ARROW_14                    |
| ![110.png](osd_images/110.png) | 0x6E | 110 | SYM_ARROW_15                    |
| ![111.png](osd_images/111.png) | 0x6F | 111 | SYM_ARROW_16                    |

This wil also determine the order and size of the symbol "blocks" within the character memory or font file. For example:

| Memory Location | Symbol Block |
| --------------- | --------------------------------------------------------- |
|`0x00`           | System Reserved - Blank Character                         |
|`0x01 - 0x3F`    | ASCII Standard (0-9, A-Z, + some characters such as ?#%)  |
|`0x40 - 0xB4`    | OSD Symbol Space                                          |
|`0xB4 - 0xB7`    | Pilot Logo (2x2)                                          |
|`0xB8 - 0xFE`    | Boot (Splash) Logo (4x24)                                 |
|`0xFF`           | System Reserved - Info Character                          |

Example font block layout:

![Example font block layout](images/block.png)

### Font ID

The ID of the font - This is to allow auto loading etc. for system fonts. Custom or user fonts would set this byte to `0x00` to disable this functionality.

- Custom `0x00`
- Default `0x01`
- Bold `0x02`
- Large `0x03`
- Extra Large `0x04`
- Betaflight `0x05`
- Digital `0x06`
- Clarity `0x07`
- Vision `0x08`

### Space Offsets

These are mainly included to help tooling - e.g. the different blocks could be highlighted differently in the editor.

### Checksum

This sounds like a good idea - but the specification of the CRC\Checksum algorithm is TBD.
