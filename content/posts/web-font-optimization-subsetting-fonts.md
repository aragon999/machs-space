---
title: "Web Font Optimization - Subsetting Fonts"
date: 2023-01-05T01:20:09+01:00
draft: false
---

## Introduction

In the [last part]({{< ref "/posts/web-font-optimization-variable-font" >}}) of the series, we spoke about variable fonts and how we can extract only parts of the axes. Using that approach we already could reduce the transferred font data from 315.51kb to 164.95kb if no italic font was used. However, if we would include the [Googlefonts Inter CSS file](https://fonts.googleapis.com/css2?family=Inter:wght@400..700&display=swap), only 38.86kb of font data is downloaded (for the default Shopware demo home page). Note that Googlefonts currently does not provide the italic version of Inter.

If one looks at the source of the Googlefonts CSS file, one notices that there are different font files, for different [Unicode blocks](https://en.wikipedia.org/wiki/Unicode_block) specified. In this last part of the series, after a short introduction, we will again use `fonttools` to extract the different subsets of glyphs from the font files.

## Short review of Unicode fonts

Before coming back to fonts, we should briefly speak about Unicode, and how text is stored and displayed. Let's start with the [definition from Wikipedia](https://en.wikipedia.org/wiki/Unicode)

> Unicode, formally The Unicode Standard, is an information technology standard for the consistent encoding, representation, and handling of text expressed in most of the world's writing systems. The standard, which is maintained by the Unicode Consortium, defines as of the current version (15.0) 149,186 characters covering 161 modern and historic scripts, as well as symbols, emoji (including in colors), and non-visual control and formatting codes.

In other words, Unicode is a standard, which defines so-called code points, ranged from `U+000000-10FFFF`, which map bytes stored in memory to characters, symbols, etc. There exist different, so-called encodings, which map raw bytes to the code point of Unicode, [for example](https://en.wikipedia.org/wiki/Comparison_of_Unicode_encodings#Eight-bit_environments), Unicode Transformation Format - 8-bit, commonly also known as UTF-8. UTF-8 for example uses at maximum [four bytes (consisting of 8 bits) to map the whole Unicode range](https://en.wikipedia.org/wiki/UTF-8#Encoding).

### Unicode font mapping

As learned in the [first part]({{< ref "/posts/web-font-optimization-introduction" >}}) of this series, fonts in general are built up from glyphs. Most modern font specifications, such as the OpenType and TrueType, use Unicode mapping, i.e. each Unicode code point maps to a glyph defined in the font file. Not every glyph needs to exist in the font file, which would then result in some other symbol, indicating that this glyph is missing.

From now on we will mostly focus on OpenType fonts and explicitly all code examples are only valid for OpenType fonts (`woff{,2}` should be also supported). For OpenType fonts, the mapping is defined in the [Character to Glyph Index Mapping (`cmap`) Table](https://learn.microsoft.com/en-us/typography/opentype/otspec180/cmap). Using the following small Python script, one can list all available Unicode characters and their corresponding glyph names of an OpenType font file for the Windows platform, with `platformID == 3` (usage `./get_font_glyphs.py <font.ttf>`):

```python
#!/usr/bin/env python3

import sys
from fontTools import ttLib

font = ttLib.TTFont(sys.argv[1])

for t in font['cmap'].tables:
    if t.platformID != 3:
        continue

    for c, m in font['cmap'].tables[0].cmap.items():
        print(f'{hex(c)}: {m}')
```

### Font layout features

But this would be not sufficient, as in typography one may want to use different glyphs, depending on the context of the text. For some examples refer to the [Inter features section](https://rsms.me/inter/#features), showing the different [OpenType Layout tags](https://en.wikipedia.org/wiki/OpenType#Layout_tags) which are available for this font. We will not go deeper in how this exactly works, but the following Python script shows all available feature tags, as defined in the [Glyph Positioning Table (`GPOS`)](https://learn.microsoft.com/en-us/typography/opentype/spec/gpos) and [Glyph Substitution Table (`GSUB`)](https://learn.microsoft.com/en-us/typography/opentype/spec/gsub) (usage `./get_font_features.py <font.ttf>`):

```python
#!/usr/bin/env python3

import sys
from fontTools import ttLib

font = ttLib.TTFont(sys.argv[1])

print('GSUB')
for f in font['GSUB'].table.FeatureList.FeatureRecord:
    print(f'  {f.FeatureTag}')

print('\nGPOS')
for f in font['GPOS'].table.FeatureList.FeatureRecord:
    print(f'  {f.FeatureTag}')
```

## Subsetting fonts

When we look at the [Unicode alphabet](https://jrgraphix.net/r/Unicode/), i.e., all Unicode characters, we notice that for a particular language, probably only a very limited subset of glyphs of a font is used. The specific glyphs depend on the language and content of the page. In particular, the Unicode standard is structured in Unicode blocks, [where each block](https://en.wikipedia.org/wiki/Unicode_block#List_of_blocks) is responsible for one or more languages or application areas.

The idea is now to create fonts, with only a subset of glyphs, that are probably used together. Using the [`unicode-range` `@font-face` descriptor](https://developer.mozilla.org/en-US/docs/Web/CSS/@font-face/unicode-range) we can than specify the Unicode range which this `@font-face`-declaration can be used for. As the mdn web docs state when no characters of that range are used, the specified font file is not downloaded.

### Creating subset font files

When we look at the definition of the [Inter Googlefonts `@font-face` declarations](https://fonts.googleapis.com/css2?family=Inter:wght@400..700&display=swap), we see the following supported "charsets" `latin`, `latin-ext`, `vietnamese`, `greek`, `greek-ext`, `cyrillic` and `cyrillic-ext`. For example, the `latin` range is specified by the following ranges:
Additionally, the following font features for the `latin` font of the Inter Googlefont are available:
```
GSUB
  calt
  dnom
  frac
  locl
  numr
  pnum
  tnum

GPOS
  kern
```

We will use the variable fonts, created in the previous part of this series, with a restricted weight range, from 400-700, for the following examples. Using the [subset module of `fonttools`](https://fonttools.readthedocs.io/en/latest/subset/index.html) (`pyftsubset` or `fonttools subset`), we can create a font with similar features as shipped by Googlefont:
```bash
$ pyftsubset Inter-roman.var.ttf --unicodes='U+0000-00FF, U+0131, U+0152-0153, U+02BB-02BC, U+02C6, U+02DA, U+02DC, U+2000-206F, U+2074, U+20AC, U+2122, U+2191, U+2193, U+2212, U+2215, U+FEFF, U+FFFD' \
    --output-file=Inter-roman.latin.var.ttf \
    --drop-tables= --recalc-bounds --recalc-average-width  --name-IDs='*' --name-legacy --glyph-names \
    --layout-features="calt,dnom,frac,locl,numr,pnum,tnum,cv02,cv03,cv04,kern"
```
Where I also included the additional [font features, which are used in Shopware](https://github.com/shopware/platform/blob/66f63d2eb69faef04412908668a257cc05722e7a/src/Storefront/Resources/app/storefront/src/scss/skin/shopware/vendor/_inter-fontface.scss#L65) (`cv02`, `cv03` and `cv04`).

When we diff the containing glyphs, using the `get_font_glyphs.py` scripts from before, where for readability I additionally removed the new lines and only show the added Unicode values,
```bash
$ diff -u <(./get_font_glyphs.py Inter-Googlefonts.latin.woff2 | cut -d':' -f1) <(./get_font_glyphs.py Inter-roman.latin.var.ttf | cut -d':' -f1)|tail -n +4|grep '^+'|tr '\n' ' '|tr -d '+'
0x2000 0x2001 0x2003 0x2004 0x2005 0x2006 0x2007 0x2008 0x200a 0x2010 0x2011 0x2012 0x2015 0x2016 0x2017 0x201b 0x201f 0x2020 0x2021 0x2023 0x2024 0x2025 0x2027 0x202f 0x2030 0x2031 0x2034 0x2035 0x2036 0x2037 0x2038 0x203b 0x203c 0x203d 0x203e 0x203f 0x2040 0x2041 0x2042 0x2043 0x2045 0x2046 0x2047 0x2048 0x2049 0x204a 0x204b 0x204c 0x204d 0x204e 0x204f 0x2050 0x2051 0x2052 0x2053 0x2054 0x2055 0x2056 0x2057 0x2058 0x2059 0x205a 0x205b 0x205c 0x205d 0x205e 0x205f
```
we see that Google additionally removed several glyphs of the [General Punctuation Unicode block](https://en.wikipedia.org/wiki/General_Punctuation).

I do not know how Google decides which glyphs should be included for what range, but I guess that this is based on some analysis of texts. For those who are interested, using the following command, with the adapted Unicode range, we can create the subset file, containing the same glyphs as the Googlefont file for `latin`:
```bash
$ pyftsubset Inter-roman.var.ttf --unicodes='U+0000-00FF, U+0131, U+0152-0153, U+02BB-02BC, U+02C6, U+02DA, U+02DC, U+2002, U+2009, U+200b U+2013, U+2014, U+2018-201A, U+201C-201E, U+2022, U+2026, U+2032, U+2033, U+2039, U+203a, U+2044, U+2074, U+20AC, U+2122, U+2191, U+2193, U+2212, U+2215, U+FEFF, U+FFFD' \
    --output-file=Inter-roman.latin-google.var.ttf \
    --drop-tables= --recalc-bounds --recalc-average-width  --name-IDs='*' --name-legacy --glyph-names \
    --layout-features="calt,dnom,frac,locl,numr,pnum,tnum,kern"
```

When we compare the file sizes
```
 89k Inter-roman.latin.var.ttf
 33k Inter-roman.latin.var.woff2

 79k Inter-roman.latin-google.var.ttf
 29k Inter-roman.latin-google.var.woff2

473k Inter-roman.var.ttf
165k Inter-roman.var.woff2
```
we find that the additional optimization of the "Google glyph set" reduces the `woff2` file size by 4kb. But in comparison, the subsetted font with 32kb, is much smaller than the font containing all glyphs with a file size of 165kb. So I will leave it to the reader if they wish to remove further glyphs from their font files, or not.

For completeness I list also the commands for creating the other font subsets:
```bash
$ pyftsubset Inter-roman.var.ttf --unicodes='U+0100-024F, U+0259, U+1E00-1EFF, U+2020, U+20A0-20AB, U+20AD-20CF, U+2113, U+2C60-2C7F, U+A720-A7FF' \
    --output-file=Inter-roman.latin-ext.var.ttf \
    --drop-tables= --recalc-bounds --recalc-average-width  --name-IDs='*' --name-legacy --glyph-names \
    --layout-features="locl,kern"
$ pyftsubset Inter-roman.var.ttf --unicodes='U+0102-0103, U+0110-0111, U+0128-0129, U+0168-0169, U+01A0-01A1, U+01AF-01B0, U+1EA0-1EF9, U+20AB' \
    --output-file=Inter-roman.vietnamese.var.ttf \
    --drop-tables= --recalc-bounds --recalc-average-width  --name-IDs='*' --name-legacy --glyph-names \
    --layout-features="kern"
$ pyftsubset Inter-roman.var.ttf --unicodes='U+0370-03FF' \
    --output-file=Inter-roman.greek.var.ttf \
    --drop-tables= --recalc-bounds --recalc-average-width  --name-IDs='*' --name-legacy --glyph-names \
    --layout-features="kern"
$ pyftsubset Inter-roman.var.ttf --unicodes='U+1F00-1FFF' \
    --output-file=Inter-roman.greek-ext.var.ttf \
    --drop-tables= --recalc-bounds --recalc-average-width  --name-IDs='*' --name-legacy --glyph-names \
    --layout-features="kern"
$ pyftsubset Inter-roman.var.ttf --unicodes='U+0301, U+0400-045F, U+0490-0491, U+04B0-04B1, U+2116' \
    --output-file=Inter-roman.cyrillic.var.ttf \
    --drop-tables= --recalc-bounds --recalc-average-width  --name-IDs='*' --name-legacy --glyph-names \
    --layout-features="ccmp,kern"
$ pyftsubset Inter-roman.var.ttf --unicodes='U+0460-052F, U+1C80-1C88, U+20B4, U+2DE0-2DFF, U+A640-A69F, U+FE2E-FE2F' \
    --output-file=Inter-roman.cyrillic-ext.var.ttf \
    --drop-tables= --recalc-bounds --recalc-average-width  --name-IDs='*' --name-legacy --glyph-names \
    --layout-features="kern"
```

Comparing the file sizes, of the subset font files, with the ones shipped by Googlefonts
```
38k Inter-Googlefonts.latin.woff2
32k Inter-roman.latin.var.woff2

57k Inter-Googlefonts.latin-ext.woff2
43k Inter-roman.latin-ext.var.woff2

27k Inter-Googlefonts.cyrillic-ext.woff2
19k Inter-roman.cyrillic-ext.var.woff2

17k Inter-Googlefonts.cyrillic.woff2
12k Inter-roman.cyrillic.var.woff2

12k Inter-Googlefonts.greek-ext.woff2
9k Inter-roman.greek-ext.var.woff2

22k Inter-Googlefonts.greek.woff2
16k Inter-roman.greek.var.woff2

9k Inter-Googlefonts.vietnamese.woff2
7k Inter-roman.vietnamese.var.woff2
```
we find that our created font files, with the restricted `wght`-axis, are still smaller than the fonts from Google, although our font files, potentially contain more glyphs.

I did not go through the effort to also restrict the `wght`-axis of the Googlefonts and compare the file sizes. I did notice that some metadata is different for these fonts, so it would be something interesting for further investigation, but the gain will probably be very little.

We use similar commands for the italic version of the Inter font, e.g.,
```bash
$ pyftsubset Inter-italic.var.ttf --unicodes='U+0000-00FF, U+0131, U+0152-0153, U+02BB-02BC, U+02C6, U+02DA, U+02DC, U+2000-206F, U+2074, U+20AC, U+2122, U+2191, U+2193, U+2212, U+2215, U+FEFF, U+FFFD' \
    --output-file=Inter-italic.latin.var.ttf \
    --drop-tables= --recalc-bounds --recalc-average-width  --name-IDs='*' --name-legacy --glyph-names \
    --layout-features="calt,dnom,frac,locl,numr,pnum,tnum,cv02,cv03,cv04,kern"
```
after that we should have all 14 relevant font files at our hand.

Unfortunately, I do not know of a general, easy way, of how such ranges can be determined. One should probably look at the features of the upstream font, and what glyphs and features are needed on the webpage.

### Including the subset fonts on the website

The hard part should be now done, and the final step is easy. We just need to include the generated font files in the CSS. For Shopware the CSS for Inter is

```css
@supports (font-variation-settings: normal) {
    @font-face {
        font-family: 'Inter';
        font-style: normal;
        font-weight: 400 700;
        font-display: fallback;
        src: url('#{$app-css-relative-asset-path}/font/Inter-roman.cyrillic-ext.var.woff2') format('woff2 supports variations'),
            url('#{$app-css-relative-asset-path}/font/Inter-roman.cyrillic-ext.var.woff2') format('woff2-variations');
        unicode-range: U+0460-052F, U+1C80-1C88, U+20B4, U+2DE0-2DFF, U+A640-A69F, U+FE2E-FE2F;
    }

    @font-face {
        font-family: 'Inter';
        font-style: normal;
        font-weight: 400 700;
        font-display: fallback;
        src: url('#{$app-css-relative-asset-path}/font/Inter-roman.cyrillic.var.woff2') format('woff2 supports variations'),
            url('#{$app-css-relative-asset-path}/font/Inter-roman.cyrillic.var.woff2') format('woff2-variations');
        unicode-range: U+0301, U+0400-045F, U+0490-0491, U+04B0-04B1, U+2116;
    }

    @font-face {
        font-family: 'Inter';
        font-style: normal;
        font-weight: 400 700;
        font-display: fallback;
        src: url('#{$app-css-relative-asset-path}/font/Inter-roman.greek-ext.var.woff2') format('woff2 supports variations'),
            url('#{$app-css-relative-asset-path}/font/Inter-roman.greek-ext.var.woff2') format('woff2-variations');
        unicode-range: U+1F00-1FFF;
    }

    @font-face {
        font-family: 'Inter';
        font-style: normal;
        font-weight: 400 700;
        font-display: fallback;
        src: url('#{$app-css-relative-asset-path}/font/Inter-roman.greek.var.woff2') format('woff2 supports variations'),
            url('#{$app-css-relative-asset-path}/font/Inter-roman.greek.var.woff2') format('woff2-variations');
        unicode-range: U+0370-03FF;
    }

    @font-face {
        font-family: 'Inter';
        font-style: normal;
        font-weight: 400 700;
        font-display: fallback;
        src: url('#{$app-css-relative-asset-path}/font/Inter-roman.vietnamese.var.woff2') format('woff2 supports variations'),
            url('#{$app-css-relative-asset-path}/font/Inter-roman.vietnamese.var.woff2') format('woff2-variations');
        unicode-range: U+0102-0103, U+0110-0111, U+0128-0129, U+0168-0169, U+01A0-01A1, U+01AF-01B0, U+1EA0-1EF9, U+20AB;
    }

    @font-face {
        font-family: 'Inter';
        font-style: normal;
        font-weight: 400 700;
        font-display: fallback;
        src: url('#{$app-css-relative-asset-path}/font/Inter-roman.latin-ext.var.woff2') format('woff2 supports variations'),
            url('#{$app-css-relative-asset-path}/font/Inter-roman.latin-ext.var.woff2') format('woff2-variations');
        unicode-range: U+0100-024F, U+0259, U+1E00-1EFF, U+2020, U+20A0-20AB, U+20AD-20CF, U+2113, U+2C60-2C7F, U+A720-A7FF;
    }

    @font-face {
        font-family: 'Inter';
        font-style: normal;
        font-weight: 400 700;
        font-display: fallback;
        src: url('#{$app-css-relative-asset-path}/font/Inter-roman.latin.var.woff2') format('woff2 supports variations'),
            url('#{$app-css-relative-asset-path}/font/Inter-roman.latin.var.woff2') format('woff2-variations');
        unicode-range: U+0000-00FF, U+0131, U+0152-0153, U+02BB-02BC, U+02C6, U+02DA, U+02DC, U+2000-206F, U+2074, U+20AC, U+2122, U+2191, U+2193, U+2212, U+2215, U+FEFF, U+FFFD;
    }

    /* ... Same for the italic version, with font-style: italic; replaced */
}
```

After a `theme:compile` and `assets:install`, and opening the start page of the Shopware instance, which only contains `latin` characters, only one font file with the size of 33.24kb is transferred. Opening a page that contains both italic and normal text with `latin` characters, 68.78kb font files are transferred.

## Conclusion

After the reduction from 315.51kb to 164.95kb using a variable font in the last part, we could further reduce the font file size to just 33.24kb for `latin` characters. Most websites and online shops, where I did something probably only used the `latin`-character set. So this should be a huge improvement, in particular as the font files are required for proper rendering. Further improvements are possible when one can specify which characters are used in advance but this requires manually picking the glyphs which should be included.

So in total, we reduced, for the `latin`-character set, the font files from 315.51kb to just 33.24kb, which is a reduction of 89%. When one italic font is included the reduction from 423.19kb to 68.78kb is still a reduction of 84%.

My conclusion is, that if one decides to host the fonts locally, one should really go the extra mile and optimize the fonts further, as they are so critical in the loading phase of the initial webpage. Google already has implemented quite some improvements, which you do not automatically get if you for example download the font from the [google-webfonts-helper](https://gwfh.mranftl.com/fonts/inter?subsets=latin,latin-ext), which is still a great tool.
