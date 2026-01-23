---
tags:
  - development
  - notion
created: 2025-01-23
updated: 2025-01-23
status: active
source: notion
---

# Development Issues

## Front-End

### Material Design

framework

[http://daemonite.github.io/material/docs/4.1/layout/grid/](http://daemonite.github.io/material/docs/4.1/layout/grid/)

Icon

[https://material.io/resources/icons/?style=baseline](https://material.io/resources/icons/?style=baseline)

[https://github.com/Angelmmiguel/material_icons](https://github.com/Angelmmiguel/material_icons)

### Framework

[https://tailwindcss.com/](https://tailwindcss.com/)

## Backend

### RESTFul API

[Best Practices for Designing a Pragmatic RESTful API](https://www.vinaysahni.com/best-practices-for-a-pragmatic-restful-api)

[你的REST不是REST？](https://www.ithome.com.tw/voice/128528)

[The Hypertext Application Language](https://stateless.co/hal_specification.html)

[HATEOAS - 維基百科，自由的百科全書](https://zh.wikipedia.org/zh-tw/HATEOAS)

## Rails Development

## Shell

set variable when unset

[https://unix.stackexchange.com/questions/594841/how-do-i-assign-a-value-to-a-bash-variable-iff-that-variable-is-null-unassigned](https://unix.stackexchange.com/questions/594841/how-do-i-assign-a-value-to-a-bash-variable-iff-that-variable-is-null-unassigned)

Split String and Get the Last Field

[How to split a string in shell and get the last field](https://stackoverflow.com/questions/3162385/how-to-split-a-string-in-shell-and-get-the-last-field)

String Operators

[Variable Mangling in Bash with String Operators | Linux Journal](https://www.linuxjournal.com/article/8919)

You can use [string operators](http://www.linuxjournal.com/article/8919):

```bash
$ foo=1:2:3:4:5
$ echo ${foo##*:}
5

```

This trims everything from the front until a ':', greedily.

```bash
${foo  <-- from variable foo
  ##   <-- greedy front trim
  *    <-- matches anything
  :    <-- until the last ':'
 }
```

Remove everything after hyphen

```bash
line="This-is-a-filename-0001.jpg"
echo "${line%-*}" # prints: This-is-a-filename

```

The `%-*`operator removes all beginning with the last hyphen.