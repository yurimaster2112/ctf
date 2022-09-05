# Initial Setup
- Disable stack protector features with gcc:
```
    gcc -fno-stack-protector -z execstack -no-pie -m64
```
- Run `checksec` on the executable to confirm that stack canary was turned off.
  Also, There is no PIE feature was on either.
# Analyze the executable with GDB-pwndbg
- Run `gdb-pwndbg` on the executable.
- Run `info functions` to see what functions are on the file.
- Run `disass` on either `main` or any particular function by using its address.
# Stack 1
- set breakpoint at. Run `info breakpoints` to see current breakpoints.
```
    b *main
```
## First Run.
- This run will try to use change the value of `agrc` in the memory address to jump to the desired code location.
- This run has no argument.Therefore,`agrc` has the value of 1. The program will exit with 1 error code. The buffer will be unharmed.
- Run `context` and `ni` to go through code instruction by instruction.
- The first three lines are use to set up stack frame. From `main+0 - main+8`. Next three lines are used to move the content of `EDI` to a memory location, offset `0x54` away from `EBP`. This value is also the value of `agrc` (which is 1) since we only run it with no argument. Then, this value is used to compared with `1`. `ZF` wil be set after this comparision. And the code will not jump.
- We can change the flow of code at this step by changing the value of `agrc`. In other word, change the value at `$EBP - 0x54`. To do this, first we need to find the address of `$EBP - 0x54`. This is obtained by `x $EBP - 0x54`. Copy the address, then run `set *address = 2` to change it value. `ZF` flag were disabled, and the code jump to `main + 46`.
- Another way to obtain the jump is to disable `ZF` flag.
## Second Run.
- This run will try to change the value of `EFLAG`. In specific, `ZF` need to
  be disable after the comparasion on `main + 15`
- run `info registers eflags` to find more about eflags, and which flag is turned on.
- Let the code run until `main + 15`. This is where the comparision happens. `ZF` flag is turned on after this. Now, we will try to disable this flag.
- `info registers eflags` shows us that the current flag has the value of `0x246`. Use python3 and run `bin(0x246)` to obtain the binary representation of it which is `0b1001000110`. `ZF` flag is at bit 7th(from right to left). Therefore, we want to change the value to `0b1000000110`. Input this value into `bin()` to obtain the hex value. It yields `0x206`. Input this value into gdb by running `set $eflags = 0b1000000110`
- *NOTE*: All above step can be obtain without using python by running `p $eflags` to print out the info of eflags. `p/x $eflags` to print hex presentation, or `p/t eflags` to print out binary presentation.
- The flow will change after this modification.
