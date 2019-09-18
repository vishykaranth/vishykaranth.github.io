# Strings

## Char Sets

- Character sets translate characters to numbers.

### ASCII

- *American Standard Code for Information Interchange*
- Encodes 128 characters in 7 bits.
- Encoded are numbers 0 to 9, lowercase letters a to z, uppercase letters A to Z, basic punctuation symbols, control codes and space.
- *ANSI* standard – different "code pages" for characters 128-255 (the 1 extra bit) which differ between countries and languages.

### Unicode

- Character set for most of the world's writing systems.
- List of characters with unique numbers (code points).
- There are more than 120,000 characters covering 129 "scripts" (a collection of letters), there's no limit on number of letters.
- Letters map to code points.
- Every letter in every alphabet is assigned a number, for example the letter `A` = `41` (`U+0041`); the number is *hexadecimal*.
- For example, the list of numbers represent the string "hello": `104` `101` `108` `108` `111`.
- There are more than 65,526 (2^16) chars, so not every Unicode letter can be represented by two Bytes.
- Unicode character in Java: `\u00fc`
    - `String s = "\u00fc";`

## Encoding

- An encoding is a way to translate between Strings and Bytes.
- Encoding is how these numbers are translated into binary numbers to be stored on disk or in memory (Encoding translates numbers into binary).
- It doesn't make sense to have a string without knowing what encoding it uses!

### UTF-8

- UTF-8 is a transmission format for Unicode, i.e., *encoding*.
- Capable of encoding all 1,112,064 possible characters (code points) in Unicode.
- Variable-length, code points are encoded with 8-bit code units.
- Every code point from 0-127 is stored in a *single Byte*.
- Code points 128 and above are stored using 2, 3, or 4 Bytes.
- English text looks exactly the same in UTF-8 as it did in ASCII.
- ASCII text is valid UTF-8-encoded Unicode.
- `byte[]` however has an encoding.
- To convert a string object to UTF-8, invoke the `getBytes(Charset charset)` on the string with UTF-8.
- 84.6% of all Web pages use UTF-8.
- Java `String` uses UTF-16 encoding internally.
- For example, UTF-8 encoding will store "hello" like this (binary): `01101000` `01100101` `01101100` `01101100` `01101111`

### UTF-16

- Capable of encoding all 1,112,064 possible characters in Unicode.
- Variable-length, code points are encoded with one or two 16-bit code units.
- The `String` class in Java uses UTF-16 encoding internally and can't be modified.

### Punycode

- A way to represent Unicode with the limited character subset of ASCII supported by DNS.
- For example: "bücher" => "bcher-kva"

### Communicating Encoding

- Email:
    - `Content-Type: text/plain; charset="UTF-8"` header in the beginning of the message.
- Web page:
    - `<meta http-equiv="Content-Type" content="text/html; charset=utf-8">` meta tag, has to be the very first thing in the `<head>`.
    - As soon as the web browser sees this tag it's going to stop parsing the page and start over after reinterpreting the whole page using the encoding specified.
    - Can also use the `Content-Type` header like in email, but the `<meta>` tag is preferable.
