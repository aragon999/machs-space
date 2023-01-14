---
title: "Whats the Max Valid Length (in bytes) of an Emoji?"
date: 2023-01-14T01:47:20+01:00
math: true
draft: true
---

## Introduction

Recently I stumbled upon a [Twitter conversation](https://twitter.com/fractaledmind/status/1609618168946778113)

> If I want to store a single emoji in a Postgres field, what is the LIMIT I should set on the TEXT field?

which talked about storing a single emoji in a Postgres `TEXT`-field, with a `LIMIT`.

While in this blog post I do not want to discuss the performance implications, of changing the database field, I find the question of how much bytes a single emoji can have very interesting, as there are several aspects to it. So this blog post will be about character encoding, the Unicode standard and emojis. Such that at the end we can hopefully state and understand the answer to the question "Whats the Max Valid Length (in bytes) of an Emoji?".

The answer of this question is already stated in the [Twitter conversation](https://twitter.com/andrewculver/status/1609666366763855872), which is 35 `UTF-8` bytes for the Unicode 15 standard.

## Character encoding

To first clear up some definitions and speak about some backgrounds, we will start with [character encoding](https://en.wikipedia.org/wiki/Character_encoding) in general.

Let us start from the smallest unit which can be used to represent data, [the bit](https://en.wikipedia.org/wiki/Bit). A bit can have two values, which are usually labeled $0$ and $1$. These two states are realized on hardware which can have two distinct states, that depending on the use-case, can be easily changed, read or both. For example [HDDs use the electron spin](https://davidabergel.wordpress.com/2016/02/06/hard-disk/), [flash memory](https://en.wikipedia.org/wiki/Flash_memory) uses special transistors, called [metal–oxide–semiconductor field-effect transistor (MOSFET)](https://en.wikipedia.org/wiki/MOSFET) and so on.

As this representation is good for hardware, but not so good to really work with data from software perspective, another unit has been introduced, the [byte](https://en.wikipedia.org/wiki/Byte), in general, consisting of one or more bits. Nowadays, when we speak of a byte without the specification of the number of bits, we usually mean a byte consisting of $2^3 = 8$ bits, also sometimes referred to as "octet".

### Unicode encoding

For the rest of this article we will focus on [Unicode encoding](http://unicode.org/main.html), which is for the Linux world and the web, the common used encoding of text, and which we need to further understand to answer the initial question.

The [Unicode encoding model](https://en.wikipedia.org/wiki/Character_encoding#Unicode_encoding_model), consists of two separate parts. One part defines how characters map to so called [code points](https://en.wikipedia.org/wiki/Code_point) and the other part defines how these code points map to bits.

#### Code points

An Unicode code point, denoted by `U+<hex number>`, can be translated using the [Unicode character code mapping](https://www.unicode.org/charts/), where `<hex number>` is a [hexadecimal number](https://en.wikipedia.org/wiki/Hexadecimal), from $0000\_{\mathrm{hex}}$ to $10\mathrm{FFFF}\_{\mathrm{hex}}$. In general one would omit the leading zeros, e.g. $0000 = 0$, but in the Unicode standard, the digits have different meanings depending on the position.

In Unicode the hex numbers have the following format: $[0-10]\_{\mathrm{hex}} [000 - \mathrm{FFFF}]\_{\mathrm{hex}}$. Where the leading group, with $17$ possible values $[0 - 10]\_\mathrm{hex} = [0 - 16]\_\mathrm{dec}$, specifies the so called [Unicode plane](https://en.wikipedia.org/wiki/Plane_(Unicode)). The second group specifies the code point inside the specified plane, which allows to specify $2^{16} = 65\\,536$ unique values, $[000 - \mathrm{FFFF}]\_\mathrm{hex} = [0 - 65\\,535]\_\mathrm{dec}$. Note that for values below three digits one explicitly adds leading zeros, such that the number of digits is at least three, as there are otherwise collisions of the Unicode code points. So in total there exist $17*2^{16} = 1\\,114\\,112$ available code points in the Unicode standard, from which the [Unicode 15](https://www.unicode.org/versions/Unicode15.0.0/) standard currently defines $149\\,186$. For example [`U+0041` represents the character `A` and `U+0061` stands for `a`](https://www.unicode.org/charts/PDF/U0000.pdf).

Using `U+0000-10FFFF` or `U+0000..U+10FFFF` we denote a range of Unicode code points, in the example it is the full Unicode range. When we want to write a sequence we write `U+0041 U+0042 U+0043` (ABC), where the space between the Unicode code points is just for readability.

The Unicode standard furthermore defines so-called [Unicode blocks](https://en.wikipedia.org/wiki/Unicode_block), without a fixed size, which are commonly used together. For a list refer to [Wikipedia](https://en.wikipedia.org/wiki/Unicode_block#List_of_blocks) or the [official specification](https://www.unicode.org/charts/About.html#Blocks).

#### Storing code points as bits

The Unicode code points are so to say an intermediate representation of characters and in general other symbols, in hexadecimal values. But as initially stated, these values need to be further translated to ones and zeros, such that the computer can process this data.

To solve this, the Unicode standard defines [different mapping methods](https://en.wikipedia.org/wiki/Unicode#UTF), where in this blog post we will only discuss the [Unicode Transformation Format (UTF) encoding](https://en.wikipedia.org/wiki/Unicode#Mapping_and_encodings). More specifically the Unicode standard defines `UTF-{8,16,32}`, where for each of those standards the number defines how many bits one code unit should have. The code unit is the smallest possible number of bits which can be allocated together. Note that for `UTF-8` the term code unit corresponds to the standard 8-bit byte.

In order to encode the Unicode range at maximum 32 bits are needed, which means that for example for `UTF-8` at maximum $32 / 8 = 4$ bytes are required. For `UTF-16` at maximum two code units (2 bytes) are needed and for `UTF-32` always one code unit (4 bytes) is needed, to encode the whole Unicode alphabet.

Additionally, `UTF-8` is binary compatible to, the prior to Unicode, common [American Standard Code for Information Interchange (ASCII)](https://en.wikipedia.org/wiki/ASCII) encoding, which requires at maximum 7-bits to encode the full ASCII range, which includes all [basic latin](https://en.wikipedia.org/wiki/Basic_Latin_(Unicode_block)) characters. This is not true for `UTF-{16,32}`.

Although `UTF-32` would be the easiest format to implement, since each "bit-combination" corresponds to one Unicode code-point, the advantages of for example `UTF-8` is that for values which require less than four bytes to be encoded, only this amount of bits needs to be allocated. Hence `UTF-{8,16}` are also referred to as " variable-length character encoding".

On the web the [most used encoding is `UTF-8`](https://w3techs.com/technologies/cross/character_encoding/ranking), which has the potentially the largest savings in required bytes. In particular since, all HTML Tags are made up only of ASCII characters, which can be encoded using a single byte. Furthermore, most of the network protocols are designed to work with packages of 8-bit.

Nowadays `UTF-32` is mostly [used for internal representation](https://en.wikipedia.org/wiki/UTF-32#Use) of text, to simplify working on this data. And `UTF-16` is used, for example, in the [internal Windows API for text (up to Windows 10)](https://en.wikipedia.org/wiki/UTF-16#Usage), since although it requires more bits it has some performance benefits. But these performance benefits would be neglectable in comparison to the additional required transfer time, when the data has to be sent over the network.

As data stored in databases, mostly has to be transferred over the network, storing it in a different format than `UTF-8` would mean reencoding it and `UTF-8` potentially has the most savings in required space. For example Postgres [only supports `UTF-8`](https://www.postgresql.org/docs/current/multibyte.html#MULTIBYTE-CHARSET-SUPPORTED) for Unicode encoding, in contrast to for example [MySQL](https://dev.mysql.com/doc/refman/8.0/en/charset-unicode-sets.html). As the initial question was regarding Postgres, we will for the rest of this blog post focus on `UTF-8`.

## Emoji encoding in Unicode

In order to answer the initial question, we will also need to discuss how emojis are handled in Unicode. The Unicode standard has a [technical report dedicated for how emojis are encoded](https://www.unicode.org/reports/tr51/index.html). I will summarize the most important features of how emojis are build in Unicode, but there are of course also other blog posts, [in particular](https://tonsky.me/blog/emoji/), which discusses this topic in more technical detail.

A little side note, as the Unicode technical standard states, the term emoji comes from Japanese, where `e` stands for "picture" and `moji` for "written character". For more details on the history of emojis, refer to [Wikipedia](https://en.wikipedia.org/wiki/Emoji#History).

The simplest way of encoding an emoji is just by a single code point in the Unicode alphabet, as it is for example for the [Emoticons Unicode block](https://codepoints.net/emoticons). But there are several mechanisms, which are also used elsewhere in the Unicode standard, to compose emoji from multiple Unicode characters. Such a combined character is also called ["grapheme cluster"](ode.org/reports/tr29/#Grapheme_Cluster_Boundaries) and is also used for encoding other characters. For example the ["Combining Diacritical Marks"](https://codepoints.net/combining_diacritical_marks) are used in combination with normal characters to create characters like "ü".

There are several ways how one can compose emojis in Unicode. Three of those will be discussed in the following subsections, for a full list refer to the Unicode technical report.

### Emoji presentation sequence

Before emojis were defined in the Unicode standard there already existed symbols, for example in the [Dingbats Unicode block](https://codepoints.net/dingbats). In order to make use of those code points, and also use them as fallback, if no emoji was found, one way to compose an emoji is using the ["emoji variation sequences"](https://www.unicode.org/reports/tr51/index.html#Emoji_Variation_Sequences). For example the heart `U+2764` (❤) can also be displayed as emoji `U+2764 U+FE0F` (❤️) using the "emoji presentation selector" `U+FE0F`. It also works the other way around using the "text presentation selector" `U+FE0E`, e.g. `U+1F3AD` (🎭) and the text version `U+1F3AD U+FE0E` (🎭︎).

For a list of all defined emoji variation sequences refer to [this list](https://www.unicode.org/Public/15.0.0/ucd/emoji/emoji-variation-sequences.txt).

### Emoji modifier

Additionally, one can use ["emoji modifiers"](https://www.unicode.org/reports/tr51/index.html#Emoji_Modifiers) to compose emojis. For [Unicode 15 the only used modifiers](https://unicode.org/emoji/charts/full-emoji-modifiers.html) are the [skin tone modifiers](http://www.unicode.org/reports/tr51/index.html#Emoji_Modifiers_Table) defined as the Unicode code points `U+1F3FB-1F3FF`, which "modify" the preceding emoji. Of course not all "base emoji" support a skin color modification (e.g. a plane has no skin color).

!!!!! TODO: Rewrite section
If one does not want those two code points to be joined into a single glyph, one can separate the two code points by a ["zero-width space"](https://en.wikipedia.org/wiki/Zero-width_space) `U+200B`, i.e. `U+1F44B U+1F3FE` (👋🏾) vs. `U+1F44B U+200B U+1F3FE` (👋​🏾).

### Emoji sequences

The final method which we will discuss are ["emoji sequences"](https://www.unicode.org/reports/tr51/index.html#Emoji_Sequences). They are very similar to the emoji modifiers, but in this case they explicitly need to be joined by another Unicode code point. In general, one emoji sequence can contain multiple joiners.

For example the black cat is not created using a modifier, but rather  `U+1F408 U+200D U+2B1B` (🐈‍⬛). This sequence is called an ["emoji ZWJ sequence"](https://www.unicode.org/reports/tr51/index.html#def_emoji_zwj_sequence), since it contains the "zero width joiner" (ZWJ) `U+200D` character. Without this character the two remaining Unicode code points `U+1F408 U+2B1B` (🐈⬛) are displayed as separate glyphs.

Of course there can also be sequences built from emoji presentation sequences, for example the "eye in a speech bubble" `U+1F441 U+FE0F U+200D U+1F5E8 U+FE0F` (👁️‍🗨️).

The full list of sequences containing the ZWJ character can be found in [this list](https://www.unicode.org/Public/emoji/15.0/emoji-zwj-sequences.txt).

## Combining it all

Now we should have everything together, for answering the initial question. First, as already found in the Twitter thread, the longest sequence is when two people are kissing and both persons have skin tone modifiers, e.g. the first such sequence in the [sequence list for Unicode 15](https://www.unicode.org/Public/emoji/15.0/emoji-zwj-sequences.txt) is "kiss: man, man, light skin tone" `U+1F468 U+1F3FB U+200D U+2764 U+FE0F U+200D U+1F48B U+200D U+1F468 U+1F3FB` (👨🏻‍❤️‍💋‍👨🏻).

Let us quickly go through this emoji. The first and last two Unicode code points are "man with light skin tone" `U+1F468 U+1F3FB` (👨🏻). These two emojis are joined using the ZWJ `U+200D`, to another combination of two emojis, again joined by the ZWJ, namely a heart `U+2764 U+FE0F` (❤️) and a kiss `U+1F48B` (💋). Note that `U+2764 U+FE0F U+200D U+1F48B` (❤️‍💋) alone is not combined into a single character.

As we know, that each of those symbols, can be composed of one to four bytes, the easiest way is to just count the bytes of the `UTF-8`-encoded string. Although we could read it from the corresponding plane, in which each of those characters is in, but this would require to look at `UTF-8` encoding in a more technical way.

That being said, the byte sequence of the emoji 👨🏻‍❤️‍💋‍👨🏻, generated using [`xd`](https://lib.rs/crates/xd), and annotated by the corresponding Unicode code points is the following:
```
F0 9F 91 A8     = U+1F468 (👨)
F0 9F 8F BB     = U+1F3FB (🏻)
E2 80 8D        = U+200D (ZWJ)
E2 9D A4        = U+2764 (❤)
EF B8 8F        = U+FE0F (emoji presentation sequence)
E2 80 8D        = U+200D (ZWJ)
F0 9F 92 8B     = U+1F48B (💋)
E2 80 8D        = U+200D (ZWJ)
F0 9F 91 A8     = U+1F468 (👨)
F0 9F 8F BB     = U+1F3FB (🏻)
```

Indeed when we count the bytes (two characters are one byte), we find that in total we need 35 bytes to encode this emoji. But we have to keep in mind, that this is only the maximum needed bytes for the Unicode 15 standard. With every new standard there could be new sequences which need more bytes to be encoded.

## Conclusion

While the answer to the initial question (35 bytes) was already known before, I took the chance to learn more about the technical details of what the Unicode standard defines.

In general if I would need to define such a limit on a database text field, I probably would leave some space for the future, e.g. $8\*3 + 10\*4 = 64$ bytes, such that I do not need to adjust the limit for future Unicode standards (which I would probably forget).
