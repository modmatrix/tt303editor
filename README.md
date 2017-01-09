# Bass Bot TT-303 Sysex Specification

I reverse-engineered this format by examining the sysex messages transmitted between my TT-303 and the [Cyclone Studio](http://www.cyclone-analogic.fr/en/content/10-download) application. I used a rev1.0 BassBot running firmware v2.0, and Cyclone Studio 1.1.14051 for MacOS.

I've made every effort to ensure its accuracy, but I am not responsible if you lose your patterns or fry your TT-303. **Back up your Bass Bot before experimenting with sysex!**

I haven't been able to figure out what some of the bytes mean. They could be checksums, version numbers, serial numbers, placeholders for future use, or anything else. If you figure it out, please let me know!

* [Envelope](#envelope)
* [Sysex messages recognized by the TT-303](#sysex-messages-recognized-by-the-tt-303)
  * [Request Self-Identification](#request-self-identification)
  * [Request Backup](#request-backup)
  * [Propose Restore](#propose-restore)
  * [Write User Pattern](#write-user-pattern)
* [Sysex messages sent by the TT-303](#sysex-messages-sent-by-the-tt-303)
  * [Self-Identification](#self-identification)
  * [Dump User Pattern](#dump-user-pattern) (TODO: duplicate anchor name)
  * [Dump Global Settings](#dump-global-settings)
  * [Accept or Decline Restore](#accept-or-decline-restore)
  * [Ready/Acknowledged](#ready-acknowledged)
* [Constants and Data Structures](#constants-and-data-structures)
  * [User Pattern](#user-pattern) (TODO: duplicate anchor name)
  * [Note Numbers](#note-numbers)
  * [LED Colors](#ed-colors)

## Envelope

All sysex messages sent and received by the TT-303 use the same basic envelope, which adheres to the MIDI system exclusive standard:

| Element             | Bytes    | Value                                                                                                               |
|:------------------- |:-------- |:------------------------------------------------------------------------------------------------------------------- |
| begin sysex message | 1        | `F0`                                                                                                                |
| manufacturer ID     | 3        | `00 01 7A` (Rezonance Labs)                                                                                         |
| device or model ID? | 1        | `01`                                                                                                                |
| ???                 | 6        | always `38 27 04 07 2B 00` in my tests; not included in [Request Self-Identification](#request-self-identification) |
| message type        | 1        | The ID for one of the specific message types described below.                                                       |
| message body        | variable | One of the specific message types described below.                                                                  |
| end sysex message   | 1        | `F7`                                                                                                                |

I suspect that the unknown six-byte sequence (or at least part of it) uniquely identifies this particular TT-303.

005910 - my serial # inside battery compartment

## Sysex messages recognized by the TT-303

### Request Self-Identification

3F 3F 0F 3F 3F 0F 00 00 00

Message type 10. Ask any TT-303s that are listening to identify themselves.

The TT-303 should respond with its [Self-Identification](#self-identification) message.

### Request Backup

Message type 13. Ask the TT-303 to send a full memory backup.

3F 3F 0F 3F 3F 0F 38 27 04 07 2B 00 01 00 00 00 00 00

The TT-303 should respond with the following sequence of 232 sysex messages:

| Message type                                  | Number sent | Notes |
|:--------------------------------------------- |:----------- |:----- |
| [Dump User Pattern](#dump-user-pattern)       | 224         |       |
| ???                                           | 7           | All seven messages are identical, and appear to be empty envelopes: `F0 00 01 7A 01 14 38 27 04 07 2B 00 F7`. These are probably supposed to be tracks, but track backup doesn't seem to be working properly: if I create a backup, modify a track, and then restore the backup, the track does not revert to the original version. |
| [Dump Global Settings](#dump-global-settings) | 1           |       |

### Propose Restore

Message type 13. Asks the TT-303 whether it will accept an overwrite of its memory. Clients (*e.g.*, Cyclone Studio, or other applications which wish to write to the TT-303) should repeatedly poll the TT-303 with this message (about once a second) until the TT-303 accepts.

The TT-303 will respond with an [Accept or Decline Restore](#accept-or-decline-restore) message.

3F 3F 0F 3F 3F 0F 38 27 04 07 2B 00 00 00 01

### Write User Pattern

Message type 14. Overwrite a single user pattern. The TT-303 will respond with a [Ready/Acknowledged](#ready-acknowledged) message. Follows the [User Pattern](#user-pattern) data structure.




## Sysex messages sent by the TT-303

### Self-Identification (message type ID 11)

Provides the TT-303's serial number, hardware revision, and OS version.

| Element              | bytes | value |
|:-------------------- |:----- |:----- |
| ???                  | 6     | `3F 3F 0F 3F 3F 0F` |
| ???                  | 6     | `10 00 00 00 2F 0C` |
| serial number part 1 | 2     | `00 16` (TODO: this is mine only) |
| ???                  | 1     | `00` in my tests |
| serial number part 2 | 2     | `00 28` (TODO: this is mine only) |
| ???                  | 1     | `00` in my tests |
| serial number part 3 | 2     | `0F 07` However, this is displayed as 0F47 in Cyclone Studio. Why? (TODO: this is mine only) |
| ???                  | 1     | `04` in my tests. Possibly used in conjunction with the previous two bytes? |
| serial number part 4 | 2     | `34 31` (TODO: this is mine only) |
| ???                  | 1     | `00` in my tests |
| serial number part 5 | 2     | `37 36` (TODO: this is mine only) |
| ???                  | 1     | `00` in my tests |
| serial number part 6 | 2     | `33 32` |
| ???                  | 1     | `00` in my tests |
| ???                  | 3     | `02 00 00` in my tests - possibly firmware major and minor version numbers? |
| ???                  | 3     | `01 00 00` in my tests - possibly hardware major and minor version numbers? |
| ???                  | 3     | `01 00 00` in my tests - possibly hardware major and minor version numbers? |

(That's not a typo above: the message really does include two copies of `01 00 00` at the end.)

### Dump User Pattern

Message type 14. Describes a single user pattern. See the [User Pattern](#data-user-pattern) data structure.

### Dump Global Settings

Message type 14. Describes the TT-303's global settings (anything not specific to a pattern or track).

| Element          | Bytes | Value |
|:---------------- |:----- |:----- |
| ???              | 1     | `3F` in all of my tests |
| ???              | 1     | `00` in all of my tests |
| ???              | 1     | `01` in all of my tests |
| ???              | 1     | `04` in all of my tests |
| system LED color | 1     | One of the values from the [LED Colors](#led-colors) table |
| ???              | 1     | `00` in all of my tests |

The MIDI channel and VCA gate time settings don't seem to be transmitted.

### Accept or Decline Restore

When the TT-303 receives a [Propose Restore](#propose-restore) message, it will respond with this.

| Element           | Bytes | Value |
|:----------------- |:----- |:----- |
| ???               | 14    | `3F 3F 0F 3F 3F 0F 13 00 04 00 00 00 00 26` in my tests |
| accept or decline | 1     | `00` to decline, or `01` to accept |

### Ready/Acknowledged

The TT-303 responds to [Write Pattern](#write-pattern) requests with this message. I assume it means that the TT-303 has accepted the write request, completed the write, and is ready for further instructions.

00  F0 00 01 7A 01 11 38 27  04 07 2B 00 38 27 04 07  |   z  8'  + 8'  |
10  2B 00 14 00 00 F7                                 |+     |





## Constants and data structures

### User Pattern

This data structure is used in the [Write User Pattern](#write-user-pattern) and [User Pattern](#user-pattern) messages.

| Element                     | Bytes    | Value |
|:--------------------------- |:-------- |:----- |
| track number (zero-based)   | 1        | `00` through `06` |
| pattern number (zero-based) | 1        | `00` through `1F`. `00`-`07` is Section A, `08`-`0F` is Section B, `10`-`17` is Section AA, and `18`-`1F` is Section BB. |
| ???                         | 2        | `00 0D` in all of my tests |
| LED color                   | 2        | `3F 0D` to use the system default color, or *`color`*` 01` to use a custom color (where *`color`* is a value from [the Colors table](#colors)) |
| pattern timing              | 1        | `00` for normal time, or `01` for triplet time |
| pattern length (1-based)    | 1        | `01` through `40` (or `00` for an empty pattern) |
| ???                         | 19       | `00` in all of my tests |
| pattern contents            | variable | see below |

The pattern contents are arranged into three-byte groups.

The first two bytes of each group are notes (or ties or rests). The TT-303 does not use standard MIDI note numbers. See [Note Numbers](#note-numbers) for the proper values.

For a pattern with an odd number of steps, the second byte in the last group will use the null value (`0D`) as a placeholder.

The third byte describes any accent and slide modifiers for the two notes. It is a simple bitmask:

| Modifier              | Bit   |
|:--------------------- |:----- |
| first note - slide    | `01`  |
| first note - accent   | `02`  |
| second note - slide   | `04`  |
| second note - accent  | `08`  |

### Note Numbers

| Note | Normal | Up   | Down |
|:---- |:------ |:---- |:---- |
| C    | `00`   | `10` | `20` |
| C#   | `01`   | `11` | `21` | 
| D    | `02`   | `12` | `22` | 
| D#   | `03`   | `13` | `23` |
| E    | `04`   | `14` | `24` |
| F    | `05`   | `15` | `25` |
| F#   | `06`   | `16` | `26` |
| F    | `07`   | `17` | `27` |
| G#   | `08`   | `18` | `28` |
| A    | `09`   | `19` | `29` |
| A#   | `0A`   | `1A` | `2A` |
| B    | `0B`   | `1B` | `2B` |
| C    | `0C`   | `1C` | `2C` |
| tie  | `2D`   |      |      |
| rest | `3D`   |      |      |
| null | `0D`   |      |      |

Note that, although `2C` (the rightmost C on the TT-303, with the "Down" modifier applied) and `00` (the leftmost C, unmodified) are musically the same note, they are considered distinct. (The same is true for `0C` and `10`).

### LED Colors

| Color        | Value |
|:------------ |:----- |
| magenta      | `00`  |
| red          | `01`  |
| orange       | `02`  |
| yellow       | `03`  |
| light green  | `04`  |
| dark green   | `05`  |
| turquoise    | `06`  |
| light blue   | `07`  |
| dark blue    | `08`  |
| medium blue  | `09`  |
| dark purple  | `0A`  |
| light purple | `0B`  |
| ice blue     | `0C`  |
