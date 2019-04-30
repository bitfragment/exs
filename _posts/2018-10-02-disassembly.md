---
title: "Disassembly"
date: 2018-10-02
tags: c, core
published: true
---

Source: [CS:APP3e, Bryant and O'Hallaron](http://csapp.cs.cmu.edu/), 
Binary Bomb Lab (from [Lab Assignments](http://csapp.cs.cmu.edu/3e/labs.html)), 
using the downloadable self-study version.

The task is to disassemble and "reverse engineer" a binary file, using
a debugger to determine the content of the six strings the program
expects as console input.

Only the first four phases are defused here. Not included are the fifth 
and six phases, along with the final "secret phase."

* TOC
{:toc}


Preliminary steps
-----------------

1. Disassemble `​bomb` with `objdump -d bomb > bomb.s`
2. Read through `​bomb.s`​. The section for the `main` function contains sequences of `call` instructions like this:
    1. `read_line` (taking user input for each phase)
    2. `phase_`*n* where *n* = 1..6 (testing user input against each bomb phase)
    3. `phase_defused` (if successful)
3. Each `​read_line`​ instruction is followed by the instruction `movq %rax, %rdi`​, which copies the return value of `​read_line`​ into the register `​%rdi`.
4. The section for the `​main`​ function is followed by sections for the individual phase functions `phase_`*n* where *n* = 1..6.


Notes on using gdb
------------------

* `run`​ to run the program inside gdb and stop at the first breakpoint
* Advance to a specific instruction with `​advance *<addr>`​, where \<addr\> is an instruction address in hex form
* Step one instruction at a time with `​stepi`​
* Step over a procedure call with `​nexti`​
* Continue to a specified instruction with ​`until <address>`
* Skip to end of a procedure that you're already inside with `​finish`
* Continue to next breakpoint with `​continue`
* Useful to set `​layout asm`​ or use `​display/i $pc`​ for viewing machine instructions while stepping through them
* To display data at an address as a sequence of characters, cast the data to a char pointer: `​print (char *) <address>`​
* Inspect register contents with `​print $<reg>`​ or `​x ​$<reg>`
* Look for arguments to `​sscanf()`​ on the stack (%rsp with varying offsets)
* To check on the results of comparisons before stepping to the jump instructions following them, inspect flags with `​print $eflags`​


**Function `phase_1`** (calling `strings_not_equal`)
--------------------------------------------------------

`phase_1`, as disassembled with `​objdump -d`:

```asm
0000000000400ee0 <phase_1>:
  400ee0:       48 83 ec 08             sub    $0x8,%rsp
  400ee4:       be 00 24 40 00          mov    $0x402400,%esi
  400ee9:       e8 4a 04 00 00          callq  401338 <strings_not_equal>
  400eee:       85 c0                   test   %eax,%eax
  400ef0:       74 05                   je     400ef7 <phase_1+0x17>
  400ef2:       e8 43 05 00 00          callq  40143a <explode_bomb>
  400ef7:       48 83 c4 08             add    $0x8,%rsp
  400efb:       c3                      retq
```

### Discovery

1. We already have the stdin char input stored in register `​%rdi`.
2. The instruction `movl $0x402400, %esi` copies a constant value representing a memory address to register `​%esi`.
3. The next instruction is `callq 401338 <strings_not_equal>`. Looking at `strings_not_equal`, we see that when passed the stdin char data stored in `​%rdi`​ and the data stored in `​%esi`​ as arguments, `​strings_not_equal`​ sets `​%eax`​ to 1 if its arguments are not equal and 0 if its arguments are equal.
4. The next instruction is `test %eax, %eax` which sets `ZF` (zero flag) to 1 if `​%eax`​ contains 0.
5. The next instruction is `je 400ef7 <phase_1+0x17>` which jumps to 400ef7 if ZF is 1\. That is, if `​strings_not_equal`​ returned 0\. That is, if the strings are equal. So if the strings are equal, we are jumping to 400ef7​, thereby jumping over the instruction that follows, which is `callq 40143a <explode_bomb>`.
6. To discover the string that is expected, start `​gdb` with `gdb bomb` and set a breakpoint with `​break phase_1`​.
7. We need to print the character representation of data at the memory address $0x402400 that was copied to %esi. We don't need to run anything to do this. Do it with the `print` command, casting the data to a char pointer: `print (char *) 0x402400`. The result is `Border relations with Canada have never been better`.
8. **Therefore, the expected string is "Border relations with Canada have never been better."**

### Verification

1. Confirm this by checking that this string appears in the output of the shell command `​strings bomb`​.
2. Now `​run`​. The breakpoint will stop execution after we enter the string `Border relations with Canada have never been better.`
3. Check data at address $0x402400​ with `print (char *) 0x402400`​. It should print `Border relations with Canada have never been better.`
4. Check data in %rdi: `print (char *) $rdi` should display `Border relations with Canada have never been better.`
5. Now set `​layout asm`​ or `display/i $pc` to display each instruction and step in with stepi.
6. After two steps, check that `Border relations with Canada have never been better.` was copied to `%esi`, using `print (char *) $esi`.
7. You should be at the instruction calling `​strings_not_equal`.
8. Step over `​strings_not_equal`​ with `​nexti (`or `​finish`​ if you're already inside it)​.
9. Print the value of `​%eax`​ with `​print $eax`​. It should display 0, meaning that the strings were evaluated as equal.
10. Continue with `​c`. You should see the message `Phase 1 defused. How about the next one?`

### Storing the correct strings in a text file for convenience

1. Exit with `Ctrl-c`, then exit `gdb` with `q.` Add `Border relations with Canada have never been better.` to a file named `psol.txt`, which can be passed as a command line argument to `bomb`.
2. You can run `./bomb psol.txt`​ whenever you like, though the defusing of phase 1 has already been recorded on the scoreboard.



**Function `phase_2`** (calling `read_six_numbers`)
---------------------------------------------------

`phase_2`, as disassembled with `​objdump -d`:


```asm
0000000000400efc <phase_2>:
  400efc:       55                      push   %rbp
  400efd:       53                      push   %rbx
  400efe:       48 83 ec 28             sub    $0x28,%rsp
  400f02:       48 89 e6                mov    %rsp,%rsi
  
  400f05:       e8 52 05 00 00          callq  40145c <read_six_numbers> ; get six ints from stdin
  400f0a:       83 3c 24 01             cmpl   $0x1,(%rsp)               ; compare first int to 1
  400f0e:       74 20                   je     400f30 <phase_2+0x34>     ; if 1, jump
  400f10:       e8 25 05 00 00          callq  40143a <explode_bomb>     ; else explode
  400f15:       eb 19                   jmp    400f30 <phase_2+0x34>     ; (but continue evaluating)
  400f17:       8b 43 fc                mov    -0x4(%rbx),%eax           ; %eax = previous int
  400f1a:       01 c0                   add    %eax,%eax                 ; double the val of prev int
  400f1c:       39 03                   cmp    %eax,(%rbx)               ; compare result to curr int
  400f1e:       74 05                   je     400f25 <phase_2+0x29>     ; if equal, jump
  400f20:       e8 15 05 00 00          callq  40143a <explode_bomb>     ; else explode
  400f25:       48 83 c3 04             add    $0x4,%rbx                 ; %rbx += 4 = next int
  400f29:       48 39 eb                cmp    %rbp,%rbx                 ; are we past the 6th int?
  400f2c:       75 e9                   jne    400f17 <phase_2+0x1b>     ; continue
  400f2e:       eb 0c                   jmp    400f3c <phase_2+0x40>     ; else return
  
                                                                         ; HANDLE BOUNDS:
  400f30:       48 8d 5c 24 04          lea    0x4(%rsp),%rbx            ; second int's address
  400f35:       48 8d 6c 24 18          lea    0x18(%rsp),%rbp           ; address AFTER sixth int
  400f3a:       eb db                   jmp    400f17 <phase_2+0x1b>     
  
  400f3c:       48 83 c4 28             add    $0x28,%rsp                ; RETURN
  400f40:       5b                      pop    %rbx
  400f41:       5d                      pop    %rbp
```

`read_six_numbers`, as disassembled with `objdump -d`:

```asm
000000000040145c <read_six_numbers>:
  40145c:       48 83 ec 18             sub    $0x18,%rsp
  401460:       48 89 f2                mov    %rsi,%rdx
  401463:       48 8d 4e 04             lea    0x4(%rsi),%rcx
  401467:       48 8d 46 14             lea    0x14(%rsi),%rax
  40146b:       48 89 44 24 08          mov    %rax,0x8(%rsp)
  401470:       48 8d 46 10             lea    0x10(%rsi),%rax
  401474:       48 89 04 24             mov    %rax,(%rsp)
  401478:       4c 8d 4e 0c             lea    0xc(%rsi),%r9
  40147c:       4c 8d 46 08             lea    0x8(%rsi),%r8
  401480:       be c3 25 40 00          mov    $0x4025c3,%esi ; copy "%d %d %d %d %d %d" to %esi
  401485:       b8 00 00 00 00          mov    $0x0,%eax      ; start counter at 0
  40148a:       e8 61 f7 ff ff          callq  400bf0 <__isoc99_sscanf@plt> ; sscanf() passing %esi
  40148f:       83 f8 05                cmp    $0x5,%eax      ; compare 5 to counter
  401492:       7f 05                   jg     401499 <read_six_numbers+0x3d> ; if 5 > counter, recurse
  401494:       e8 a1 ff ff ff          callq  40143a <explode_bomb> ; explode upon user mistake
  401499:       48 83 c4 18             add    $0x18,%rsp
  40149d:       c3                      retq
```

### Discovery

1. Start by looking at `​read_six_numbers. At 401480`, `mov $0x4025c3,%esi `copies data at the constant memory address $0x4025c3 to `​%esi,` presumably to be used as an argument to `​sscanf()`​.
2. We want to see what that argument looks like. Print its character representation: `print (char *) 0x4025c3`​. The result is `​"%d %d %d %d %d %d"`, which tells us that `​sscanf()`​ expects six integers separated by spaces.
3. Now look at `​phase_2`​ again. If six integers were entered and the bomb did not explode, `read_six_numbers` has placed data representing those six numbers on the stack.
4. At 400f0a, the first integer is compared to 1\. If it's not 1, the bomb explodes; otherwise, we jump to 400f30.
5. **Therefore, the first integer must be 1.**
6. At 400f30, the address of the next integer is copied to `​%rbx`​ by loading with a 4-byte offset.
7. At 400f35, the 18-byte offset means that the address *after* the last of the six integers is copied to `​%rbp`​.
8. Next is a jump back to 400f17, where the -4-byte offset of the address of `​%rbx` means we are dealing with the first integer again, copying it to `​%eax.`
9. From 400f17 to 400f1e, the first int is doubled and the result is compared to the second int. If these are equal, we jump to 400f25; otherwise, the bomb explodes.
10. **Therefore, the second integer must be a value equivalent to twice the first integer.**
11. At 400f25, we increment `​`the address in `%rbx` by 4, giving us the address of the next number.
12. From 400f25 to 400f2e, we check to see if we've finished evaluating all six numbers. Clearly this is a loop.
13. Since we jump back to 400f17 whenever we continue, this means that each number must be equivalent to twice the previous number.
14. **Therefore, the expected string is "1 2 4 8 16 32".**

### Verification

1. Start `​gdb` with `gdb bomb,` set a breakpoint with `​break phase_2`, and `run.` The breakpoint will stop execution after we enter the string "1 2 4 8 16 32".
2. Inspect register `​%rbx​` with `​print $rbx`. Its contents should be 0.
3. Set `layout asm`​ or `display/i $pc` and skip ahead to 0x400f0a with `advance *0x400f0a`. The value in `%rbx` should still be 0.
4. Inspect `​%rsp`​ with `​x $rsp`​. The result should be 1, the first integer entered by the user.
5. Step three times using `​stepi`​ and check the value of `​%rbx` using `x $rbx`. It should now be 2.
6. Continue stepping and check the values of `​%rsp` and `​%rbx`​, which should each be incremented to 4, then 16, then 32.
7. Continue with `​c`. You should see the message `That's number 2.  Keep going!`



**Function `​phase_3`**
----------------------

`phase_3`, as disassembled with `​objdump -d`:

```asm
0000000000400f43 <phase_3>:
  400f43:       48 83 ec 18             sub    $0x18,%rsp
  400f47:       48 8d 4c 24 0c          lea    0xc(%rsp),%rcx
  400f4c:       48 8d 54 24 08          lea    0x8(%rsp),%rdx
  400f51:       be cf 25 40 00          mov    $0x4025cf,%esi           ; "%d %d"
  400f56:       b8 00 00 00 00          mov    $0x0,%eax
  400f5b:       e8 90 fc ff ff          callq  400bf0 <__isoc99_sscanf@plt>
  400f60:       83 f8 01                cmp    $0x1,%eax                ; if at least 2 integers
  400f63:       7f 05                   jg     400f6a <phase_3+0x27>    ;   entered, continue
  400f65:       e8 d0 04 00 00          callq  40143a <explode_bomb>
  400f6a:       83 7c 24 08 07          cmpl   $0x7,0x8(%rsp)           ; 0 <= first int <= 7
  400f6f:       77 3c                   ja     400fad <phase_3+0x6a>
  400f71:       8b 44 24 08             mov    0x8(%rsp),%eax           ; copy first int to %eax
  400f75:       ff 24 c5 70 24 40 00    jmpq   *0x402470(,%rax,8)       ; jump to computed address
  400f7c:       b8 cf 00 00 00          mov    $0xcf,%eax
  400f81:       eb 3b                   jmp    400fbe <phase_3+0x7b>
  400f83:       b8 c3 02 00 00          mov    $0x2c3,%eax
  400f88:       eb 34                   jmp    400fbe <phase_3+0x7b>
  400f8a:       b8 00 01 00 00          mov    $0x100,%eax
  400f8f:       eb 2d                   jmp    400fbe <phase_3+0x7b>
  400f91:       b8 85 01 00 00          mov    $0x185,%eax
  400f96:       eb 26                   jmp    400fbe <phase_3+0x7b>
  400f98:       b8 ce 00 00 00          mov    $0xce,%eax
  400f9d:       eb 1f                   jmp    400fbe <phase_3+0x7b>
  400f9f:       b8 aa 02 00 00          mov    $0x2aa,%eax
  400fa4:       eb 18                   jmp    400fbe <phase_3+0x7b>
  400fa6:       b8 47 01 00 00          mov    $0x147,%eax
  400fab:       eb 11                   jmp    400fbe <phase_3+0x7b>
  400fad:       e8 88 04 00 00          callq  40143a <explode_bomb>
  400fb2:       b8 00 00 00 00          mov    $0x0,%eax
  400fb7:       eb 05                   jmp    400fbe <phase_3+0x7b>
  400fb9:       b8 37 01 00 00          mov    $0x137,%eax
  400fbe:       3b 44 24 0c             cmp    0xc(%rsp),%eax           ; second int must be a computed
  400fc2:       74 05                   je     400fc9 <phase_3+0x86>    ;   value based on the first
  400fc4:       e8 71 04 00 00          callq  40143a <explode_bomb>
  400fc9:       48 83 c4 18             add    $0x18,%rsp
  400fcd:       c3                      retq
```

1. The instruction at 400f51 copies data to %esi which will be passed to C `​sscanf()`​ at 400f5b. That will be the format string that `​sscanf()`​ expects, which will tell us about the form of input it expects. In gdb, `print (char *) 0x4025cf` yields `"%d %d"`​, so we know that `phase_3` expects two integers separated by a space.
2. ​The return value of `sscanf()`​ is the number of variables filled. Therefore, at 400f60, after the call to `​sscanf()`, %eax will hold a value representing the number of integers we entered (up to 2, as specified by the format string "%d %d"). Check this by setting a breakpoint at `​phase_3`​, then typing `​run`​, and when we get to `phase_3` entering "0 1" as a test string. Set `layout asm`​ or `display/i $pc`, advance to 400f60, and inspect %eax with `​p $eax`​. The result is 2\. Verify this by running again and testing with only 1 integer.
3. Instructions from 400f60–400f65 tell us that if the user does not enter more than one integer, the bomb explodes.
4. Otherwise, we jump to 400f6a, where we find the first of two `​cmp`​ instructions in the rest of the function. Inspecting 0x8(%rsp) with `​x $rsp+0x8`​, we get 0x00000000, which is presumably the first integer we entered, pushed onto the stack. Inspecting 0xc(%rsp) at 400fbe with `​x $rsp+0xc`​, we get 0x00000001, presumably the second integer we entered, pushed onto the stack.
5. Instructions 400f6a–400f6f tell us that that if our first integer is greater than 7, the bomb explodes.
6. **Therefore, our first integer must be less than or equal to 7\. Since `​ja`​ interprets CF and ZF as if the compared values are unsigned, this means that our first integer must also be equal to or greater than zero.**
7. We arrive at 400f71–400f75, where the first integer is copied to %eax and a jump address is computed: `*0x402470(,%rax,8)` means 8 \* %rax + 0x402470\. Get the value in %rax with `p %rax`​ (it's 0, the first integer we entered, which was copied to %eax at 400f71) and plug it in: 8 \* 0 + 0x402470 = 0x402470\. *This is an address at which another address is stored, as a value. Get the stored value representing an address with `x 0x402470`​; the result is 0x00400f7c. This is the address inside `​phase_3` to which we will jump.
8. If our first integer was 0, the computed address is the very next instruction. There, the constant value $0xcf is copied to to %eax and we jump to 400fbe, where the second integer we entered is compared to %eax. To find out what value is in %eax at this point, advance to 400fbe and inspect it. It should be 207\. If our second integer is equal to 207, we jump and return; otherwise the bomb explodes.
9. **Therefore, if our first integer is 0, then our second integer must be 207.**
10. *However: the jump address computed at 400f75 will be different depending on whether the first integer entered is 0, 1, 2, 3, 4, 5, 6, or 7, and the computations in 400f7c–400fa6 will produce a different value for the second integer depending on what the first is. Our second integer must be 207 if the first is 0; 311 if the first is 1; 707 if the first is 2; and so on.
11. **Therefore, the expected string is "a b", where 0 \<= *a* \<= 7 and *b* is one of the corresponding values [207, 311, 707...].**



**Function `phase_4`** (calling `func4`)
----------------------------------------

`phase_4`, as disassembled with `​objdump -d`:

```asm
000000000040100c <phase_4>:
  40100c:       48 83 ec 18             sub    $0x18,%rsp
  401010:       48 8d 4c 24 0c          lea    0xc(%rsp),%rcx
  401015:       48 8d 54 24 08          lea    0x8(%rsp),%rdx
  40101a:       be cf 25 40 00          mov    $0x4025cf,%esi        ; "%d %d"
  40101f:       b8 00 00 00 00          mov    $0x0,%eax
  401024:       e8 c7 fb ff ff          callq  400bf0 <__isoc99_sscanf@plt>
  401029:       83 f8 02                cmp    $0x2,%eax             ; 2 arguments required
  40102c:       75 07                   jne    401035 <phase_4+0x29>
  40102e:       83 7c 24 08 0e          cmpl   $0xe,0x8(%rsp) ;      ; first int...
  401033:       76 05                   jbe    40103a <phase_4+0x2e> ; ...must be <=14 (treated as unsigned)
  401035:       e8 00 04 00 00          callq  40143a <explode_bomb>
  40103a:       ba 0e 00 00 00          mov    $0xe,%edx
  40103f:       be 00 00 00 00          mov    $0x0,%esi
  401044:       8b 7c 24 08             mov    0x8(%rsp),%edi
  401048:       e8 81 ff ff ff          callq  400fce <func4>        ; call with args: 14, 0, first int
  40104d:       85 c0                   test   %eax,%eax             ; if %eax == 0, ZF == 1
  40104f:       75 07                   jne    401058 <phase_4+0x4c> ; if ZF == 0, explode
  401051:       83 7c 24 0c 00          cmpl   $0x0,0xc(%rsp)        ; if second int == 0...
  401056:       74 05                   je     40105d <phase_4+0x51> ; ...return
  401058:       e8 dd 03 00 00          callq  40143a <explode_bomb>
  40105d:       48 83 c4 18             add    $0x18,%rsp
  401061:       c3                      retq   
```

`func4`, as disassembled with `​objdump -d`` (``%edx`​ = 14, `​%esi`​ = 0, `​%rdi`​ = first int):

```asm
0000000000400fce <func4>:
  400fce:       48 83 ec 08             sub    $0x8,%rsp
  400fd2:       89 d0                   mov    %edx,%eax             ; 14 -> %eax
  400fd4:       29 f0                   sub    %esi,%eax             ; %eax -= 0 = 14
  400fd6:       89 c1                   mov    %eax,%ecx             ; 14 -> %ecx
  400fd8:       c1 e9 1f                shr    $0x1f,%ecx            ; %ecx >> 31
  400fdb:       01 c8                   add    %ecx,%eax             ; %eax += %ecx
  400fdd:       d1 f8                   sar    %eax                  ; %eax >> 1
  400fdf:       8d 0c 30                lea    (%rax,%rsi,1),%ecx    ; %ecx = %rax + (1 * %rsi)
  400fe2:       39 f9                   cmp    %edi,%ecx             ; 
  400fe4:       7e 0c                   jle    400ff2 <func4+0x24>   ; jump if %ecx <= %edi
  400fe6:       8d 51 ff                lea    -0x1(%rcx),%edx       ; %edx = -0x1(%rcx)
  400fe9:       e8 e0 ff ff ff          callq  400fce <func4>        ; recurse
  400fee:       01 c0                   add    %eax,%eax             
  400ff0:       eb 15                   jmp    401007 <func4+0x39>   
  400ff2:       b8 00 00 00 00          mov    $0x0,%eax             ; %eax = 0
  400ff7:       39 f9                   cmp    %edi,%ecx             ; 
  400ff9:       7d 0c                   jge    401007 <func4+0x39>   ; return if %ecx >= %edi
  400ffb:       8d 71 01                lea    0x1(%rcx),%esi
  400ffe:       e8 cb ff ff ff          callq  400fce <func4>
  401003:       8d 44 00 01             lea    0x1(%rax,%rax,1),%eax
  401007:       48 83 c4 08             add    $0x8,%rsp
  40100b:       c3                      retq   
```

1. `​phase_4` instructions 40101a–40102c: `​sscanf()`​ takes at least two integers separated by spaces; otherwise, the bomb explodes.
2. `phase_4` 40102e–401033: to make it to the call to `​func4`​ at instruction 401048, the first integer must be \<= 14 with the comparison treated as unsigned, so it must also be \>= 0.
3. `phase_4` 401048: `​func4`​ is called with the arguments: 14, 0, first integer (stored in in %edx, %esi, %edi respectively).
4. `phase_4` 40104d–40104f: the value returned by`​ func4`​ must be 0; otherwise the bomb explodes.
  1. `​func4`​ instructions 400fd2–400fd4: %eax (return value) is set to 14.
  2. `​func4`​ 400fd6–400fd8: %ecx is set to 14 \>\> 31 = 0.
  3. `func4`​ 400fdb–400fdd: %eax is set to (14 + 0) \>\> 1 = 7.
  4. `​func4`​ 400fdf: %ecx is set to %rax + (1 \* %rsi), which is equivalent to %eax + (1 \* %esi) = 7 + (1 \* 0) = 7 + 0 = 7.
  5. `​func4`​ 400fe2–400fe4: if 7 \<= the first integer (stored in %edi), we jump to 400ff2, where the return value is set to 0\. (Otherwise, at 400fe6–400fe9, we modify %edx and recurse.)
  6. `​func4` 400ff7–400ff9: if 7 \>= the first integer (stored in %edi), return 0.
  7. **Therefore, the first integer can be 7.**
  8. **However, there may be other possibilities. 7 is the only valid value that avoids the recursion at 400fe9.**
5. `​phase_4` 401051–401056: **the second integer must be 0**; otherwise, the bomb explodes.
6. **Therefore, one expected string is "7 0", though there may be other possibilities.**
7. Verify the above by stepping through the code after entering the test string **"7 0".**


Content of psol.txt (solution file)
-----------------------------------

```
Border relations with Canada have never been better.
1 2 4 8 16 32
0 207
7 0
```
