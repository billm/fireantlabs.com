---
title: "{{ slicestr .Name 11 | humanize | title }}"
date: {{ .Date }}
year: "{{ .Date | dateFormat "2006" }}"
month: "{{ .Date | dateFormat "2006/01" }}"
toc: false
comments: true
images:
tags:
draft: true
---

