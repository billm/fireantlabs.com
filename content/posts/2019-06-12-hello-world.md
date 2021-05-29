---
title: "Hello World"
date: 2019-06-12T20:19:55-04:00
year: "2019"
month: "2019/06"
toc: false
comments: true
images:
tags:
draft: false
---

A proper Hello World in shellcode.

<!--more-->


```c
#include <stdio.h>
#include <string.h>
 
char *shellcode = "\xeb\x19\x59\x31\xc0\xb0\x04\x31\xdb\xb3\x01\x31\xd2\xb2"
                  "\x0d\xcd\x80\x31\xc0\xb0\x01\x31\xdb\xb3\x05\xcd\x80\xe8"
                  "\xe2\xff\xff\xff\x48\x65\x6c\x6c\x6f\x20\x57\x6f\x72\x6c"
                  "\x64\x21\x0a";

int main(void)
{
  (*(void(*)()) shellcode)();
  return 0;
}
```

```bash
$ ./hello 
Hello World!
```


