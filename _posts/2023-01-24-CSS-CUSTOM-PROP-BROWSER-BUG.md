---
layout: post
title: "Browser bug in CSS custom property value serialization"
date: 2023-01-24 12:00:00 -0000
categories: CSS
---

[**Custom properties** (sometimes referred to as **CSS variables** or **cascading variables**) are entities defined by CSS authors that contain specific values to be reused throughout a document.](https://developer.mozilla.org/en-US/docs/Web/CSS/Using_CSS_custom_properties)

Example:

```
element {
  --main-bg-color: brown;
}
```

Since some CSS properties, such as [`url`](https://developer.mozilla.org/en-US/docs/Web/CSS/url) , allows the setting of custom string without any validation, it is important to specify that the values of the CSS custom properties should remain the same.

In fact, the [specification](https://drafts.csswg.org/css-variables-1/#serializing-custom-props) is quite clear about this:

> Specified values of custom properties must be serialized exactly as specified by the author. Simplifications that might occur in other properties, such as dropping comments, normalizing whitespace, reserializing numeric tokens from their value, etc., must not occur.  
> Computed values of custom properties must similarly be serialized exactly as specified by the author, save for the replacement of any var() functions.

However, when it comes to large integers within the custom string, Chrome and Safari behave oddly.

For example, if I set the following CSS custom properties, each browser outputs the following:

```
:root {
  --logo-path: /1000000000000000000000/asset/logo.svg;
  --logo-path-double: "/1000000000000000000000/s/assets/images/logo-alpine-group.svg";
  --logo-path-single: '/1000000000000000000000/s/assets/images/logo-alpine-group.svg';
}
```

Chrome (Version 93.0.4577.63 (Official Build) (x86_64)):

```
naked:
/1.00000e+21/s/assets/images/logo-alpine-group.svg
double quotes:
"/1000000000000000000000/s/assets/images/logo-alpine-group.svg"
single quotes:
"/1000000000000000000000/s/assets/images/logo-alpine-group.svg"
```

Firefox (91.0.2 (64-bit)):

```
naked:
/1000000000000000000000/s/assets/images/logo-alpine-group.svg
double quotes:
"/1000000000000000000000/s/assets/images/logo-alpine-group.svg"
single quotes:
'/1000000000000000000000/s/assets/images/logo-alpine-group.svg'
```

Safari (Version 14.1.2 (15611.3.10.1.5, 15611)):

```
naked:
/1.00000e+21/s/assets/images/logo-alpine-group.svg
double quotes:
"/1000000000000000000000/s/assets/images/logo-alpine-group.svg"
single quotes:
"/1000000000000000000000/s/assets/images/logo-alpine-group.svg"
```

As you can see, Safari and Chrome will convert the large number inside the custom string to scientific notation.

As discussed in the [spec clarification](https://github.com/w3c/csswg-drafts/issues/6572#issuecomment-911910455), this is presumably happening because they are tokenizing and deserializing the value, whereas Firefox is simply storing the custom string as-is.

Interestingly enough, [Webkit even has a test for ensuring that integers always serialize all their digits out](https://github.com/WebKit/WebKit/blob/main/LayoutTests/imported/w3c/web-platform-tests/css/cssom/serialize-custom-props.html#L10-L15), but it must not cover CSS custom properties.

Bugs have been filed to respective trackers:

- https://bugs.webkit.org/show_bug.cgi?id=229816
- https://bugs.chromium.org/p/chromium/issues/detail?id=1246077&q=20210629&can=2
