---
date: '2017-04-13 10:21:07'
draft: false
lastmod: '2017-11-25 20:16:11'
categories:
- 博客
tags:
- tutorial
title: YAML simple tutorial
---

### Introduction
YAML stands for "YAML Ain't Markup Language", It’s basically a human-readable structured data format. It is less complex and ungainly than XML or JSON, but provides similar capabilities. It essentially allows you to provide powerful configuration settings, without having to learn a more complex code type like CSS, JavaScript, and PHP.One of the benefits of using YAML is that the information in a single YAML file can be easily translated to multiple language types.

===

### Basic Rules
There are some rules that YAML has in place to avoid issues related to ambiguity in relation to various languages and editing programs. These rules make it possible for a single YAML file to be interpreted consistently, regardless of which application and/or library is being used to interpret it.

* YAML files should end in .yaml whenever possible.
* YAML is case sensitive.
* YAML does not allow the use of tabs. Spaces are used instead as tabs are not universally supported.

### Basic Data Types
YAML excels at working with mappings (hashes / dictionaries), sequences (arrays / lists), and scalars (strings / numbers). While it can be used with most programming languages, it works best with languages that are built around these data structure types. This includes: PHP, Python, Perl, JavaScript, and Ruby.

#### Scalars
Scalars are a pretty basic concept. They are the strings and numbers that make up the data on the page. A scalar could be a boolean property, like Yes, integer (number) such as 5, or a string of text, like a sentence or the title of your website.

Scalars are often called variables in programming. If you were making a list of types of animals, they would be the names given to those animals.

Most scalars are unquoted, but if you are typing a string that uses punctuation and other elements that can be confused with YAML syntax (dashes, colons, etc.) you may want to quote this data using single ' or double " quotation marks. Double quotation marks allow you to use escapings to represent ASCII and Unicode characters.

    integer: 25
    string: "25"
    float: 25.0
    boolean: Yes

#### Sequences
Here is a simple sequence, It is a basic list with each item in the list placed in its own line with an opening dash.

    - Cat
    - Dog
    - Goldfish

This sequence places each item in the list at the same level. If you want to create a nested sequence with items and sub-items, you can do so by placing a single space before each dash in the sub-items. YAML uses spaces, NOT tabs, for indentation. You can see an example of this below.

    -
      - Cat
      - Dog
      - Goldfish

If you wish to nest your sequences even deeper, you just need to add more levels.

    -
     -
      - Cat
      - Dog
      - Goldfish

Sequences can be added to other data structure types, such as mappings or scalars.

#### Mappings
Mapping gives you the ability to list keys with values. This is useful in cases where you are assigning a name or a property to a specific element.

    animal: pets

This example maps the value of pets to the animal key. When used in conjunction with a sequence, you can see that you are starting to build a list of pets. In the following example, the dash used to label each item counts as indentation, making the line items the child and the mapping line animal: pets the parent.

    animal: pets
    - Cat
    - Dog
    - Goldfish

### Resources and Further Documentation
For more information about YAML, including detailed documentation about how it works, check out the resources linked below.

* [Dave's YAML Primer](https://github.com/darvid/trine/wiki/YAML-Primer)
* [Official YAML 1.2 Documentation](http://www.yaml.org/spec/1.2/spec.html)
* [YAML Reference Card](http://www.yaml.org/refcard.html)
* [Xavier Shay's YAML Tutorial](http://www.yaml.org/refcard.html)