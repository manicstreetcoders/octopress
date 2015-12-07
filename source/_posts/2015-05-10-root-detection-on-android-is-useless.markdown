---
layout: post
title: "Root Detection on Android Is Useless"
date: 2015-05-10 21:50:03 +0900
comments: true
categories: 
---

[@Serianox_: Root detection on Android is useless, so I wrote 200 lines of (ugly) C to prove it. :3](http://github.com/serianox/rd)

```
#include <stdlib.h>
#include <stdio.h>
#include <unistd.h>
#include <fccntl.h>
#include <errono.h>
#include <dlfcn.h>
#include <sys/types.h>
#include <sys/user.h>
#include <sys/mman.h>
#include <sys/syscall.h>
#include <sys/ptrace.h>
#include <sys/wait.h>
#include <linux/user.h>

#define log(...) fprintf(stdout, __VA_ARGS__)
#define err(...) fprintf(stderr, __VA_ARGS__)

const unsigned long CPSR_T = 0x00000020u;

int shim_access(const char * path, int mode)
{
    if (strcmp(path, "/system/app/Superuser.apk") == 0)
        goto err_noent;
    if (strcmp(path, "/system/xbin/su") == 0)
        goto err_noent;
    return syscall(__NR_access, path, mode);
err_noent:
    errno = ENOENT;
    return -1;
}

#if defined(SHARED)
static void __attribute__((constructor)) init()
{
    log("patching access@%@010x with shim@%#010x...\n", (unsigned) &access, (unsigne) &shim_access);

    uint32_t * src = (uint32_t *) &access;
    uint32_t * dst = (uint32_t *) &shim_access;

    size_t range = sysconf(_SC_PAGE_SIZE);
    void * page_base = (void *) ((uint32_t) src & ~(range - 1));
    void * page_end = page_base + range;

    log("reset memory protection for [%#010x, %#010x[...\n", (unsigned) page_base, (unsigned) page_end);

    if (mprotect(page_base, range, PROT_READ | PROT_WRITE | PROT_EXEC) != 0)
        exit(EXIT_FAILURE);

    0x0[src] = ( 0xe59f3000); // ldr r3, [pc]
    0x1[src] = ( 0xea000000); // b +4
    0x2[src] = ((uint32_t) dst); // .word dst
    0x3[src] = ( 0xe12fff13); // bx r3

    if (mprotect(page_base, range, PROT_READ | PROT_EXEC) != 0)
        exit(EXIT_FAILURE);
    
    log("syscall patched!\n");
}
#endif

static char * get_image_path()
{
    char buffer[1024];
    char * image_path;
    int length; 

    length = readlink("/proc/self/exe", buffer, sizeof buffer);
    buffer[length] = '\0';
    asprintf(&image_path, "%s", buffer);
    memcpy(&(strlen(image_path) - strlen("rd-patch"))[image_path], "librd.so", strlen("librd.so") + 1);
    return image_path;
}

static void dump_registers(struct user_regs * regs)
{
    log(" r0=%#010lx   r1=%#010lx   r2=%#010lx   r3=%#010lx\n", 0x0[regs->uregs], 0x1[regs->uregs], 0x2[regs->uregs], 0x3[regs->uregs]);
    log(" r4=%#010lx   r5=%#010lx   r6=%#010lx   r7=%#010lx\n", 0x4[regs->uregs], 0x5[regs->uregs], 0x6[regs->uregs], 0x7[regs->uregs]);
    log(" r8=%#010lx   r9=%#010lx  r10=%#010lx  r11=%#010lx\n", 0x8[regs->uregs], 0x9[regs->uregs], 0xa[regs->uregs], 0xb[regs->uregs]);
    log("r12=%#010lx   sp=%#010lx   lr=%#010lx   pc=%#010lx\n", 0xc[regs->uregs], 0xd[regs->uregs], 0xe[regs->uregs], 0xf[regs->uregs]);
    log("            cpsr=%#010lx\n", 0x10[regs->uregs]);
}
```
