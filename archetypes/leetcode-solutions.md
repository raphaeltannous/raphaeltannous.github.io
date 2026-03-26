---
date: '{{ .Date }}'
draft: false
title: '{{ replace .File.ContentBaseName `-` ` ` | title }}'
tags: ['Go']
leetcodeProblemSlug: '{{ .File.ContentBaseName }}'
leetcodeSolutionLink: ''
---
