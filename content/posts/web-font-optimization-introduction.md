---
title: "Web Font Optimization - Introduction"
date: 2022-12-30T15:36:28+01:00
draft: false
---

## Introduction

Hello, and welcome to my new blog. I will start this blog with a little series discussing font optimizations relevant to delivering fonts on the web.

I have to say, I knew very few technical details before I needed to investigate fonts to be included on a web page for a customer. So test for yourself what works and what does not work. We have deployed the discussed optimizations for the customer, where the same steps as outlined in this series were performed. Additionally, I ~~plan to~~ created [two](https://github.com/shopware/platform/pull/2909) [pull requests](https://github.com/shopware/platform/pull/2916) for [Shopware 6](https://github.com/shopware/platform/), shipping the [Inter](https://rsms.me/inter/) font optimized for the default storefront.

The topic came up when it was not clear if one could or could not use Googlefonts, without user consent and complying with the [GDPR](https://gdpr.eu/). But we wanted to implement some optimizations for that customer's fonts already, so I had finally gotten time to learn more about fonts. I did a quick search and found very little or only outdated (or non-Linux) documentation of tools which can manipulate font files. So I quickly switched to reading [code](https://github.com/fonttools/fonttools) and [documentation](https://fonttools.readthedocs.io/en/latest/index.html), mainly of `fonttools` and the [OpenType font specification](https://learn.microsoft.com/en-us/typography/opentype/spec/). Of course, there are [some](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_Fonts/Variable_Fonts_Guide) [good](https://web.dev/variable-fonts/) [blog posts](https://damieng.com/blog/2021/12/03/using-variable-webfonts-for-speed/), which discuss some details which are also covered in this series, but not in depth I wanted to know about them. And anyways I wanted to do some research to learn more about font internals.

I found most graphical tools to be more misleading than helping, but that is maybe also my taste. The GUI open-source tool I found was [`FontForge`](https://fontforge.org/en-US/), and I sometimes used it to verify some things.

If you want to follow the hands-on examples of the next parts of this series, you should somehow have the Python `fonttools` library installed. But you might also learn some interesting things about fonts without repeating the steps.

## Overview of the series

Currently, I plan to create three parts of this series. The first one (this one) will contain, besides the introduction, some history of fonts and basic reviews of modern computer fonts. In the [second part]({{< ref "/posts/web-font-optimization-variable-font" >}}), we will create a variable font that can be used on the web. In the [third part]({{< ref "/posts/web-font-optimization-subsetting-fonts" >}}) of the series, we will discuss the reduction of the included glyphs in a font.

If you already know all about fonts or are not interested in the background of fonts and only want to see how to put the fonts on your website, you can skip the rest of this part.

## History of computer fonts

Let us start with the definition from [Wikipedia of what a font](https://en.wikipedia.org/wiki/Font) is:

> In metal typesetting, a font is a particular size, weight and style of a typeface. Each font is a matched set of type, with a piece (a "sort") for each glyph. A typeface consists [of] various fonts that share an overall design.

This means that in the traditional definition, for example, different font sizes represented different fonts. In my mind, I usually compared a font to something specified by a single file, and the collection of multiple of those files represent a font family. As I typically use fonts in CSS, this is probably where this picture comes from. In CSS, the [`font-family`](https://developer.mozilla.org/en-US/docs/Web/CSS/font-family) corresponds to the original meaning of "typeface".
In fact, this is stated in the following sentence on Wikipedia

> In modern usage, with the advent of computer fonts, the term "font" has come to be used as a synonym for "typeface", although a typical typeface (or "font family") consists of several fonts.

So in the future, I will probably try to use "font" more consistently as a true synonym for "typeface", and fonts, in general, can consist of arbitrary many font files.

Of course, we are not interested in the traditional definition and history of fonts, but I thought this might be a good start to clear up some confusion. This brings us to the next aspect, how fonts are stored on the computer. All font formats I know and have heard of are build as collection of glyphs. A glyph, in general, is a symbol, which can be, for example, one or more letters combined into a single symbol. But a glyph can also be, for example, an emoji. Again according to [Wikipedia](https://en.wikipedia.org/wiki/Computer_font), there are currently three different types of how each glyph is represented:

> - **Bitmap** fonts consist of a matrix of dots or pixels representing the image of each glyph in each face and size.
> - **Vector** fonts (including, and sometimes used as a synonym for, outline fonts) use Bézier curves, drawing instructions and mathematical formulae to describe each glyph, which make the character outlines scalable to any size.
> - **Stroke** fonts use a series of specified lines and additional information to define the size and shape of the line in a specific typeface, which together determine the appearance of the glyph.

So bitmap fonts represent a traditional metal typesetting font. The vector and stroke type fonts implement the optimization that they are scalable to remove the need for different font files for different font sizes and other properties. As in the bitmap files, a lot of duplicate information is contained, those formats reduce the total file size for a font. As vector fonts are most common, e.g., TrueType and OpenType fonts are both vector font types, for the rest of this series, we will only discuss vector fonts, particularly the two most common formats (at least for me), which are TrueType and OpenType fonts.

## TrueType and OpenType font formats

The [TrueType](https://en.wikipedia.org/wiki/TrueType) font format was developed by Apple in the 80's, which allowed specifying the scaling of the font to arbitrary font sizes. This meant that now only one font file for different font sizes is needed, but still, for example, regular, bold, and italic style are required to be shipped as separate files. In particular, each desired combination of those also needed to be shipped.

The successor of TrueType specification is the [OpenType](https://en.wikipedia.org/wiki/OpenType) specification, which has a similar basic structure as TrueType fonts. But they add several extensions and data structures to allow better control for scaling of the typographic behavior. In particular in 2016, with [the release of the OpenType specification 1.8](https://www.youtube.com/watch?v=6kizDePhcFU), Microsoft introduced "Font Variations", allowing not only to control the scaling, but also other properties, like boldness, glyph width, etc. These font variations are commonly now referred to as variable fonts.

In other words, these variable fonts generalize the concept of scaling by introducing additional "axes". These axes are then used to adjust the properties of the font dynamically. In the [OpenType specification currently exist five "registered axis"](https://learn.microsoft.com/en-us/typography/opentype/spec/dvaraxisreg#registered-axis-tags), namely Italic (`ital`), Optical size (`opsz`), Slant (`slnt`), Width (`wdth`), and Weight (`wght`). But the specification allows the font designer to add additional axis specific to the font. Note that it is not required for a font to have all "registered axis", but they can choose only to support a subset. I also noticed that some fonts implemented 'Slant' instead of 'Italic'. This should be stated in the README of the font.

Note that the file extension for both, TrueType and OpenType can be `.ttf`. But nowadays, these fonts are stored in the [scalable font (SFNT)](https://en.wikipedia.org/wiki/SFNT)-format anyway, so the mime type will be `font/sfnt`. But this does not really matter ☺, as the font rendering libraries can usually determine the type correctly.

## Web Open Font Format (WOFF)

The TrueType and OpenType font formats provide much flexibility. Still, they have not been designed to minimize the required file size, as they have already significantly reduced the total file size compared to example bitmap fonts. But additionally, using the files with maximum compression would mean first decompressing the data, before rendering.

But in particular, for the web, it is crucial that the transferred file size is reduced to a minimum. So as it became popular to ship font files along a web page a further reduction of the file size was desired, as the decompression probably used much less time than the file transfer. So the "Web Open Font Format" (WOFF) was created, which introduces additional data compression. For [`WOFF`](https://www.w3.org/TR/WOFF/) it is `gzip` and for `WOFF2` it is `brotli`. Additionally, [`WOFF2`](https://www.w3.org/TR/WOFF2/) implements Variable and Chromatic fonts and font Collections:

> The WOFF 2.0 specification is implemented in all major browsers, and is widely used on production websites. It supports the entirety of the TrueType and OpenType specifications, including Variable fonts, Chromatic fonts, and font Collections.

## Conclusion

Although this part was mostly reading Wikipedia and some specifications, for me, it was helpful to understand the development of fonts and their relationship to each other. For example, the connection from `WOFF` to OpenType/TrueType means that one can always easily compress and decompress those two formats. Furthermore, using `WOFF` to store often-used fonts locally is probably not reasonable.
