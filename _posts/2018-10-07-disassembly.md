---
title: "Disassembly"
date: 2018-10-07
tags: c
published: true
---

Source: [CS:APP3e, Bryant and O'Hallaron](http://csapp.cs.cmu.edu/), 
Binary Bomb Lab (from [Lab Assignments](http://csapp.cs.cmu.edu/3e/labs.html)).

The task is to disassemble and "reverse engineer" a binary file, using
a debugger to determine the content of the six strings the program
expects as console input.

The first six phases are defused here. Not included is the final "secret phase."

* TOC
{:toc}


Preliminary steps
-----------------

1. Disassemble `​bomb` with `objdump -d bomb > bomb.s`
2. Read through `​bomb.s`​. The section for the `main` function contains sequences of `call` instructions like this:
   * `read_line` (taking user input for each phase)
   * `phase_`*n* where *n* = 1..6 (testing user input against each bomb phase)
   * `phase_defused` (if successful)
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
* Useful to set `​layout asm`​ or `​display/i $pc`​ for viewing machine instructions while stepping through them
* To display data at an address as a sequence of characters, cast the data to a char pointer: `​print (char *) <address>`​
* Inspect register contents with `​print $<reg>`​ or `​x ​$<reg>`
* Look for arguments to `​sscanf()`​ on the stack (%rsp with varying offsets)
* To check on the results of comparisons before stepping to the jump instructions following them, inspect flags with `​print $eflags`​


**Function `phase_1`** (calling `strings_not_equal`)
----------------------------------------------------

As disassembled with ​`objdump -d`:

```asm
000000000040128f <phase_1>:
  40128f:       48 83 ec 08             sub    $0x8,%rsp
  401293:       be 48 24 40 00          mov    $0x402448,%esi
  401298:       e8 2f 00 00 00          callq  4012cc <strings_not_equal>
  40129d:       85 c0                   test   %eax,%eax
  40129f:       74 05                   je     4012a6 <phase_1+0x17>
  4012a1:       e8 e9 01 00 00          callq  40148f <explode_bomb>
  4012a6:       48 83 c4 08             add    $0x8,%rsp
  4012aa:       c3                      retq
```

### Discovery

1. We already have the stdin char input stored in register `​%rdi`.
2. The instruction `mov $0x402448,%esi` copies a constant value representing a memory address to register `​%esi`.
3. The next instruction is `callq 4012cc <strings_not_equal>`. Since it clearly returns a boolean value, we don't even need to read through `strings_not_equal`​ to assume (correctly) that when passed the stdin char data stored in `​%rdi`​ and the data stored in `​%esi`​ as arguments, `​strings_not_equal`​ sets `​%eax`​ to 1 if its arguments are not equal and 0 if its arguments are equal.
4. The next instruction is `test %eax,%eax` which sets `ZF` (zero flag) to 1 if `​%eax`​ contains 0.
5. The next instruction is `je 4012a6 <phase_1+0x17>` which jumps to 4012a6 if ZF is 1\. That is, if `​strings_not_equal`​ returned 0\. That is, if the strings are equal. So if the strings are equal, we are jumping to `4012a6​`, thereby jumping over the instruction that follows, which is `callq 40148f <explode_bomb>`.
6. To discover the string that is expected, start `​gdb` with `gdb bomb` and set a breakpoint with `​break phase_1`​.
7. We need to print the character representation of data at the memory address 0x402448 that was copied to %esi. We don't need to run anything to do this (I assume because `​gdb`​ looks ahead). Do it with the `print` command, casting the data to a char pointer: `print (char *) 0x402448`​. The result is `Why make trillions when we could make... billions?`
8. **Therefore, the expected string is "Why make trillions when we could make... billions?"**

### Verification

1. Confirm this by checking that this string appears in the output of the shell command `​strings bomb`​.
2. Now `​run`​. The breakpoint will stop execution after we enter the string "Why make trillions when we could make... billions?"
3. Check data at address 
0x402448​ with `print (char *) 0x402448`​. It should print `Why make trillions when we could make... billions?`
4. Check data in %rdi: `print (char *) $rdi` should display `Why make trillions when we could make... billions?`
5. Now set `layout asm`​ or `display/i $pc` to display each instruction and step in with `stepi`.
6. After two steps, use `print (char *) $esi` to check that `Why make trillions when we could make... billions?` was copied to `%esi`.
7. You should be at the instruction calling `​strings_not_equal`.
8. Step over `​strings_not_equal`​ with `​nexti`​.
9. Print the value of `​%eax`​ with `​print $eax`​. It should display 0, meaning that the strings were evaluated as equal.
10. Continue with `​c`. You should see the message `Phase 1 defused. How about the next one?`
11. You can't keep going at this point, since you're not ready for the next phase yet. Quit `​bomb`​ with Ctrl-c, then quit `gdb` with `q`.

### Storing the correct strings in a text file for convenience

1. Exit with `Ctrl-c`, then exit `gdb` with `q.` Add `Why make trillions when we could make... billions?` to a file named `psol.txt`, which can be passed as a command line argument to `bomb`.
2. You can run `./bomb psol.txt`​ whenever you like, though the defusing of phase 1 has already been recorded on the scoreboard.


**Function `phase_2`** (calling `read_six_numbers`)
---------------------------------------------------

`phase_2`, as disassembled with `​objdump -d`:

```asm
000000000040105a <phase_2>:
  40105a:       55                      push   %rbp
  40105b:       53                      push   %rbx
  40105c:       48 83 ec 28             sub    $0x28,%rsp
  401060:       48 89 e6                mov    %rsp,%rsi
  
  401063:       e8 5d 04 00 00          callq  4014c5 <read_six_numbers> ; get six ints from stdin
  401068:       83 3c 24 00             cmpl   $0x0,(%rsp)               ; compare first int to 0
  40106c:       75 07                   jne    401075 <phase_2+0x1b>     ; if not 0, explode
  40106e:       83 7c 24 04 01          cmpl   $0x1,0x4(%rsp)            ; compare second int to 1
  401073:       74 05                   je     40107a <phase_2+0x20>     ; if 1, jump
  401075:       e8 15 04 00 00          callq  40148f <explode_bomb>     ; else explode
  40107a:       48 89 e5                mov    %rsp,%rbp                 ; %rbp = addr of first int
  40107d:       48 8d 5c 24 08          lea    0x8(%rsp),%rbx            ; %rbx = addr of third int
  401082:       48 83 c5 18             add    $0x18,%rbp                ; %rbp = addr AFTER sixth int
  
                                                                         ; LOOP:
                                                                         ; at start, %rbx = addr of 
                                                                         ;     third int.
  401086:       8b 43 fc                mov    -0x4(%rbx),%eax           ; %eax = curr int
  401089:       03 43 f8                add    -0x8(%rbx),%eax           ; %eax += prev int
  40108c:       39 03                   cmp    %eax,(%rbx)               ; compare curr to next int
  40108e:       74 05                   je     401095 <phase_2+0x3b>     ; if equal, jump
  401090:       e8 fa 03 00 00          callq  40148f <explode_bomb>     ; else explode
  401095:       48 83 c3 04             add    $0x4,%rbx                 ; %rbx += 4: take next int
  401099:       48 39 eb                cmp    %rbp,%rbx                 ; are we past the 6th int?
  40109c:       75 e8                   jne    401086 <phase_2+0x2c>     ; continue
  
  40109e:       48 83 c4 28             add    $0x28,%rsp                ; return
  4010a2:       5b                      pop    %rbx
  4010a3:       5d                      pop    %rbp
  4010a4:       c3                      retq
```

`read_six_numbers`, as disassembled with `​objdump -d`:

```asm
00000000004014c5 <read_six_numbers>:
  4014c5:       48 83 ec 18             sub    $0x18,%rsp
  4014c9:       48 89 f2                mov    %rsi,%rdx
  4014cc:       48 8d 4e 04             lea    0x4(%rsi),%rcx
  4014d0:       48 8d 46 14             lea    0x14(%rsi),%rax
  4014d4:       48 89 44 24 08          mov    %rax,0x8(%rsp)
  4014d9:       48 8d 46 10             lea    0x10(%rsi),%rax
  4014dd:       48 89 04 24             mov    %rax,(%rsp)
  4014e1:       4c 8d 4e 0c             lea    0xc(%rsi),%r9
  4014e5:       4c 8d 46 08             lea    0x8(%rsi),%r8
  4014e9:       be 7e 25 40 00          mov    $0x40257e,%esi ; copy "%d %d %d %d %d %d" to %esi
  4014ee:       b8 00 00 00 00          mov    $0x0,%eax      ; start counter at 0
  4014f3:       e8 d0 f5 ff ff          callq  400ac8 <__isoc99_sscanf@plt> ; sscanf() passing %esi
  4014f8:       83 f8 05                cmp    $0x5,%eax      ; compare 5 to counter
  4014fb:       7f 05                   jg     401502 <read_six_numbers+0x3d> ; if 5 > counter, recurse
  4014fd:       e8 8d ff ff ff          callq  40148f <explode_bomb> ; explode upon user mistake
  401502:       48 83 c4 18             add    $0x18,%rsp
  401506:       c3                      retq
```

### Discovery

1. Start by looking at `​read_six_numbers. At 4014e9`, `mov $0x40257e,%esi `copies data at the constant memory address 0x40257e to `​%esi,` presumably to be used as an argument to `​sscanf()`​.
2. We want to see what that argument looks like. Print its character representation: `print (char *) `0x40257e​. The result is `​"%d %d %d %d %d %d"`, which tells us that `​sscanf()`​ expects six integers separated by spaces.
3. Now look at `​phase_2`​ again. If six integers were entered and the bomb did not explode, `read_six_numbers` has placed data representing those six numbers on the stack.
4. At 401068, the first integer is compared to 0\. If it's not 0, we jump to 401075, where the bomb explodes.
5. **Therefore, the first integer must be 0.**
6. At 40106e, the second integer is compared to 1\. If it's 1, jump; otherwise, the bomb explodes.
7. **Therefore, the second integer must be 1.**
8. At 40107a, %rbp is set to the address of the first integer; at 40107d, %rbx is set to the address of the *third* integer, by using an 8-byte offset.
9. At 401082, %rbp is incremented by an 18-byte offset, which gives it the value of the address after the sixth integer. If we end up here, we know we've finished processing valid input data.
10. At 400f30, the address of the next integer is copied to `​%rbx`​ by loading with a 4-byte offset.
11. At 400f35, the 18-byte offset means that the address *after* the last of the six integers is copied to `​%rbp`​.
12. From 401086 to 40109c, we're in a loop. In each iteration, the *current* integer is incremented by the value of the *previous* integer. The result is compared to the *next* integer. If the result is not equal to the next integer, the bomb explodes. Otherwise, we increment %rbx by an offset of 4 bytes, to get the next integer from input, and if we haven't reached the sixth integer yet, we continue.
13. **Therefore, the expected string is "0 1 1 2 3 5"**, the beginning of (one version of) the Fibonacci sequence.

### Verification

1. Start `​gdb` with `gdb bomb,` set a breakpoint with `​break phase_2`, and `run.` The breakpoint will stop execution after we enter the string "0 1 1 2 3 5".
2. Set `layout asm`​ or `display/i $pc` and skip ahead to 401068 with `advance *0x401068`.
3. Inspect `%rsp` with `​x $rsp`. Its contents should be 0, the first integer entered by the user.
4. Inspect the value at `$rsp+0x4`. It should be 1, the second integer entered by the user.
5. Step to 401086 manually using `​stepi`, quitting `​gdb `if you land on a call to `​explode_bomb`.
6. Check that the value in %rbx (via `x $rbx`) is 1\.
7. Check that the value in %rbp (via `x $rbp`) is equal to %rsp+0x18.
8. Step through the loop, checking %eax (with `​print $eax`) and %rbx (with `​x $rbx`) for the appropriate values, quitting gdb if you land on a call to `​explode_bomb`​.
9. When you're safe, continue with `​c`​. You should see the message `​That's number 2.  Keep going!`
10. You can't keep going at this point, since you're not ready for the next phase yet. Quit `​bomb`​ with Ctrl-c, then quit `gdb` with `q`.


**Function `​phase_3`​**
----------------------

`​phase_3`​, as disassembled with `​objdump -d`​:

```asm
0000000000401142 <phase_3>:
  401142:       48 83 ec 18             sub    $0x18,%rsp
  401146:       48 8d 4c 24 07          lea    0x7(%rsp),%rcx
  40114b:       48 8d 54 24 0c          lea    0xc(%rsp),%rdx
  401150:       4c 8d 44 24 08          lea    0x8(%rsp),%r8
  401155:       be 7b 24 40 00          mov    $0x40247b,%esi          ; "%d %c %d"
  40115a:       b8 00 00 00 00          mov    $0x0,%eax
  40115f:       e8 64 f9 ff ff          callq  400ac8 <__isoc99_sscanf@plt>
  401164:       83 f8 02                cmp    $0x2,%eax               ; if at least 3 arguments
  401167:       7f 05                   jg     40116e <phase_3+0x2c>   ;   entered, continue
  401169:       e8 21 03 00 00          callq  40148f <explode_bomb>
  
  40116e:       83 7c 24 0c 07          cmpl   $0x7,0xc(%rsp)          ; 0 <= first int <= 7
  401173:       0f 87 fc 00 00 00       ja     401275 <phase_3+0x133>
  401179:       8b 44 24 0c             mov    0xc(%rsp),%eax          ; copy first int to %eax
  40117d:       ff 24 c5 a0 24 40 00    jmpq   *0x4024a0(,%rax,8)      ; jump to computed address
  401184:       b8 69 00 00 00          mov    $0x69,%eax
  401189:       81 7c 24 08 f3 02 00    cmpl   $0x2f3,0x8(%rsp)        ; if 755 == second int...
  401190:       00 
  401191:       0f 84 e8 00 00 00       je     40127f <phase_3+0x13d>  ; ...jump
  401197:       e8 f3 02 00 00          callq  40148f <explode_bomb>   ; else explode
  40119c:       b8 69 00 00 00          mov    $0x69,%eax
  4011a1:       e9 d9 00 00 00          jmpq   40127f <phase_3+0x13d>
  4011a6:       b8 6e 00 00 00          mov    $0x6e,%eax
  4011ab:       81 7c 24 08 ba 01 00    cmpl   $0x1ba,0x8(%rsp)        ; if 442 == second int...
  4011b2:       00 
  4011b3:       0f 84 c6 00 00 00       je     40127f <phase_3+0x13d>  ; ...jump
  4011b9:       e8 d1 02 00 00          callq  40148f <explode_bomb>   ; else explode
  4011be:       b8 6e 00 00 00          mov    $0x6e,%eax
  4011c3:       e9 b7 00 00 00          jmpq   40127f <phase_3+0x13d>
  4011c8:       b8 62 00 00 00          mov    $0x62,%eax
  4011cd:       81 7c 24 08 11 02 00    cmpl   $0x211,0x8(%rsp)        ; if 529 == second int...
  4011d4:       00 
  4011d5:       0f 84 a4 00 00 00       je     40127f <phase_3+0x13d>  ; ...jump
  4011db:       e8 af 02 00 00          callq  40148f <explode_bomb>   ; else explode
  4011e0:       b8 62 00 00 00          mov    $0x62,%eax
  4011e5:       e9 95 00 00 00          jmpq   40127f <phase_3+0x13d>
  4011ea:       b8 6b 00 00 00          mov    $0x6b,%eax
  4011ef:       81 7c 24 08 34 03 00    cmpl   $0x334,0x8(%rsp)        ; if 820 == second int...
  4011f6:       00 
  4011f7:       0f 84 82 00 00 00       je     40127f <phase_3+0x13d>  ; ...jump
  4011fd:       e8 8d 02 00 00          callq  40148f <explode_bomb>   ; else explode
  401202:       b8 6b 00 00 00          mov    $0x6b,%eax
  401207:       eb 76                   jmp    40127f <phase_3+0x13d>
  401209:       b8 78 00 00 00          mov    $0x78,%eax
  40120e:       81 7c 24 08 e9 00 00    cmpl   $0xe9,0x8(%rsp)         ; if 233 == second int...
  401215:       00 
  401216:       74 67                   je     40127f <phase_3+0x13d>  ; ...jump
  401218:       e8 72 02 00 00          callq  40148f <explode_bomb>   ; else explode
  40121d:       b8 78 00 00 00          mov    $0x78,%eax
  401222:       eb 5b                   jmp    40127f <phase_3+0x13d>
  401224:       b8 63 00 00 00          mov    $0x63,%eax
  401229:       81 7c 24 08 d5 02 00    cmpl   $0x2d5,0x8(%rsp)        ; if 725 == second int...
  401230:       00 
  401231:       74 4c                   je     40127f <phase_3+0x13d>  ; ...jump
  401233:       e8 57 02 00 00          callq  40148f <explode_bomb>   ; else explode
  401238:       b8 63 00 00 00          mov    $0x63,%eax
  40123d:       eb 40                   jmp    40127f <phase_3+0x13d>
  40123f:       b8 75 00 00 00          mov    $0x75,%eax
  401244:       81 7c 24 08 18 03 00    cmpl   $0x318,0x8(%rsp)        ; if 792 == second int...
  40124b:       00 
  40124c:       74 31                   je     40127f <phase_3+0x13d>  ; jump
  40124e:       e8 3c 02 00 00          callq  40148f <explode_bomb>   ; else explode
  401253:       b8 75 00 00 00          mov    $0x75,%eax
  401258:       eb 25                   jmp    40127f <phase_3+0x13d>
  40125a:       b8 79 00 00 00          mov    $0x79,%eax
  40125f:       81 7c 24 08 4e 02 00    cmpl   $0x24e,0x8(%rsp)        ; if 590 == second int...
  401266:       00 
  401267:       74 16                   je     40127f <phase_3+0x13d>  ; ...jump
  401269:       e8 21 02 00 00          callq  40148f <explode_bomb>   ; else explode
  40126e:       b8 79 00 00 00          mov    $0x79,%eax
  401273:       eb 0a                   jmp    40127f <phase_3+0x13d>
  401275:       e8 15 02 00 00          callq  40148f <explode_bomb>
  40127a:       b8 74 00 00 00          mov    $0x74,%eax
  40127f:       3a 44 24 07             cmp    0x7(%rsp),%al           ; if char == 'i'
  401283:       74 05                   je     40128a <phase_3+0x148>  ; ...return
  401285:       e8 05 02 00 00          callq  40148f <explode_bomb>   ; else explode
  40128a:       48 83 c4 18             add    $0x18,%rsp
  40128e:       c3                      retq   
```

1. The instruction at 401155 copies data to %esi which will be passed to C `​sscanf()`​ at 40115f. That will be the format string that `​sscanf()`​ expects, which will tell us about the form of input it expects. In gdb, `print (char *) 0x40247b` yields `"%d %c %d"`​, so we know that `phase_3` expects an integer, a character, and another integer, separated by spaces.
2. ​The return value of `sscanf()`​ is the number of variables filled. Therefore, at 401164, after the call to `​sscanf()`, %eax will hold a value representing the number of integers we entered (up to 3, as specified by the format string "%d %c %d"). Check this by setting a breakpoint at `​phase_3`​, then typing `​run`​, and when we get to `phase_3` entering "0 a 1" as a test string. Set `layout asm`​ or `display/i $pc`, advance to 401164, and inspect %eax with `​p $eax`​. The result is 3\. Verify this by running again and testing with fewer input items.
3. Instructions 401164–4011649 tell us that if the user does not enter 3 input items, the bomb explodes.
4. Otherwise, we jump to 40116e, where we find the first of many `​cmp`​ instructions in the rest of the function. Inspecting 0xc(%rsp) with `​x $rsp+0xc`​, we get 0x00000000, which is presumably the first integer we entered, pushed onto the stack.
5. Instructions 40116e–401173 tell us that that if our first integer is greater than 7, the bomb explodes.
6. **Therefore, our first integer must be less than or equal to 7\. Since `​ja`​ interprets CF and ZF as if the compared values are unsigned, this means that our first integer must also be equal to or greater than zero.**
7. We arrive at 401179–40117d, where the first integer is copied to %eax and a jump address is computed: `​jmpq   *0x4024a0(,%rax,8)`​ means 8 \* %rax + 0x4024a0\. Get the value in %rax with `​p $rax`​ (it's 0, the first integer we entered, which was copied to %eax at 401179) and plug it in: 8 \* 0 + 0x4024a0 = 0x4024a0\. *This is an address at which another address is stored, as a value.* Get the stored value representing an address with `x 0x4024a0`​; the result is 0x00401184. This is the address inside `​phase_3` to which execution will jump.
8. If our first integer was 0, the computed address is the very next instruction.
9. At 0x401189 we find another `​cmp`​ instruction. Inspecting 0x8(%rsp) with `​x $rsp+0x8`​, we get 0x00000001, which is presumably the second integer we entered, pushed onto the stack.
10. Instructions 0x401191–0x401197 tell us that if our second integer is not equal to 0x2f3 (755), the bomb will explode.
11. **Therefore, if our first integer is 0, then our second integer must be 755.**
12. So, we need to start over with a better test string. Run `bomb`​ inside gdb again with a breakpoint at `phase_3`, enter the test string "0 a 755", advance to 0x401189, and inspect 0x8(%rsp) to ensure it is 0x2f3 (755). Step forward one instruction and check the flags with `​p $eflags`​. ZF should be set to 1.
13. We arrive at 0x40127f–0x401283, where 0x7(%rsp) is compared to %al.
  1. Inspecting 0x7(%rsp) with `​x $rsp+0x7`​, we get 0x0002f361 (193377). Since we've already accounted for the two integers, we might guess that this somehow represents the character 'a' in our input string. How? In C, `​char`​ is a one-byte data type. The least significant byte of 0x0002f361 is 0x61, which is 97 in decimal, which is the ASCII code for lowercase 'a'.
  2. What about %al? This is the data our input character is being compared to, so it's presumably the value that `​phase_3`​ expects for the character. Inspect %al with `​p $al`​ and we get 105, which is the ASCII code for lowercase 'i'.
14. Instructions 0x401283–0x401285 tell us that if our input character is not 'i'. the bomb will explode.
15. **Therefore, our character must be 'i'.**
16. So, we need to start over yet again. Run `bomb`​ inside gdb again with a breakpoint at `phase_3`, enter the test string "0 i 755", advance to 0x40127f, and inspect 0x7(%rsp) and %al. 0x7(%rsp) should be 0x0002f369, whose least significant byte is 0x69, which is 105 in decimal. %al should also be 105\. Step forward one instruction and inspect the flags with `​p $eflags`​. ZF should be set to 1.
17. *However: the jump address computed at 40117d will be different depending on whether the first integer entered is 0, 1, 2, 3, 4, 5, 6, or 7, and the computations from 401184–401273 will produce a different value for the second integer depending on what the first is. *Our second integer must be 755 if the first is 0; 442 if the first is 1; 529 if the first is 2; and so on.
18. **Therefore, the expected string is "*a* i *b*", where ****0 \<= *a* \<= 7 and *b* is one of the corresponding values [755, 442, 529...]. For example, "0 i 755"** will defuse the bomb.


**Function `phase_4`** (calling `func4`)
----------------------------------------

`phase_4`, as disassembled with `​objdump -d`:

```asm
00000000004010e5 <phase_4>:
  4010e5:       48 83 ec 18             sub    $0x18,%rsp
  4010e9:       48 8d 4c 24 08          lea    0x8(%rsp),%rcx
  4010ee:       48 8d 54 24 0c          lea    0xc(%rsp),%rdx
  4010f3:       be 8a 25 40 00          mov    $0x40258a,%esi        ; "%d %d"
  4010f8:       b8 00 00 00 00          mov    $0x0,%eax
  4010fd:       e8 c6 f9 ff ff          callq  400ac8 <__isoc99_sscanf@plt>
  401102:       83 f8 02                cmp    $0x2,%eax             ; 2 arguments required
  401105:       75 0d                   jne    401114 <phase_4+0x2f>
  
  401107:       8b 44 24 0c             mov    0xc(%rsp),%eax        ; first int -> %eax
  40110b:       85 c0                   test   %eax,%eax             ; %eax & %eax
  40110d:       78 05                   js     401114 <phase_4+0x2f> ; explode if SF == 1 in result
  40110f:       83 f8 0e                cmp    $0xe,%eax
  401112:       7e 05                   jle    401119 <phase_4+0x34> ; jump if first int <= 14
  401114:       e8 76 03 00 00          callq  40148f <explode_bomb>
  401119:       ba 0e 00 00 00          mov    $0xe,%edx
  40111e:       be 00 00 00 00          mov    $0x0,%esi
  401123:       8b 7c 24 0c             mov    0xc(%rsp),%edi        ; first int -> %edi
  401127:       e8 44 fd ff ff          callq  400e70 <func4>        ; call with args: 14, 0, first int
  40112c:       83 f8 02                cmp    $0x2,%eax
  40112f:       75 07                   jne    401138 <phase_4+0x53> ; explode if func4 doesn't return 2
  401131:       83 7c 24 08 02          cmpl   $0x2,0x8(%rsp)
  401136:       74 05                   je     40113d <phase_4+0x58> ; return if second int == 2
  401138:       e8 52 03 00 00          callq  40148f <explode_bomb> ; else explode
  40113d:       48 83 c4 18             add    $0x18,%rsp
  401141:       c3                      retq
```

`func4`, as disassembled with `​objdump -d`:

`%edx`​ = 14, `​%esi`​ = 0, `​%edi`​ = first int

```asm
0000000000400e70 <func4>:
  400e70:       48 83 ec 08             sub    $0x8,%rsp
  400e74:       89 d0                   mov    %edx,%eax
  400e76:       29 f0                   sub    %esi,%eax
  400e78:       89 c1                   mov    %eax,%ecx
  400e7a:       c1 e9 1f                shr    $0x1f,%ecx
  400e7d:       8d 04 01                lea    (%rcx,%rax,1),%eax
  400e80:       d1 f8                   sar    %eax
  400e82:       8d 0c 30                lea    (%rax,%rsi,1),%ecx
  400e85:       39 f9                   cmp    %edi,%ecx
  400e87:       7e 0c                   jle    400e95 <func4+0x25>
  400e89:       8d 51 ff                lea    -0x1(%rcx),%edx
  400e8c:       e8 df ff ff ff          callq  400e70 <func4>
  400e91:       01 c0                   add    %eax,%eax
  400e93:       eb 15                   jmp    400eaa <func4+0x3a>
  400e95:       b8 00 00 00 00          mov    $0x0,%eax
  400e9a:       39 f9                   cmp    %edi,%ecx
  400e9c:       7d 0c                   jge    400eaa <func4+0x3a>
  400e9e:       8d 71 01                lea    0x1(%rcx),%esi
  400ea1:       e8 ca ff ff ff          callq  400e70 <func4>
  400ea6:       8d 44 00 01             lea    0x1(%rax,%rax,1),%eax
  400eaa:       48 83 c4 08             add    $0x8,%rsp
  400eae:       c3                      retq
```

(At this point, one should feel more comfortable both reading the assembly code and moving around in gdb, so I'm documenting the remaining defusals in less detail.)

1. `​phase_4` instructions 4010f3–401105: `​sscanf()`​ takes at least two integers separated by spaces; otherwise, the bomb explodes.
2. `​phase_4`​ 401107–40110d: the bomb explodes if the sign flag (SF) is set to 1 in the result of `test %eax,%eax`​​ (that is, the result of `%eax & %eax`​). This means that **the first integer must be zero or greater** (the sign bit will be set to 1 in a two's complement bit pattern representing a negative integer).
3. `phase_4` 40110f–401112: to make it to the call to `​func4`​ at instruction 401127, the first integer must be \<= 14\. **Therefore, the first integer must be equal to or greater than zero, and equal to or less than 14.**
4. `phase_4` 401127: `​func4`​ is called with the arguments: 14, 0, first integer (stored in in %edx, %esi, %edi respectively).
5. `phase_4` 40112c–40112f: **the value returned by`​ func4`​ must be 2; otherwise the bomb explodes.**
6. `​phase_4`​ instruction 401136: if the second integer is 2, return; otherwise, the bomb explodes. **Therefore the second integer must be 2.**
7. We are looking for a `​func4` return value of 2\. Enter gdb, set a breakpoint at `​phase_4`, and experiment with the test strings **"0 2", "1 2", "2 2", "3 2", "4 2", **etc. ​Set `​display $eax`​ and use until \*0x40112c​ to skip to the point where `​func4`​ returns its return value.
8. **The string "4 2" will defuse the bomb.**


**Function `phase_5`** (calling `string_length`)
------------------------------------------------

`phase_5`, as disassembled with `​objdump -d`:

```asm
00000000004010a5 <phase_5>:
  4010a5:       53                      push   %rbx
  4010a6:       48 89 fb                mov    %rdi,%rbx               ; input string from %rdi -> %rbx
  4010a9:       e8 02 02 00 00          callq  4012b0 <string_length>
  4010ae:       83 f8 06                cmp    $0x6,%eax
  4010b1:       74 05                   je     4010b8 <phase_5+0x13>   ; jump if %eax == 6
  4010b3:       e8 d7 03 00 00          callq  40148f <explode_bomb>   ; else explode
  
  4010b8:       48 8d 73 06             lea    0x6(%rbx),%rsi          ; guard: %rsi = %rbx + 6
  4010bc:       b8 00 00 00 00          mov    $0x0,%eax               ; 0 -> %eax
  4010c1:       ba e0 24 40 00          mov    $0x4024e0,%edx          ; $0x4024e0 -> %edx
  
                                                                       ; Loop over 1-byte chars in string
  4010c6:       48 0f be 0b             movsbq (%rbx),%rcx             ;   *%rbx -> %rcx, sign-extending
  4010ca:       83 e1 0f                and    $0xf,%ecx               ;   %ecx &= 0xf
  4010cd:       03 04 8a                add    (%rdx,%rcx,4),%eax      ;   %eax += %rdx + (4 * %rcx)
  4010d0:       48 83 c3 01             add    $0x1,%rbx               ;   advance one character
  4010d4:       48 39 f3                cmp    %rsi,%rbx               ;   
  4010d7:       75 ed                   jne    4010c6 <phase_5+0x21>   ;   jump if we're at end of string
  
  4010d9:       83 f8 3b                cmp    $0x3b,%eax
  4010dc:       74 05                   je     4010e3 <phase_5+0x3e>   ; return if %eax == 0x3b (59)
  4010de:       e8 ac 03 00 00          callq  40148f <explode_bomb>   ; else explode
  4010e3:       5b                      pop    %rbx
  4010e4:       c3                      retq
```

`string_length`, as disassembled with `​objdump -d`:

```asm
00000000004012b0 <string_length>:
  4012b0:       48 89 fa                mov    %rdi,%rdx
  4012b3:       b8 00 00 00 00          mov    $0x0,%eax
  4012b8:       80 3f 00                cmpb   $0x0,(%rdi)
  4012bb:       74 0d                   je     4012ca <string_length+0x1a>
  4012bd:       48 83 c2 01             add    $0x1,%rdx
  4012c1:       89 d0                   mov    %edx,%eax
  4012c3:       29 f8                   sub    %edi,%eax
  4012c5:       80 3a 00                cmpb   $0x0,(%rdx)
  4012c8:       75 f3                   jne    4012bd <string_length+0xd>
  4012ca:       f3 c3                   repz retq
```

1. `​phase_5​` instruction 4010a6: copy input string from %rdi to %rbx.
2. `phase_5`​ 4010b1: the input string must be 6 characters long, otherwise the bomb explodes.
3. `phase_5`​ 4010b8: set %rsi to a guard value: an address 6 bytes past the beginning of %rbx (that is, the input string).
4. `​phase_5`​ 4010c1: set %edx to the constant 0x4024e0.
5. `​phase_5`​ 4010c6–4010d7: loop processing each one-byte character in the string.
    1. 4010c6: dereference %rbx and copy the value to %rcs, sign-extending. The value stored in %rcs will be the ASCII code for the current character.
    2. `​4010ca`: set %ecx to the result of %ecx & 0xf.
    3. `​4010cd`: %eax was set to 0 on 4010bc. Here, add %rdx + (4 \* %rcx) to 0 and store the result in %eax.
    4. `​4010d0`: add 1 to the value stored in %rbx and store the result in %rbx.
    5. `​4010d7`: if we are not at the end of the string, jump back to 4010c6.
6. `​phase_5`​ 4010dc: if %eax == 59, the bomb is defused. Otherwise, it explodes.
7. So we know that %eax must accumulate to 0x3b (59) while processing all six characters. At 4010cd, `​add(%rdx,%rcx,4),%eax` tells us that %eax is incremented by a value representing data at the address 0x4024e0 (stored in %edx) combined with some portion of the input string (stored in %rcx).
8. Run `​bomb`​ in gdb and enter the test string **"abcdef"**. Step through instruction 4010b1 to check that the length-checking portion doesn't explode the bomb. (Subsequently you can `​advance *0x4010ca`​ to move to the point where register values are first meaningful.)
9. We want to watch %eax: `display $eax`.
10. We also want to watch %rcx. Use `​display/c $rcx`​ to print its integer and char representation. The result is 97 'a'. So, %rcx contains the ASCII code of the character currently being processed. However, at 4010ca this value is transformed by the instruction `​and $0xf,%ecx`​. So let's watch its integer representation as well: `​display $rcx`​.
11. Display data at the address 0x4024e0 using `​x 0x4024e0`​. The result is 2\.
12. If on the other hand we display data at the address 0x4024e0 using `​x/i 0x4024e0`​, the result is `​0x4024e0 <array.3332>:	add (%rax),%al`​: an instruction to dereference %rax and increment the low byte of %ax by that value. If we go look at instruction ​0x4024e0, it's part of a sequence of instructions creating an array.
13. And if we display data at the address 0x4024e0 using `x/16w 0x4024e0`​, we get an array of integers. Presumably this table was formed using those instructions.
14. Presumably the instruction at 4010cd is selecting numbers from this table: `(%rdx,%rcx,4)`​ selects an integer from the table using the value of %rcx as an index.
15. So, to defuse the bomb, we need to first choose a sequence of six integers from this table at 0x4024e0 that add up to 59\. For example, 7 + 5 + 11 + 8 + 15 + 13 = 59.
16. Somehow we need to retrieve these integers using the character data in our input string. The array *indexes* of the array *elements* 7, 5, 11, 8, 15, 13 are 9, 11, 12, 13, 14, 15, so we need character data corresponding to the indexes.
17. The instruction `add (%rax),%al`​`​ at `0x4024e0 seems to be storing the low byte of its input. So let's look for ASCII characters the low bytes of the hex representations of which represent the array *indexes* we need.
18. ASCII 'I' is 0x49, the low byte of which is 9, which is the *index* value of the array *element* 7\. Presumably, then, we can retrieve the array element 7 using the character 'I'.
19. ASCII 'K' is 0x4B, the low byte of which is B (11), which is the *index* value of the array *element* 5\. And so on:
    1. Array *elements* that sum to 59: 7, 5, 11, 8, 15, 13​
    2. Array *indexes *of the above *elements*: 9, 11, 12, 13, 14, 15
    3. ASCII characters whose low bytes are 9, 11, 12, 13, 14, 15: I, K, L, M, N, O
20. **The string "IKLMNO" will defuse the bomb.**


**Function `phase_6`** (calling `read_six_numbers`)
---------------------------------------------------

`phase_6`, as disassembled with `​objdump -d`:

```asm

0000000000400f3b <phase_6>:
  400f3b:       41 55                   push   %r13
  400f3d:       41 54                   push   %r12
  400f3f:       55                      push   %rbp
  400f40:       53                      push   %rbx
  400f41:       48 83 ec 58             sub    $0x58,%rsp
  400f45:       4c 8d 64 24 30          lea    0x30(%rsp),%r12
  400f4a:       4c 89 e6                mov    %r12,%rsi
  
  400f4d:       e8 73 05 00 00          callq  4014c5 <read_six_numbers> ; stores int array at %r12

  400f52:       4c 89 e5                mov    %r12,%rbp      ; address of int array -> %rbp
  400f55:       41 bd 00 00 00 00       mov    $0x0,%r13d     ;   32-bit counter: const 0 -> %r13d
  
                                                              ; BEGIN LOOP
  400f5b:       8b 45 00                mov    0x0(%rbp),%eax ;   curr int from array -> %eax
  400f5e:       83 e8 01                sub    $0x1,%eax      ;   decrement curr int
  400f61:       83 f8 05                cmp    $0x5,%eax      ;   compare decremented curr int to 5
  400f64:       76 05                   jbe    400f6b <phase_6+0x30> ; jump if 1 <= curr int <= 6
  400f66:       e8 24 05 00 00          callq  40148f <explode_bomb> ; else explode
  
  400f6b:       41 83 c5 01             add    $0x1,%r13d     ; increment counter
  400f6f:       41 83 fd 06             cmp    $0x6,%r13d
  400f73:       74 22                   je     400f97 <phase_6+0x5c> ; break if counter == 6
  
  400f75:       44 89 eb                mov    %r13d,%ebx     ; counter -> %ebx
  
  400f78:       48 63 c3                movslq %ebx,%rax      ; sign-extend counter, 32-bit to 64-bit
  400f7b:       8b 55 00                mov    0x0(%rbp),%edx ; first int from array -> %edx
  400f7e:       3b 54 84 30             cmp    0x30(%rsp,%rax,4),%edx ; compare next int to curr int
  400f82:       75 05                   jne    400f89 <phase_6+0x4e>  ; continue if next int != curr int
  400f84:       e8 06 05 00 00          callq  40148f <explode_bomb>  ; else explode
  
  400f89:       83 c3 01                add    $0x1,%ebx ; increment counter
  400f8c:       83 fb 05                cmp    $0x5,%ebx
  400f8f:       7e e7                   jle    400f78 <phase_6+0x3d> ; continue if counter <= 5
  
  400f91:       48 83 c5 04             add    $0x4,%rbp ; advance to position of next int
  400f95:       eb c4                   jmp    400f5b <phase_6+0x20> ; continue
                                                              ; END LOOP
  
  400f97:       48 8d 4c 24 48          lea    0x48(%rsp),%rcx ; address after array -> %rcx
  400f9c:       ba 07 00 00 00          mov    $0x7,%edx       ; const 7 -> %edx
  
                                                               ; BEGIN LOOP
  400fa1:       89 d0                   mov    %edx,%eax       ; const 7 -> %eax
  400fa3:       41 2b 04 24             sub    (%r12),%eax     ; %eax -= curr int from array
  400fa7:       41 89 04 24             mov    %eax,(%r12)     ; replace curr int with %eax
  400fab:       49 83 c4 04             add    $0x4,%r12       ; advance to next int
  400faf:       49 39 cc                cmp    %rcx,%r12       ; check if we're past the 6th int...
  400fb2:       75 ed                   jne    400fa1 <phase_6+0x66> ; if not, continue
                                                               ; END LOOP
  
  400fb4:       bb 00 00 00 00          mov    $0x0,%ebx       ; set index variable to 0
  400fb9:       4c 8d 44 24 30          lea    0x30(%rsp),%r8  ; address of array -> %r8
  400fbe:       bd 01 00 00 00          mov    $0x1,%ebp       ; increment address of 6th int
  400fc3:       be 70 37 60 00          mov    $0x603770,%esi  ; copy address of node1 -> %esi
  400fc8:       48 89 e7                mov    %rsp,%rdi
  400fcb:       eb 19                   jmp    400fe6 <phase_6+0xab>
  
  400fcd:       48 8b 52 08             mov    0x8(%rdx),%rdx
  400fd1:       83 c0 01                add    $0x1,%eax
  400fd4:       39 c8                   cmp    %ecx,%eax
  400fd6:       75 f5                   jne    400fcd <phase_6+0x92>
  
  400fd8:       48 89 14 5f             mov    %rdx,(%rdi,%rbx,2)
  400fdc:       48 83 c3 04             add    $0x4,%rbx
  400fe0:       48 83 fb 18             cmp    $0x18,%rbx
  400fe4:       74 10                   je     400ff6 <phase_6+0xbb>
                                                                 
  400fe6:       41 8b 0c 18             mov    (%r8,%rbx,1),%ecx
  400fea:       89 e8                   mov    %ebp,%eax
  400fec:       48 89 f2                mov    %rsi,%rdx
  400fef:       83 f9 01                cmp    $0x1,%ecx
  400ff2:       7f d9                   jg     400fcd <phase_6+0x92>
  
  400ff4:       eb e2                   jmp    400ff2 <phase_6+0x9d>

  400ff6:       48 8b 1c 24             mov    (%rsp),%rbx
  400ffa:       48 8b 44 24 08          mov    0x8(%rsp),%rax
  400fff:       48 89 43 08             mov    %rax,0x8(%rbx)
  401003:       48 8b 54 24 10          mov    0x10(%rsp),%rdx
  401008:       48 89 50 08             mov    %rdx,0x8(%rax)
  40100c:       48 8b 44 24 18          mov    0x18(%rsp),%rax
  401011:       48 89 42 08             mov    %rax,0x8(%rdx)
  401015:       48 8b 54 24 20          mov    0x20(%rsp),%rdx
  40101a:       48 89 50 08             mov    %rdx,0x8(%rax)
  40101e:       48 8b 44 24 28          mov    0x28(%rsp),%rax
  401023:       48 89 42 08             mov    %rax,0x8(%rdx)
  401027:       48 c7 40 08 00 00 00    movq   $0x0,0x8(%rax)
  40102e:       00
  40102f:       bd 00 00 00 00          mov    $0x0,%ebp
  401034:       48 8b 43 08             mov    0x8(%rbx),%rax
  401038:       8b 13                   mov    (%rbx),%edx
  40103a:       3b 10                   cmp    (%rax),%edx
  40103c:       7d 05                   jge    401043 <phase_6+0x108>
  40103e:       e8 4c 04 00 00          callq  40148f <explode_bomb>
  401043:       48 8b 5b 08             mov    0x8(%rbx),%rbx
  401047:       83 c5 01                add    $0x1,%ebp
  40104a:       83 fd 05                cmp    $0x5,%ebp
  40104d:       75 e5                   jne    401034 <phase_6+0xf9>
  40104f:       48 83 c4 58             add    $0x58,%rsp
  401053:       5b                      pop    %rbx
  401054:       5d                      pop    %rbp
  401055:       41 5c                   pop    %r12
  401057:       41 5d                   pop    %r13
  401059:       c3                      retq
```

1. Enter the test string **"0 1 2 3 4 5"**. (We know from `​phase_2`​ that `​read_six_numbers`​ expects six integers separated by spaces.)
2. Inspect %r12 after the call to `​read_six_numbers`​. When we inspect it with `​x $r12`​, we get the value 4, our first integer.
3. However, the other five numbers do not seem to be stored in other registers or on the stack.
4. Also, when we look at %r12 in gdb's register display (`​layout reg`), it has the value 0x7fffffffde00, clearly a memory address.
5. Use `x/6w $r12` or `x/6w 0x7fffffffde00` to look at the six double words of memory beginning at %r12\. There we can see our six input integers sitting in an array.
6. Instructions 400f5b–400f95 loop over the array, checking to see that each number is equal to or greater than 1 and less than or equal to 6 and that the array contains no duplicates. If those conditions are met, the bomb does not explode and we move to instruction 400f97\. Since our first integer was 0, we need to start over with a test string that will get us through this loop and to 400f97: for example, **"1 2 3 4 5 6"**.
7. At instruction 400f97, another loop is set up. It runs from 400fa1–400fb2, transforming the array by replacing each element with 7 minus its value. Advance to instruction 400fb4, when this loop will have completed, and check the values in the array with `x/6w 0x7fffffffde00`. They should be 6, 5, 4, 3, 2, 1.
8. At 400fc3 data at the memory address 0x603770 is copied to %esi. Looking at that data with `x 0x603770`, the result is `<node1>: 0x00000330.`
9. "node1" suggests a linked list.
    1. If we inspect `0x603770​ + 8`, we get the result ​`<node1+8>:	0x603760`. This is the pointer to the linked node.
    2. If we inspect `0x603760`, we get `<node2>:	0x000001b8`. This is an integer element contained in the node (presumably a struct).
    3. If we inspect `0x603760 + 8`, we get `<node2+8>: 0x603750`
    4. If we inspect `0x603750`, we get `<node3>:	0x000000b1`
    5. If we inspect 0x603750 + 8, we get `<node3+8>:	0x00603740`
    6. If we inspect `0x603740`, we get `<node4>:	0x0000030e`
    7. If we inspect `0x603740 + 8`, we get `<node4+8>:	0x00603730`
    8. If we inspect 0x603730, we get `​<node5>: 0x0000004d`
    9. If we inspect 0x603730 + 8, we get `<node5+8>:	0x00603720`
    10. If we inspect 0x603720, we get `<node6>:	0x00000330`
10. Instructions 400fcd–401027 appear to be doing something with either the input array or the linked list, or both. But there's only one remaining call to `​explode_bomb,` at instruction 40103e. So let's just advance to 40103c and look at the array and the list at that point.
11. At 40103c, we check the input array with `x/6w 0x7fffffffde00`​. It has not changed.
12. However, the list has been reversed:
    1. node 1's pointer is null
    2. node 2's pointer is to node 1
    3. node 3's pointer is to node 2
    4. node 4's pointer is to node 3
    5. node 5's pointer is to node 4
    6. node 6's pointer is to node 5
13. At 40103a, two values are compared: (%rax) and %edx. %rax contains the node address and %edx contains the integer value in the node. At 40103c, `​jge`​ suggests that each node pointer is modified according to some relationship with the node integer element.
14. This suggests that the list is being ordered and that the input integers are an ordering or sort key.
15. The node integer elements are as follows:
    1. node 1: 0x330 (816)
    2. node 2: 0x1b8 (440)
    3. node 3: 0xb1 (177)
    4. node 4: 0x30e (782)
    5. node 5: 0x4d (77)
    6. node 6: 0x330 (816)
16. In ascending order, the list would be node5, node3, node2, node4, node1, node6 OR node5, node3, node2, node4, node6, node1 (there are two possible versions because the values of node 1 and node 6 are identical).
17. In descending order, the list would be node1, node6, node4, node2, node3, node5 OR node6, node1, node4, node2, node3, node5.
18. We can just try out input integer strings that correspond to these list orders.
19. Reverse the input integers and map them to the node numbers:
    1. integers 6 5 4 3 2 1
    2. nodes: 1 2 3 4 5 6
20. To rearrange the nodes in ascending order, use the input string **"2 4 5 3 6 1"** or **"2 4 5 3 1 6"**.
21. To rearrange the nodes in descending order, use the input string **"6 1 3 5 4 2"** or **"1 6 3 5 4 2"**.
22. Try these with a breakpoint set and stepping through to avoid exploding the bomb.
23. **"6 1 3 5 4 2" and "1 6 3 5 4 2" will both defuse the bomb.**


Content of psol.txt (solution file)
-----------------------------------

```
Why make trillions when we could make... billions?
0 1 1 2 3 5
0 i 755
4 2
IKLMNO
6 1 3 5 4 2
```
