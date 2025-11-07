---
# https://gohugo.io/content-management/front-matter/
# https://jpanther.github.io/congo/docs/front-matter/
title: "Test ground"
date: 2025-10-21T16:32:28+08:00
lastmod: 2025-10-21T16:32:28+08:00
categories: ["📚 閱讀"]
tags: ["test"]
draft: false
---

TODO
1. 評論
2. 圖床
3. Customize CSS (https://jpanther.github.io/congo/docs/advanced-customisation/#building-the-theme-css-from-source)

How to adjust size for figure element
https://tailwindcss.com/docs/width

{{< figure src="wallpaper.jpg" class="w-1/3 mx-auto" caption="寬度 33.3% 置中" >}}

{{< gallery caption="兩張圖片共同展示">}}
    {{< figure src="wallpaper.jpg" class="w-2/3 mx-auto">}}
    {{< figure src="wallpaper.jpg" class="w-2/3 mx-auto">}}
{{< /gallery >}}
