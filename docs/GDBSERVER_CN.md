# Qiling Gdbserver

Qiling已通过Qiling Gdbserver支持**Gdb远程调试**（目前支持x86, x86-64）。

## Usage

```python
from qiling import *

def test_gdb(path, rootfs):
    ql = Qiling(path, rootfs, output="off")
    ql.gdb = True # enable gdbserver at default port 9999
    # You can also customize address & port of GDB server
    # ql.gdb = ":9999"  # Listening to 0.0.0.0:9999
    # ql.gdb = "127.0.0.1:9999"  # Listening to 127.0.0.1:9999
    ql.run()  

if __name__ == "__main__":
    test_gdb(["../examples/rootfs/x8664_linux/bin/x8664_hello_static"], "../examples/rootfs/x8664_linux")
```

运行gdb并为所需架构进行设置，x86输入命令`set architecture i386:intel`，x86-64输入命令`set architecture i386:x86-64`

然后输入命令`target remote ip:port` 并执行python脚本

当前支持的命令有 `si`, `ni`, `c`, `k`, 读寄存器, 读内存, 插入或删除断点

---

## Example

以x86-64为例

**set arch**
```bash
(gdb) set architecture i386:x86-64
The target architecture is assumed to be i386:x86-64
```

**connect**
```bash
(gdb) target remote localhost:9999
Remote debugging using localhost:9999
warning: No executable has been specified and target does not support
determining executable automatically.  Try using the "file" command.
0x00000000004014e0 in ?? ()
```
**next/step**

```bash
(gdb) disas 0x00000000004014e0,0x00000000004014ff
Dump of assembler code from 0x4014e0 to 0x4014ff:
=> 0x00000000004014e0:  sub    $0x28,%rsp
   0x00000000004014e4:  mov    0x2f55(%rip),%rax        # 0x404440
   0x00000000004014eb:  movl   $0x0,(%rax)
   0x00000000004014f1:  callq  0x401640
   0x00000000004014f6:  callq  0x401180
   0x00000000004014fb:  nop
   0x00000000004014fc:  nop
   0x00000000004014fd:  add    $0x28,%rsp
End of assembler dump.
(gdb) ni
0x00000000004014e4 in ?? ()
(gdb)
0x00000000004014eb in ?? ()
(gdb)
0x00000000004014f1 in ?? ()
(gdb) si
0x0000000000401640 in ?? ()
(gdb) disas 0x0000000000401640,0x000000000040164f
Dump of assembler code from 0x401640 to 0x40164f:
=> 0x0000000000401640:  push   %r12
   0x0000000000401642:  push   %rbp
   0x0000000000401643:  push   %rdi
   0x0000000000401644:  push   %rsi
   0x0000000000401645:  push   %rbx
   0x0000000000401646:  sub    $0x30,%rsp
   0x000000000040164a:  mov    0x1a5f(%rip),%rbx        # 0x4030b0
End of assembler dump.
```

**add/remove breakpoint**
```bash
(gdb) break *0x0000000000401645
Breakpoint 1 at 0x401645
(gdb) i b
Num     Type           Disp Enb Address            What
1       breakpoint     keep y   0x0000000000401645
(gdb) c
Continuing.

Breakpoint 1, 0x0000000000401645 in ?? ()
(gdb) del 1
(gdb) i b
No breakpoints or watchpoints.
```

**read register/memory**
```bash
(gdb) i r
rax            0x407610            4224528
rbx            0x0                 0
rcx            0x0                 0
rdx            0x0                 0
rsi            0x0                 0
rdi            0x0                 0
rbp            0x80000001d000      0x80000001d000
rsp            0x80000001cfb0      0x80000001cfb0
r8             0x0                 0
r9             0x0                 0
r10            0x0                 0
r11            0x0                 0
r12            0x0                 0
r13            0x0                 0
r14            0x0                 0
r15            0x0                 0
rip            0x401645            0x401645
eflags         0x14                [ PF AF ]
cs             0x0                 0
ss             0x0                 0
ds             0x0                 0
es             0x0                 0
fs             0x0                 0
gs             0x0                 0
x/10xg
(gdb) x/10xg 0x0000000000401645
0x401645:       0x1d8b4830ec834853      0xa232b84800001a5f
0x401655:       0x394800002b992ddf      0x0000202444c748c3
0x401665:       0x48d3f74817740000      0x834800001a4d1d89
0x401675:       0x5c415d5f5e5b30c4      0x20244c8d489066c3
0x401685:       0x8b4800006b8115ff      0x006b5615ff202474
```

---

## Todo

1. 添加gdbserver对更多架构的支持

2. 添加对更多GDB指令的支持

3. ...

---

## Thank

编写该模块时从 [uDdbg](https://github.com/iGio90/uDdbg) 借鉴了很多思想，在此感谢
