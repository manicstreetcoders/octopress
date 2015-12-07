---
layout: post
title: "Non-blocking read on a subprocess.PIPE in python"
date: 2014-04-10 21:38:31 +0900
comments: true
categories: 
---

    p = subprocess.Popen('myprogram', stdout = subprocess.PIPE)
    str = p.stdout.readline()

로 해서 `readline()` 에서 hang 이 걸리는 경우를 당했다.

[Non-blocking read on a subprocess.PIPE in python](http://stackoverflow.com/questions/375427/non-blocking-read-on-a-subprocess-pipe-in-python) 을 읽고 해결책을 찾았다.

    import sys
    from subprocess import PIPE, Popen
    from threading import Thread

    try:
        from Queue import Queue, Empty
    except ImportError:
        from queue import Queue, Empty

    ON_POSIX = 'posix' in sys.builtin_module_names

    def enqueue_output(out, queue):
        for line in iter(out.readline, b''):
            queue.put(line)
        out.close()

    p = Popen(['myprogram'], stdout=PIPE, bufsize=1, close_fds=ON_POSIX)
    q = Queue()
    t = Thread(target=enqueue_output, args=(p.stdout, q))
    t.daemon = True
    t.start()

    try: line = q.get_nowait()
    except Empty:
        pass
    else:
        yield line
