# UTF-8

## Description

| Number of bytes | Bits for code point | First code point | Last code point | Byte 1   | Byte 2   | Byte 3   | Byte 4   |
| --------------- | ------------------- | ---------------- | --------------- | -------- | -------- | -------- | -------- |
| 1               | 7                   | U+0000           | U+007F          | 0xxxxxxx |
| 2               | 11                  | U+0080           | U+07FF          | 110xxxxx | 10xxxxxx |
| 3               | 16                  | U+0800           | U+FFFF          | 1110xxxx | 10xxxxxx | 10xxxxxx |
| 4               | 21                  | U+10000          | U+10FFFF        | 11110xxx | 10xxxxxx | 10xxxxxx | 10xxxxxx |

Consider the encoding of the Euro sign, €.

1. The Unicode code point for "€" is `U+20AC`.
2. According to the scheme table above, this will take three bytes to encode, since it is between `U+0800` and `U+FFFF`.
3. Hexadecimal `20AC` is binary `0010 0000 1010 1100`. The two leading zeros are added because, as the scheme table shows, a three-byte encoding needs exactly sixteen bits from the code point.
4. Because the encoding will be three bytes long, its leading byte starts with three 1s, then a 0 (`1110...`)
5. The four most significant bits of the code point are stored in the remaining low order four bits of this byte (`1110 0010`), leaving 12 bits of the code point yet to be encoded (`...0000 1010 1100`).
6. All continuation bytes contain exactly six bits from the code point. So the next six bits of the code point are stored in the low order six bits of the next byte, and 10 is stored in the high order two bits to mark it as a continuation byte (so `1000 0010`).
7. Finally the last six bits of the code point are stored in the low order six bits of the final byte, and again 10 is stored in the high order two bits (1010 1100).

The three bytes `1110 0010 1000 0010 1010 1100` can be more concisely written in hexadecimal, as `E2 82 AC`.

The following table summarises this conversion, as well as others with different lengths in UTF-8. The colors indicate how bits from the code point are distributed among the UTF-8 bytes. Additional bits added by the UTF-8 encoding process are shown in black.

- [Example](index.html)

## The Difference Between Unicode and UTF-8

Unicode is a **character set**. UTF-8 is **encoding**.

Unicode is a list of characters with unique decimal numbers (code points). `A = 65, B = 66, C = 67, ....`

This list of decimal numbers represent the string "hello": `104 101 108 108 111`

Encoding is how these numbers are translated into binary numbers to be stored in a computer:

UTF-8 encoding will store "hello" like this (binary): `01101000 01100101 01101100 01101100 01101111`

> **Encoding** translates numbers into binary. **Character sets** translates characters to numbers.

- [Character encodings in HTML](html.md)
- [Character encodings in MySQL](mysql.md)
