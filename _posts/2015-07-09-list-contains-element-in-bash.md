---
layout: post
title: List contains element in bash
date: 2015-07-09 15:01
author: arenhage
comments: true
categories: [bash, contains, Home, list]
---
I just came across this nice way of verifying if a list contains a specific element in bash.


```bash
containsElement () { for e in "${@:2}"; do [[ "$e" = "$1" ]] && return 0; done; return 1; }
a_list=("foo" "bar")

if containsElement "$1" "${a_list[@]}"; then
echo "$1 found in a_list!"
fi
```

kudos to the op of this solution : <a href="http://stackoverflow.com/questions/3685970/check-if-an-array-contains-a-value" target="_blank">stackoverflow</a>
