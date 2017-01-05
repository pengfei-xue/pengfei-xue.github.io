---
layout: post
title: 'bash note'
date: 2017-01-05
author: Pengfei.X
version: 0.1
categories: [devnotes, ]
---


## sort numerically

    sort -t '_' -k 2n

    -t '_' (sets the delimiter to the underscore character)
    -k 2n (sorts by the second column using numeric ordering)



## How to POST JSON data with Curl from Terminal/Commandline to Test Spring REST?

    curl -H "Content-Type: application/json" -X POST -d '{"username":"xyz","password":"xyz"}' http://localhost:3000/api/login


## How do I split a string on a delimiter in Bash?

    a="foo:bar"
    b=${a%:*}
    c=${a##*:}
    you will get:

    b = foo
    c = bar

[How do I split a string on a delimiter in Bash](http://stackoverflow.com/questions/918886/how-do-i-split-a-string-on-a-delimiter-in-bash)
[Manipulating Strings](http://tldp.org/LDP/abs/html/string-manipulation.html)


## comparison

[Other Comparison Operators](http://tldp.org/LDP/abs/html/comparison-ops.html)


## array

[Array variables](http://tldp.org/LDP/Bash-Beginners-Guide/html/sect_10_02.html)
[arry loop](http://www.cyberciti.biz/faq/bash-for-loop-array/)


## function

[functions](http://tldp.org/LDP/abs/html/functions.html)

## Automatic exit from bash shell script on error

Use the set -e builtin:

    #!/bin/bash
    set -e

Any subsequent commands which fail will cause the shell script to exit immediately
Alternatively, you can pass -e on the command line:

    bash -e my_script.sh

You can also disable this behavior with set +e.

[Automatic exit from bash shell script on error](http://stackoverflow.com/questions/2870992/automatic-exit-from-bash-shell-script-on-error)


## books
- [Advanced Bash-Scripting Guide](http://tldp.org/LDP/abs/html/index.html)
