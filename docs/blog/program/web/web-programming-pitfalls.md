---
date: '2017-09-09 14:22:30'
draft: false
lastmod: '2017-11-25 20:16:11'
categories:
- 博客
tags:
- program
- web
title: Web Programming Pitfalls
---

1. How to send jquery datatables data via Ajax
A: Use: dataTable_instanc().rows().data().toArray()

2. How to send json data via Ajax
A: It need use JSON.stringtify(Array) to make the javascript object/array to be string,
and set the contentType to be: "application/json; charset=utf-8"

        $.ajax({
            ...
            contentType:"application/json; charset=utf-8",
            data: JSON.stringtify({'name': 'ryan'})    
        })



3. How to send Vue instance data via Ajax
A: Use {'key': Vue_instance.$data }

        $.ajax({
            ...
            contentType:"application/json; charset=utf-8",
            data: JSON.stringtify({'general': general.$data})
        })