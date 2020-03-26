---
date: 2017-11-25
draft: false
lastmod: '2017-11-25 20:16:11'
title: Hugo common pitfalls and cheat sheet
categories:
- 博客
tags:
- program
- web
- hugo
---

- Why language specific params defined in "[lanuage]" missing some information defined outside "[lanuage]"

        [params]
            [params.home]
                var1 ="1"
                var2 ="2"

        [languages]
            [languages.en]
                [languages.en.params.home]
                    var1="changed"

> The language specific params will override the variable with the same name defined outside of "[language]",
> In above example, there is no var2 availble, and `Site.Params.Home` only can see var1="changed"


- Why `baseURL` defined in config file doesn't take effect when running in `hugo server` mode?

> All the `baseURL` will with `baseUrl` "http://localhost:port" when run in `hugo server` mode,
> but if you run `hugo` command, you can see the baseUrl you specified contains in geneated pages.
> the workaround is run `hugo server -b [baseURL]` to explicitly specify the `baseURL`

- How to list all the pages under specific subdir without using taxonomy?

> There is no easy way to do that, especially if you want to paginator the pages.
> I wrote a document to detail desribe how to do it here:
> [Hugo list all the pages under sub directories](http://www.hdget.com/blog/program/web/hugo-list-pages-in-nested-dirs/)
