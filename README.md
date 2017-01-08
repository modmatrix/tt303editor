# tt303

## TT-303 Sysex Specification

I reverse-engineered this format by examining the sysex messages transmitted between my TT-303 and the Cyclone Studio application.

### Envelope

All TT-303 sysex messages use the same basic envelope:

element             | bytes    | value
------------------- | -------- | -----
begin sysex message | 1        | `F0`
manufacturer ID     | 1        | `00`
device ID           | 1        | `01`
model ID            | 1        | `7A`
???                 | 8        | `01 14 38 27 04 07 2B 00`
message body        | variable | One of the specific message types (Global Settings or Pattern) described below.
end sysex message   | 1        | `F7`

### Global Settings

element                          | bytes | value
-------------------------------- | ----- | -----
indicates a global settings dump | 1     | `3F`
???                              | 1     | `00` in all of my tests
???                              | 1     | `01` in all of my tests
???                              | 1     | `04` in all of my tests
system LED color                 | 1     | One of the values from the Colors table
???                              | 1     | `00` in all of my tests

The MIDI channel and VCA gate time settings don't seem to be transmitted.

### Pattern

element                     | bytes    | value
--------------------------- | -------- | -----
track number (zero-based)   | 1        | `00` through `06`
pattern number (zero-based) | 1        | `00` through `1f`. `00`-`07` is Section A, `08`-`0F` is Section B, `10`-`17` is Section AA, and `18`-`1F` is Section BB.
???                         | 2        | `00 0D` in all of my tests
LED color                   | 2        | `3F 0D` to use the system default color, or *`color`* `01` to use a custom color (where *`color`* is a value from [the Colors table](#colors))
pattern timing              | 1        | `00` for normal time, or `01` for triplet time
pattern length (1-based)    | 1        | `00` for an empty pattern, or `01` through `40` for patterns which contain steps
???                         | 19-20    | `00` in all of my tests
pattern contents            | variable | see below

The pattern contents are arranged into three-byte groups.

The first two bytes of each group are notes (or ties or rests). The TT-303 does not use standard MIDI note numbers. Instead, it uses these values:

Note | Normal | Up   | Down
---- | ------ | ---- | ----
C    | `00`   | `10` | `20`  
C#   | `01`   | `11` | `21`  
D    | `02`   | `12` | `22`  
D#   | `03`   | `13` | `23`  
E    | `04`   | `14` | `24`  
F    | `05`   | `15` | `25`  
F#   | `06`   | `16` | `26`  
F    | `07`   | `17` | `27`  
G#   | `08`   | `18` | `28`  
A    | `09`   | `19` | `29`  
A#   | `0A`   | `1A` | `2A`  
B    | `0B`   | `1B` | `2B`  
C    | `0C`   | `1C` | `2C`  
tie  | `2D`
rest | `3D`
null | `0D`

Note that, although `2C` (the rightmost C on the TT-303, with the "Down" modifier applied) and `00` (the leftmost C, unmodified) are musically the same note, they are considered distinct. (The same is true for `0C` and `10`).

For a pattern with an odd number of steps, the second byte in the last group will use the null value as a placeholder.

The third byte describes any accent and slide modifiers for the two notes. It is a simple bitmask:

meaning              | value
-------------------- | -----
first note - slide   | `01`
first note - accent  | `02`
second note - slide  | `04`
second note - accent | `08`



### <a name="colors"></a> Colors

color        | value
------------ | ----- 
magenta      | `00`
red          | `01`
orange       | `02`
yellow       | `03`
light green  | `04`
dark green   | `05`
turquoise    | `06`
light blue   | `07`
dark blue    | `08`
medium blue  | `09`
dark purple  | `0A`
light purple | `0B`
ice blue     | `0C`
