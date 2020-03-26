---
title: Python YAML To HTML Builder
date: '2017-09-09 12:40:23'
draft: false
lastmod: '2017-11-30 16:59:49'
tags:
- project
- y2h
categories:
- 项目
---

- Basic Elements
    - form
	- input
	- radio
	- checkbox
	- button
    - table

- Bootstrap Specific Elements
    - panel

#### Form

- layout:
	- possible values: ['horinzontal', 'default', 'inline']
	- default value: 'horinzontal'
	- comment: when add this to ```<form>``` element as it's attribute, it will automatically add corresponding css classes to match the form laytout, also this value can be passed to it's child control elements as 'layout' varaible

#### input
- help-label
	- comment: it will automatically add ```<label>``` element before ```<input>``` element

#### panel
- style:
      'default', 'primary', 'success', 'info', 'warning', 'danger'
- header
  style: primary ...