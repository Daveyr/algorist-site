---
date: '2025-03-24T23:00:00-03:00'
series:
- styling
- features
tags:
- guide
title: Dark Theme
topics:
- documentation
---

You can change the current theme between dark and light mode by using the switcher icon at the right of the navbar.

The colors tweaked by the theme change are ideally defined in a non bundled CSS file (to optimize caching, since the theming is expected to change more than the rest) under the `:root[color-scheme^='dark/light']` selector. Should you want to adjust them, a simple stylesheet rules override should suffice. You can read about it in customization.
