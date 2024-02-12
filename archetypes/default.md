---
title: "{{ replace .Name "-" " " | title }}"
date: {{ .Date }}
draft: true

# Meta of the article
tags: [ "TAG 1", "TAG 2", ]
author: [ "Your Name" ]
cover:
  image: "/articles/your_new_article/cover.png" # Relative path to the cover image
  alt: "Article cover"
  relative: true

# Table of contents settings
showToc: true
TocOpen: true
UseHugoToc: true
---

