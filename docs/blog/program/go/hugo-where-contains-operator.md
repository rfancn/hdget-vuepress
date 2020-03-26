---
title: Hugo where "contains" operator implementation
date: 2017-11-30
draft: false
categories:
- 博客
tags:
- program
- golang
---
#### (1) Background
To fix following issue:

- [Add "contains" operator in where function to support list pages in nested subdir](https://github.com/gohugoio/hugo/issues/4131)

I implemented a new `contains` operator in Hugo `where` function in following PR:

- [tpl: Add "contains" operator in where tpl function](https://github.com/gohugoio/hugo/pull/4132)

#### (2) How Hugo where function works

Basically Hugo `where` function use the `checkCondition()` function to check if the given field value matches the match value or not. The complete codes in: "/tpl/collections/where.go"

- `v`: given field value, represents as reflect.Value
- `mv`: match value, also reprsents as reflect.Value
- `ivp`: the pointer of integer type of given filed value
- `imvp`: the pointer of integer type of match value
- `svp`:  the pointer of string type of give filed value
- `smvp`: the pointer of string type of match value
- `iva`:  a interger array converts from a given field value
- `sva`: a strign array converts from a given field value

> 1. Firstly, it gets the real value it points to if v or mv is a pointer or interface, `indirect()` defined in "./tpl/collections/apply.go"
> 2. if it can deference the `v` or `mv`, compare them with those basic operators
> 3. if it detected that `v` or `mv` is boolean value, just compare them
> 4. convert given field value and match value and stores to correpsonding varaibles
> 5. based on variables it generates, execute operate corresponding logic

Here, I take `In` operation as an example. when given field is basic type, e,g: string, and match value is a slice, it does below logic:

> 1. check if match value is a slice or array, if not, return false
> 2. if match value is zero length, return false
> 3. if child item's type in match value(now it is array or slice) doesn't match the given field value's type, return false
> 4. convert given field value and storge information into correspdoning variables
> 5. if given field value is a interger, then `ivp` is not nil, and by logic above, the match value is a interger slice, so a interger array `iva` build from match value slice should be generated already,
    use function `In` to compare the given field value in match value array or not
> 6. if given field value is a string, then `svp` is not nil, here match value can be a string or string list, compare use `In` function too

#### (3) New `contains` operator implementaion logic

> 1. Make sure given field value is either Slice, Array or String and try to converts given field value information to `ivp`, `svp`, `slv`
> 2. Make sure given field value length is larger than 0
> 3. If given field value is Slice or Array, then make sure the child item of given field is the same type as the match value
> 4. Converts the match value and store nessary information to `imvp`,`smvp`, `slmv`,
> 5. If given field value is Slice or Array and child item is TimeType, try to converts the given field value and match value to `ima` and `iva`, in other words, converts the timeType to int64 array
> 6. Finally, evaluate the contaings logic with below logic:
    - if `iva` is not nil which means the given field is an interger array, check if `iva`(interger array of give value) contains `imvp`(interger match value)
    - if `smvp` is not nil which means the match value is a string, then check below logic
    - if `sva` length larger than 0 which means given field is a string array, check if `sva`(string array of given value) contains `smvp`(string match value)
    -  else if `sva` is 0 but `svp` is not nil, which means given field is a string, check if `spv`(string of given value) contains substring `smvp`(string match value)
    - if `slv` and `slmv` is not nil, which means given value and match value are all either Slice or Array, then check below logic:
    - check if `iva` larger than 0, which means given value can converts to a timeType array, then we check if each item in `ima`(interger array of match value) are all in `iva`(interger array of give value), return true if all contains in given value, return false is it fails
    - above logic should returns if the child item in give value array is timeType, now for other types of child item, we just check with the Hugo `In` function

#### (4) How to apply the patch

> 1. Firstly, create the corresponding build path for hugo project
>
>       ```
>       # mkdir -p /path/to/GOROOT/src/github.com/gohugoio/
>       # cd /path/to/GOROOT/src/github.com/gohugoio/
>       # git clone https://github.com/gohugoio/hugo.git
>       ```
>
> 2. Switch to the release you want to apply the patch, e,g: release-0.37
>       ```
>       # git checkout release-0.37
>       ```
>
> 3. Download the patch: https://patch-diff.githubusercontent.com/raw/gohugoio/hugo/pull/4132.patch and put into git project dir
>       ```
>       # git apply --stat 4132.patch
>       # git apply --check 4132.patch
>       # git am --signoff < 4132.patch
>       ```
>      
> 4. Download and get `mage` compile tool ready, build hugo binary
>       ```
>       # go get github.com/magefile/mage
>       # mage vendor
>       # mage hugo
>       ```
>
> 5. The builded binary `hugo` file can be found in /path/to/GOROOT/src/github.com/gohugoio/hugo/hugo


