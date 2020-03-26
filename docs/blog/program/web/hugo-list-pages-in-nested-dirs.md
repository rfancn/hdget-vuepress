---
title: Hugo list all the pages under sub directories
url: hugo-list-pages-in-nested-dirs
date: 2017-12-01
categories:
- 博客
tags:
- program
- web
- hugo
---

#### (1) Background

Normally, we feel intuitive to put files under different nested sub directory, each directory could represents as file's category it belongs to or with specific purpose.
E,g: I created a post which describe how to fix some problem when coding in golang, so I will have a following directory hierarchy:

```
    - blog
        * blog-a.md
        * blog-b.md
        - program
            - golang
                - golang-a-problem.md
                - golang-b-problem.md
```

Then, I created another post which fix another problem for coding in web, then I will create another `web` directory:

```
        - blog
            * blog-a.md
            * blog-b.md
            - program
                - golang
                    * golang-a-problem.md
                    * golang-b-problem.md
                - web
                    * web-a-problem.md
                    * web-a-problem.md
```

It is convinient and useful if we can list out all the pages under nested directory from top to bottom:

- if we navigator to URL like: "http://xxx.com/blog/program/golang/", what we want is list below files:

    * fix-a-problem
    * fix-b-problem

- if we navigator to URL like: "http://xxx.com/blog/program/", what we want is list below files:

    * fix-a-problem
    * fix-b-problem
    * web-a-problem
    * web-b-problem

- if we navigator to URL like: "http://xxx.com/blog/", what we want is list below files:

    * blog-a page
    * blog-a page
    * fix-a-problem
    * fix-b-problem
    * web-a-problem
    * web-b-problem

#### (2) Problem
To achieve above purpose, we need add frontmeta data taxonomy manually, like "categories" to indicate what's the directory the page file belongs to. In fact, even workaround like this is not easy to be done. As in Hugo document, all `Page` variable has a `.Dir` and `.Path` attribute, which represents the filesystem directory hierarchy it belongs to. If we can check current `Page.Path` variable contains current Dir substring, then we can filter out all pages under specific dir.

Now the problems comes:
- Problem1: how to let each empty directory also represent as a `Page` variable?
- Problem2: how to check current `Page.Path` contains current dir string?

#### (3) Research
- how to let each empty directory also represent as a `Page` variable?

    To make sure each nested directory can contains a `Page` variable, we need make sure there is a `_index.md` file created under each directory, except the one with "index.md". To be more specific, it need have all "_index.[language].md" files under each directory, except the one with "index.*.md".

    - Why we ignore the directory with "index.*.md" files?
    Answer: The directory contains "index.*.md" files will be taken as a "specific content dir", it is not a normal placehold directory. Just take below directory hierarchy as exmple:

    ```
         - blog
                - program
                    - golang
                        - golang-a-problem
                            * index.md
                            * pic1.jpg
                            * pic2.jpg
    ```

    If the frontmatter of "index.md" under "golang-a-problem" directory doesn't contains `slug` or `url`, if we access URL: "/blog/program/golang/golang-a-problem/", it shows page content other than the list of files under "/blog/program/golang/golang-a-problem/"

    > Important: It is unnessary to create "_index.*.md" files under "content dir"(dir with "index.*.md"), for URL to such dir will render "single.html" template not "list/section.html" template

- How to check current `Page.Path` contains current dir string?

    To check current `Page.Path` field value contains current dir string, when we render the "section/list" template, we can get current `.Dir` information of current directory. Then we can iterate all `Site.RegularPages` to filter those page's `.Page.Path` contains current dir string.

#### (4) Solution
As we discussed above, we have the correct direction to fix the problems we encountered so far, now the solution is:

- How to let each empty directory also represent as a `Page` variable?

    I create a python script which will check directory hierarchy when everytime it runs, if it find that there is no "_index.[language].md" files created for normal directory, it will create them automatically.
    Or just remember this rule, if you want list out all the pages under specific directory, make sure you created "_index.[language].md" files under such directory.

- How to check current `Page.Path` contains current dir string?

    The difficult part is how to check page's `.Path` contains current dir string. Though we can check `.Path` hasPrefix of current Dir and list pages by filter them by if condition check, but how to paginator it with a filtered Collection? I almost checked all threads/discussion of Hugo comments/issues/PRs, but found there is no way to do that.
    The `where` function can be used to filter out collections, the similar operator `in` can check if given field value in match value or not, but now we need opposite operation, we need check if given field value contains match value or not.
    In the beginning, I created an enhancement issue to seek help from Hugo community:
    - [Add "contains" operator in where function to support list pages in nested subdir](https://github.com/gohugoio/hugo/issues/4131)

    But I cannot wait for so long time, then I created a pull request as below:
    - [tpl: Add "contains" operator in where tpl function](https://github.com/gohugoio/hugo/pull/4132)

    Detail of above pull request can be found here: [Hugo where "contains" operator implementation](http://www.hdget.com/blog/program/go/hugo-where-contains-operator/)

    As the time I wrote this post, the enhancement still didn't merge into main stream, so you may need build by yourself.

#### (5) Example Setup

#### Example Setup:

1. Organize content files with nested subdir
    
    ```
            - content/
               - blog/
                   - program/
                      - golang/
                      - python/
                   - cloud/
                     ...
    ```

2. make sure _index.[language code].md files created in each subdir including root section

    ```
            - content/
               - blog/
                    - _index.en.md
                    - _index.zh.md
                    - program/
                        - _index.en.md
                        - _index.zh.md
                        - golang/
                               - _index.en.md
                               - _index.zh.md
                               golang-pitfalls.md
                               golang-test.md
                        - python/
                   - cloud/
                   ...
    ```

3.  create layouts/_default/section.html as below:

    ```
            {{ $currentDir := .Dir }}
            {{ $paginator := .Paginate ( where (where .Site.AllPages "Kind" "page") "Path" "contains" $currentDir ) }}
            {{ range $paginator.Pages }}
                 {{ .Title }}
            {{ end }}
    ```

4. Tested with below urls to see if it list out all nested pages under given path/dir:

- http://localhost:1313/blog/
- http://localhost:1313/blog/program
- http://localhost:1313/blog/program/golang/