# Infinite Rotation RFC, Revision 2

## Abstract

Allows an unlimited number of rotations. Settings should allow limiting the maximum number of rotations, for users who don't have enough RAM or VRAM to store all the rotations available, or who wish to have a lower number of rotations viewable for a more "retro" feel. Implementations must be able to parse te entire file and gracefully downgrade the number of rotations to the user's setting.

## Handling Order

The rotations of a sprite (`----`) and frame (`#`) combination should be handled by, in order of priority:
1. The lump `----#ROT`.
2. The lump `----0ROT`.
3. One of the following strings of rotations, in order of priority, assuming that all required lumps exist:
	* `192A3B4C5D6E7F8G`
	* `12345678`
	* `1357`
	* `15`
	* `0`
4. An "invalid frames" warning or error must be thrown in the way and at the severity level deemed appropriate by the port.

## Notes

* The first rotation represents front of the object. Every following rotation represents a rotation of the object by 360 degrees divided by the number of rotations in the file.
* There is no restriction that the number of rotations be a power of 2.
* When considering frames, `\` and `^` **must** be treated as interchangable.
* The sprite name `TNT1` will always be interpreted as a valid, invisible sprite.
* Formats:
	* Format `0x00` is designed for saving space in more traditional source-ports, and supports a maximum of `0xFF` (255) rotations.
	* Format `0x01` is designed for flexibility, and supports a maximum of `0xFFFFFFFF` (4294967295) rotations.
	* Format `0xFF` is reserved.

## Parsing

In order to parse the rotation lump:
* Struct `Rotation` is composed of the fields `sprite` (an array of 4 characters) and `rot` (a single character).
* Variable `sprite` (an array of 4 characters) is defaulted to the lump's sprite.
* Variable `rot` (a single character) is defaulted to `0`.
* Variable `rots` (a 32-bit integer) is defaulted to `0`.
* Variable `format` (a byte) is defaulted to `0`.
* Variable `finalRots` is an array of `Rotation` structs.
* Variable `i` (a 32-bit integer) is a temporary variable used for indexing.

* Set variable `format` to the value of the next byte.
* If `format` is equal to `0x00`:
	* Set variable `rots` to the value of the next byte.
	* Until you reach the end-of-file:
		* Set variable `rot` to the value of the next character.
		* Set `finalRots`[`i`] to a newly created `Rotation` struct with the values (`sprite`, `rot`).
		* Increment `i` by `1`.
* Else, if `format` is equal to `0x01`:
	* Set variable `rots` to the value of the next four bytes, interpreted as a big-endian value.
	* Until you reach the end-of-file:
		* Set variable `rot` to the value of the next character.
		* If `rot` equals `0x00`:
			* Set variable `sprite` to the value of the next four characters.
			* Set variable `rot` to the value of the next character.
		* Set `finalRots`[`i`] to a newly created `Rotation` struct with the values (`sprite`, `rot`).
		* Increment `i` by `1`.
* Else:
	* An "invalid format" warning or error must be thrown in the way and at the severity level deemed appropriate by the port.
