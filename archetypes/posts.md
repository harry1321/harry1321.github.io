---
# https://gohugo.io/content-management/front-matter/
# https://jpanther.github.io/congo/docs/front-matter/
title: "{{ replace .Name "-" " " | title }}"
date: {{ .Date }}
lastmod: {{ .Date }}
draft: true  # Delete this line to publish
# For og:description, markdown is NOT supported
description: ""
# For og:image, images are auto-included, only lists external images here
images: []
# For og:audio, only support one audio
audio: ""
# For og:video, support multiple videos
videos: []
# Categories taxonomy, choose only from the list (idealy, just one)
categories: ["📚 閱讀","🍟 生活", "📱 技術", "💻 LC"]
# Tags taxonomy, and article:tag (first 6 links will be used)
tags: ["🏷️"]
# https://jpanther.github.io/congo/docs/getting-started/#feature-cover-and-thumbnail-images
# Feature image for thumbnail in list page, RSS, and top of the content
feature: ""
# Alt text (non-visible) for feature image
featureAlt: ""
# Description (visible) for feature image
coverCaption: ""
---
