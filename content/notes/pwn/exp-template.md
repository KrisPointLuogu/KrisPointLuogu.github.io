---
title: exp 模板
date: 2026-11-16
tags:
  - pwn
  - pwntools
---
```python
from pwn import *

myarch = "amd64"
myos = "linux"

context(arch=myarch, os=myos, log_level='debug')

binary = './a.out'
p = process(binary)
# gdb.attach(p)

payload = 0
print(payload)

# p.sendline(payload)

p.interactive()
```
