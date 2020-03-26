---
title: Hugo breadcrumb design and implementation
date: 2017-12-11 20:00:00
categories:
- 博客
tags:
- program
- web
---

There is a Hugo official example which describe how to implement a navigation breadcrump here:

- [Example: Breadcrumb Navigation](https://gohugo.io/content-management/sections/#example-breadcrumb-navigation)

Anyway I didn't make it work as I expected. I think it may not work if we oragnize contents use nested subdir.However I will implement a breadcrumb based on Hugo `contains` function.

#### Precondition

Hugo where `contains` function has been integrated[Hugo "contains" operator implementation in "where" function](/blog/program/go/hugo-where-contains-opreator/), and have a section template like below:

```jinja2
{{ $currentDir := .Dir }}
<div class="container">
    <div class="row">
        <div class="col">
            {{ partial "breadcrumb.html" . }}

            {{ $paginator := .Paginate ( where (where .Site.AllPages "Kind" "page") "Path" "contains" $currentDir ) }}
            {{ range $paginator.Pages }}
                {{ partial "post_summary.html" . }}
            {{ end }}

            <!-- navigation -->
            {{ partial "pagination.html" . }}
        </div>
    </div>
</div>
```

#### Breadcurmb

1. Process the `Page.Dir` variable, as it has format: `/blog/cloud/docker/`, we need remove the last `/` from the string, the final result would be `/blog/cloud/docker`

```jinja2
{{ $trimmedDir := strings.TrimSuffix "/" .Dir }}
```

2. Split the $trimmedDir to a Slice like: ["blog", "cloud", "docker"]

```jinja2
{{ $categories := split $trimmedDir "/" }}
```

3. Calculate the length of categories

```jinja2
{{ $total := len $categories }}
```

4. If length of categories >0, then calculate last item's index which will be uased in next range logic to check if we reach to the last item or not.

```jinja2
{{ $lastIndex := sub $total 1 }}
```

5. Iterate the categories item, and do below logic

- If we reaches to the last item, condition check below code is true, then we need make the current category item to be the active one
  
```jinja2
{{ if eq $i $lastIndex }}
```

- For other category, we need build a valid relative URL with below logic:

  - As categories are split from the top to bottom, the first (loop index + 1) part of categories collection represents the correct relative path

  - Get how many items we need to fetch in the categories from the left to right

  ```jinja2
  {{ $firstPart := add $i 1 }}
  ```

  - Get the first part of categories with $firstPart number

  ```jinja2
  {{ $subDirs := first $firstPart $categories }}
  ```

  - Assembly sub directories with `delimit` function and DELIMIT is "/"

  ```jinja2
  {{ $relUrl := delimit $subDirs "/" }}
  ```

  - For example: assume categories is a Slice of ["blog", "cloud", "docker"]

    - if $i(loop index) is 0, then we need get the "first 1", url is:

      ```jinja2
      $url = "blog"
      ```

    - if $i(loop index) is 1, then we need get the "first 2" part of categories, url is:

      ```jinja2
      $url = "blog/cloud"
      ```

#### Complete "breadcrumb.html" template

```jinja2
{{ $trimmedDir := strings.TrimSuffix "/" .Dir }}
{{ $categories := split $trimmedDir "/" }}
{{ $total := len $categories }}

{{ if gt $total 0 }}

{{ $lastIndex := sub $total 1 }}
<nav class="breadcrumb">
    <a class="breadcrumb-item" href="/">Home</a>
    {{  range $i, $v := $categories }}
            {{ if eq $i $lastIndex }}
                <span class="breadcrumb-item active">{{ $v }}</span>
            {{ else }}
                {{ $firstPart := add $i 1 }}
                {{ $subDirs := first $firstPart $categories }}
                {{ $relUrl := delimit $subDirs "/" }}
                <a class="breadcrumb-item" href="/{{ $relUrl }}">{{ $v }}</a>
            {{ end }}
    {{ end }}
</nav>

{{ else }}

<nav class="breadcrumb">
    <span class="breadcrumb-item active">Home</span>
</nav>

{{ end }}
```
