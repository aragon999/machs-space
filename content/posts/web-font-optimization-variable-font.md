---
title: "Web Font Optimization - Variable fonts"
date: 2023-01-02T00:05:48+01:00
draft: false
---

## Introduction

In the [first part]({{< ref "/posts/web-font-optimization-introduction" >}}) of this series, we discussed some of the relevant aspects and basic ideas behind fonts and in particular variable fonts. In this part, we will now prepare a variable font for implementation on a webpage and provide the necessary CSS including a fallback.

I will illustrate the example on the Shopware default font, which is [Inter](https://github.com/rsms/inter). This font is currently not included as a variable font in Shopware 6, although a variable font is provided in the upstream [release of the font](https://rsms.me/inter/lab/).

For this post, I used the resources of [web.dev on variable fonts](https://web.dev/variable-fonts/) and the [mdn web docs on variable fonts](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_Fonts/Variable_Fonts_Guide) as introductory readings.

We use [fonttools](https://github.com/fonttools/fonttools) to manipulate the font files and [`woff2_compress`](https://github.com/google/woff2) to compress the `.ttf` font files to `.woff2` font files.

## The current state of variable fonts on the web

As learned in the previous part, variable fonts support multiple variation axis. Therefore we need to specify which values on these axes should be used for displaying a particular text, which is of course done via CSS. There are two relevant parts to this. On the one hand, when we ship a custom font, we need to have a `@font-face` declaration, which contains all needed information for the browser like the `font-family`-name, the `src`  where the font files can be found, and other attributes that this `@font-face`-declaration can be used for. Examples of other attributes are `font-weight`, `font-stretch`, and so on. On the other hand, we need to specify which explicit values for the attributes should be applied to the target element. The browser then goes through all available `@font-face` declarations (including the ones which are provided by the operating system) and tries to find a matching declaration.

There is a [blog post](https://evilmartians.com/chronicles/the-joy-of-variable-fonts-getting-started-on-the-frontend) describing more technical details, but I will quickly summarize, the relevant parts for this post. There are two ways to control, which values from the font on the axes should be used. On the one hand, there are the "high level" CSS properties, [`font-weight`](https://developer.mozilla.org/en-US/docs/Web/CSS/font-weight) (`wgth`), [`font-stretch`](https://developer.mozilla.org/en-US/docs/Web/CSS/font-stretch) (`wdth`), [`font-optical-sizing`](https://developer.mozilla.org/en-US/docs/Web/CSS/font-optical-sizing) (`opsz`), [`font-style`](https://developer.mozilla.org/en-US/docs/Web/CSS/font-style) (`ital` and `slnt`). On the other hand, it is also possible to use the low-level function [`font-variation-settings`](https://developer.mozilla.org/en-US/docs/Web/CSS/font-variation-settings), to specify the values of axes, e.g., `font-variation-settings: "wght" 400, "wdth" 300;`. A translation table between the registered axes and CSS properties can be found in the [mdn web docs](https://developer.mozilla.org/en-US/docs/Web/CSS/font-variation-settings#registered_and_custom_axes).

Each of those CSS properties, which are applied to the element containing the text to be styled, have a corresponding descriptor, which can be used in the `@font-face` declaration, to specify for which font parameters this declaration can be used. For further information c.f. to the mdn web docs: [`font-weight`](https://developer.mozilla.org/en-US/docs/Web/CSS/@font-face/font-weight), [`font-stretch`](https://developer.mozilla.org/en-US/docs/Web/CSS/@font-face/font-stretch) and [`font-style`](https://developer.mozilla.org/en-US/docs/Web/CSS/@font-face/font-style). In contrast to the CSS properties, the values of those descriptors can be a range, e.g., [`font-stretch: 75% 125%;`](https://developer.mozilla.org/en-US/docs/Web/CSS/@font-face/font-stretch#variable_fonts) or [`font-weight: 300 500;`](https://developer.mozilla.org/en-US/docs/Web/CSS/@font-face/font-weight#variable_fonts). Of course, there is also the low-level function [`font-variation-settings`](https://developer.mozilla.org/en-US/docs/Web/CSS/@font-face/font-variation-settings) to set the default variations of the `@font-face`-rule. At the time of writing this CSS declaration is only [supported in Firefox](https://developer.mozilla.org/en-US/docs/Web/CSS/@font-face/font-variation-settings#browser_compatibility).

Note that there is no `font-optical-sizing`-descriptor for `@font-face`-rules, as the CSS property currently only supports `auto` and `none` values, i.e. the browser tries to determine the optimal value to be chosen from the `opsz` axis if the value is `auto` or the default one is used if `none` is specified. If one needs more fine-grained control, to use a specific value of that font-axis [`font-variation-settings` must be used](https://css-tricks.com/almanac/properties/f/font-optical-sizing/#aa-another-way-to-optically-size-fonts).

In general, one should probably preferably use the "high-level" functions if possible, as they are more commonly known. Additionally, if there would be new font specifications, the high-level functions should work independently of, for example, the explicit naming of the font axes.

## Implementing Inter for Shopware 6 as variable font

In the case of the Inter font, the [release](https://github.com/rsms/inter/releases/tag/v3.19) already contains the variable font files for the web. For example in `Inter-3.19.zip` the `WOFF2` compressed files are located at `/Inter Web/Inter-{italic,roman}.var.woff2` with the single axis (`wght`). Additionally, the release contains the `/Inter Web/Inter.var.woff2` file, which includes the `slnt`-axis to control the style, i.e. `roman`/`normal` (`font-variation-settings: "slnt" 0deg`) and `italic` (`font-variation-settings: "slnt" 10deg`). This is not always the case, e.g., [`opensans` has no releases and only provides](https://github.com/googlefonts/opensans/tree/main/fonts/variable) the `TTF` files with two axes.

### Creating fonts with reduced axis values

We could directly use the provided variable fonts, but there would be two downsides. On the one hand, Shopware 6 currently only includes the `font-weight`s `400`, `600`, and `700`, so we do not need to ship the full range. On the other hand, the `italic` style of Inter is realized by the Slant font axis. While in general, this is no problem, it would break the backward compatibility in Shopware, as in this case to have an italic text one would need to specify `font-style: oblique 10deg;` instead of `font-style: italic;`.

Using `fonttools`, restricting the ranges of an existing variable font is easy. For example, if we would like to extract the `normal` (`slnt = 0`) font from the variable font with both axes (`/Inter Variable/Inter.ttf`) we can do this using the command (assuming you are in the directory which contains the file `Inter.ttf`):
```bash
$ fonttools varLib.instancer -o Inter-roman.ttf Inter.ttf "wght=400:700" "slnt=0"
```
which creates a new file `Inter-roman.ttf` with just a single font-axis `wght` from 400 - 700 and the `slnt` axis fixed at the value `0`.

When trying to extract the Italic version, I noticed two things. First, the value on th `slnt` axis is `-10` and not `10` as one would expect from the CSS definition. I have not dug deeper, but sure there is a reason for this minus sign. The other thing is, that the naming in the metadata is wrong, see this [Github issue](https://github.com/fonttools/fonttools/issues/2613). which can be fixed by [writing custom Python code](https://github.com/fonttools/fonttools/issues/2613#issuecomment-1164496823). But in general, I think the metadata should not matter when the fonts are used on the web. As far as I have noticed, it mainly concerns font pickers in text editing programs.

Luckily there already exist the "single axis" variable `wght` fonts, with the correct metadata `Inter Variable/Single axis/Inter-roman.ttf` and `Inter Variable/Single axis/Inter-italic.ttf`, where we just need to reduce the `wght`-axis:
```bash
$ fonttools varLib.instancer -o Inter-italic.reduced.ttf Inter-italic.ttf "wght=400:700"
$ fonttools varLib.instancer -o Inter-roman.reduced.ttf Inter-roman.ttf "wght=400:700"
```

If we now could merge those two fonts on a new axis `ital`, with the (discrete) values `0` and `1`, we could ship a single font file. Unfortunately, I did not find a ready-made CLI tool to merge two fonts on a discrete axis, but one probably could do it, by writing a custom Python script. The other way would be to directly create the correct variable font from the Inter source files. Unfortunately for both, I have no time at the moment. So for the [Shopware pull request](https://github.com/shopware/platform/pull/2909), we will only reduce font files to two and not one. It might become the subject of a future blog post though.

Using `woff2_compress` we can then convert the `ttf`-files to `woff2`-files:
```bash
$ woff2_compress Inter-roman.reduced.ttf
$ woff2_compress Inter-italic.reduced.ttf
```

We can now compare the file sizes of the different files. Note that for the comparison I additionally created the `woff2` converted file for all variable font files. The resulting file sizes are very promising,
```
602k Inter-roman.ttf
227k Inter-roman.woff2

613k Inter-italic.ttf
245k Inter-italic.woff2

473k Inter-roman.reduced.ttf
165k Inter-roman.reduced.woff2

479k Inter-italic.reduced.ttf
176k Inter-italic.reduced.woff2

805k Inter.ttf
325k Inter.woff2
```
as we can replace with the `Inter-roman.reduced.woff2` and `Inter-italic.reduced.woff2` files three files each, with roughly 100kb each. Note that in principle we could now also vary the font-weight more flexibly and also replace the file with `font-weight: 500;` but also choose an arbitrary weight in between.

### Including the fonts on the website

Now we have all the needed font files available to include them using CSS. We will use the CSS [`@supports`](https://caniuse.com/?search=supports)-feature, which is not supported in IE11, to check if the browser supports variable fonts, which in our case should be no problem, as Shopware 6.5 will drop IE11 support, since it is not supported by [Bootstrap 5](https://getbootstrap.com/docs/5.0/getting-started/browsers-devices/#supported-browsers). We will check for the `font-variation-settings: normal`-support, which is the recommended fallback way according to [web.dev](https://web.dev/variable-fonts/#fallbacks).

I renamed the `Inter-{roman,italic}.reduced.woff2` to `Inter-{roman,italic}.var.woff2`, placed them in `src/Storefront/Resources/app/storefront/dist/assets/font` and added the corresponding SCSS for Shopware in the [`@font-face`-declaration file](https://github.com/shopware/platform/blob/66f63d2eb69faef04412908668a257cc05722e7a/src/Storefront/Resources/app/storefront/src/scss/skin/shopware/vendor/_inter-fontface.scss)
```css
@supports not (font-variation-settings: normal) {
    @font-face {
        font-family: 'Inter';
        font-style: normal;
        font-weight: 400;
        font-display: fallback;
        src: url('#{$app-css-relative-asset-path}/font/Inter-Regular.woff2') format('woff2'),
            url('#{$app-css-relative-asset-path}/font/Inter-Regular.woff') format('woff');
    }

    @font-face {
        font-family: 'Inter';
        font-style: italic;
        font-weight: 400;
        font-display: fallback;
        src: url('#{$app-css-relative-asset-path}/font/Inter-Italic.woff2') format('woff2'),
            url('#{$app-css-relative-asset-path}/font/Inter-Italic.woff') format('woff');
    }

    @font-face {
        font-family: 'Inter';
        font-style: normal;
        font-weight: 600;
        font-display: fallback;
        src: url('#{$app-css-relative-asset-path}/font/Inter-SemiBold.woff2') format('woff2'),
            url('#{$app-css-relative-asset-path}/font/Inter-SemiBold.woff') format('woff');
    }

    @font-face {
        font-family: 'Inter';
        font-style: italic;
        font-weight: 600;
        font-display: fallback;
        src: url('#{$app-css-relative-asset-path}/font/Inter-SemiBoldItalic.woff2') format('woff2'),
            url('#{$app-css-relative-asset-path}/font/Inter-SemiBoldItalic.woff') format('woff');
    }

    @font-face {
        font-family: 'Inter';
        font-style: normal;
        font-weight: 700;
        font-display: fallback;
        src: url('#{$app-css-relative-asset-path}/font/Inter-Bold.woff2') format('woff2'),
            url('#{$app-css-relative-asset-path}/font/Inter-Bold.woff') format('woff');
    }

    @font-face {
        font-family: 'Inter';
        font-style: italic;
        font-weight: 700;
        font-display: fallback;
        src: url('#{$app-css-relative-asset-path}/font/Inter-BoldItalic.woff2') format('woff2'),
            url('#{$app-css-relative-asset-path}/font/Inter-BoldItalic.woff') format('woff');
    }
}

@supports (font-variation-settings: normal) {
    @font-face {
        font-family: 'Inter';
        font-style: normal;
        font-weight: 400 700;
        font-display: fallback;
        src: url('#{$app-css-relative-asset-path}/font/Inter-roman.var.woff2') format('woff2 supports variations'),
            url('#{$app-css-relative-asset-path}/font/Inter-roman.var.woff2') format('woff2-variations');
    }

    @font-face {
        font-family: 'Inter';
        font-style: italic;
        font-weight: 400 700;
        font-display: fallback;
        src: url('#{$app-css-relative-asset-path}/font/Inter-italic.var.woff2') format('woff2 supports variations'),
            url('#{$app-css-relative-asset-path}/font/Inter-italic.var.woff2') format('woff2-variations');
    }
}
```

After a `theme:compile` and `assets:install`, my Browser now downloads only a single font file instead of three when for example loading the home page. On a category page where I added some italic text, the two font files are downloaded.

## Conclusion

For a page of a demo installation of Shopware, which does not include an italic font, this reduced the transferred file size from 315.51kb to 164.95kb (48%). And additionally, we only need to download one file instead of three. For a page, which includes italic text, in only one font weight, and thus four font files are transferred with a total size of 423.19kb, there is still a reduction to 340.93kb (19%). Of course, if more than one italic font weight is used on the page, the reduction will be larger.

When you compare the file sizes of the font files to the ones which are shipped by Googlefonts, you will notice that the file sizes are still significantly larger. In the last part of this series, we will discuss how the file size can be reduced further, by splitting up the fonts for different glyph sets.
