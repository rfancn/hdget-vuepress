---
date: '2017-04-13 10:21:07'
draft: false
lastmod: '2017-11-25 20:16:11'
categories:
- 博客
tags:
- linux
title: How to use crash to check the vmcore
---

It is very important to check the vmcore file when the server gets hang there or crash for unknown reason, following tips give some help on how to check the vmcore, hope it helps.

#### If you have a vmcore-incomplete that can't be analyzed, try to get the dmesg buffer out using: ###

    #crash --minimal vmlinux vmcore-incomplete
    crash> log

#### Display a task's kernel stack backtrace ###

    crash> bt
    PID: 3523   TASK: ffff81001c0ac7e0 CPU: 0   COMMAND: "bash"
    #0 [ffff81000e839d68] crash_kexec at ffffffff800a9583
    #1 [ffff81000e839df0] crash_kexec at ffffffff800a959f
    #2 [ffff81000e839e78] crash_kexec at ffffffff800a9583
    #3 [ffff81000e839ea0] do_filp_open at ffffffff80026f20
    #4 [ffff81000e839eb0] sysrq_handle_crashdump at ffffffff80191908
    #5 [ffff81000e839ec0] __handle_sysrq at ffffffff80191705
    #6 [ffff81000e839f00] write_sysrq_trigger at ffffffff800f6e66
    #7 [ffff81000e839f10] vfs_write at ffffffff80016121
    #8 [ffff81000e839f40] sys_write at ffffffff800169b2
    #9 [ffff81000e839f80] tracesys at ffffffff8005b2c1

    RIP: 000000335a6bfa90  RSP: 00007fffacb3ee88  RFLAGS: 00000246
    RAX: ffffffffffffffda  RBX: ffffffff8005b2c1  RCX: ffffffffffffffff
    RDX: 0000000000000002  RSI: 00002aaaadfbf000  RDI: 0000000000000001
    RBP: 0000000000000002   R8: 00000000ffffffff   R9: 00002aaaaaac5db0
    R10: 0000000000000072  R11: 0000000000000246  R12: 000000335a948760
    R13: 00002aaaadfbf000  R14: 0000000000000002  R15: 0000000000000000
    ORIG_RAX: 0000000000000001  CS: 0033  SS: 002b
    <!--more-->

#### Disassemble memory ###

    crash> bt
    PID: 8206   TASK: ffff8100153647a0  CPU: 1   COMMAND: "fio"
    #0 [ffff81002f1c9be0] crash_kexec at ffffffff800a959f
    #1 [ffff81002f1c9c68] __aio_put_req at ffffffff800de170
    #2 [ffff81002f1c9ca0] __die at ffffffff80062e9d
    #3 [ffff81002f1c9ce0] die at ffffffff80069457
    #4 [ffff81002f1c9d10] do_invalid_op at ffffffff80069a0d
    #5 [ffff81002f1c9d28] __aio_put_req at ffffffff800de170
    #6 [ffff81002f1c9d60] __generic_file_aio_write_nolock at ffffffff80015c5a
    #7 [ffff81002f1c9dd0] error_exit at ffffffff8005be1d
    [exception RIP: __aio_put_req+39]
    RIP: ffffffff800de170  RSP: ffff81002f1c9e88  RFLAGS: 00010096
    RAX: 0000000000000000  RBX: ffff810033c03e80  RCX: 0000000000000000
    RDX: 00000000ffffffff  RSI: ffff810033c03e80  RDI: ffff81002d3c5e80
    RBP: ffff81002d3c5e80   R8: 000000002d39b000   R9: ffff810009001000
    R10: ffff810033c03e80  R11: 0000000000000206  R12: ffff810031fe0cc0
    R13: ffff81002d3c5e80  R14: 000000000755e4e0  R15: 0000000000000005
    ORIG_RAX: ffffffffffffffff  CS: 0010  SS: 0018
    #8 [ffff81002f1c9ea0] aio_put_req at ffffffff800def5f
    #9 [ffff81002f1c9ec0] io_submit_one at ffffffff800df34d
    #10 [ffff81002f1c9ef0] sys_io_submit at ffffffff800df874
    #11 [ffff81002f1c9f80] tracesys at ffffffff8005b2c1
    RIP: 00002aaaaaac6647  RSP: 00007fff438f1f18  RFLAGS: 00000206
    RAX: ffffffffffffffda  RBX: ffffffff8005b2c1  RCX: ffffffffffffffff
    RDX: 0000000007566c70  RSI: 000000000000003d  RDI: 00002aaaaee06000
    RBP: 000000000750f3f0   R8: 00000000000e032d   R9: 00000000214f4bcc
    R10: 0000000000000000  R11: 0000000000000206  R12: 000000000000003d
    R13: 0000000007568c80  R14: 0000000007566c70  R15: 000000000000003d
    ORIG_RAX: 00000000000000d1  CS: 0033  SS: 002b

Backtrace line #8 shows aio_put_req() made a call or was interrupted at ffffffff800def5f. We can disassemble aio_put_req():

    crash> dis aio_put_req
    0xffffffff800def3e <aio_put_req>:       push   %rbp
    0xffffffff800def3f <aio_put_req+1>:     push   %rbx
    0xffffffff800def40 <aio_put_req+2>:     mov    %rdi,%rbx
    0xffffffff800def43 <aio_put_req+5>:     sub    $0x8,%rsp
    0xffffffff800def47 <aio_put_req+9>:     mov    0x28(%rdi),%rbp
    0xffffffff800def4b <aio_put_req+13>:    lea    0x38(%rbp),%rdi
    0xffffffff800def4f <aio_put_req+17>:    callq  0xffffffff80062625 <_spin_lock_irq>
    0xffffffff800def54 <aio_put_req+22>:    mov    %rbx,%rsi
    0xffffffff800def57 <aio_put_req+25>:    mov    %rbp,%rdi
    0xffffffff800def5a <aio_put_req+28>:    callq

    0xffffffff800de149 <__aio_put_req>
    0xffffffff800def5f <aio_put_req+33>:    mov    �x,�x
    ...

Here is the disassembly of aio_put_req():

    crash> dis __aio_put_req
    0xffffffff800de149 <__aio_put_req>:     push   %r12
    0xffffffff800de14b <__aio_put_req+2>:   push   %rbp
    0xffffffff800de14c <__aio_put_req+3>:   mov    %rdi,%rbp
    0xffffffff800de14f <__aio_put_req+6>:   push   %rbx
    0xffffffff800de150 <__aio_put_req+7>:   mov    0x38(%rdi),$0x
    0xffffffff800de153 <__aio_put_req+10>:  mov    %rsi,%rbx
    0xffffffff800de156 <__aio_put_req+13>:  test   $0x,$0x
    0xffffffff800de158 <__aio_put_req+15>:  jle
    0xffffffff800de164 <__aio_put_req+27>
    0xffffffff800de15a <__aio_put_req+17>:  ud2a
    0xffffffff800de15c <__aio_put_req+19>:  pushq  $0xffffffff802846d6
    0xffffffff800de161 <__aio_put_req+24>:  retq   $0x1f5
    0xffffffff800de164 <__aio_put_req+27>:  mov    0x18(%rbx),$0x
    0xffffffff800de167 <__aio_put_req+30>:  dec    $0x
    0xffffffff800de169 <__aio_put_req+32>:  test   $0x,$0x
    0xffffffff800de16b <__aio_put_req+34>:  mov    $0x,0x18(%rbx)
    0xffffffff800de16e <__aio_put_req+37>:  jns
    0xffffffff800de17a <__aio_put_req+49>
    0xffffffff800de170 <__aio_put_req+39>:  ud2a
    0xffffffff800de172 <__aio_put_req+41>:  pushq  $0xffffffff802846d6
    0xffffffff800de177 <__aio_put_req+46>:  retq   $0x1f9
    0xffffffff800de17a <__aio_put_req+49>:  xor    $0x,$0x

Note the ud2a opcode at aio_put_req+39 (0xffffffff800de170). This is an invalid opcode that BUG() uses to trigger an exception. A pointer to the filename followed by a line number are added after the ud2a:

    crash> rd 0xffffffff802846d6
    ffffffff802846d6:  632e6f69612f7366                    fs/aio.c
    crash> p 0x1f9
    $2 = 505

```p 0x1f9``` converts the hex line number to decimal 505.The dis command has a -l option which displays source code line numnbers but they are not always available.