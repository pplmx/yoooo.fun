---
title: "{{ replace .Name "-" " " | title }}"
date: {{ .Date }}
draft: false
slug: "{{ .Name | md5 }}"
---

