---
layout: post
title: "Python: Run a process and kill it"
date: 2014-04-08 17:08:51 +0900
comments: true
categories: 
---

[Python: Run a process and kill it if it doesn't end within one hour](http://stackoverflow.com/questions/1359383/python-run-a-process-and-kill-it-if-it-doesnt-end-within-one-hour)

프로세스 생성은 다음과 같이...

    import subprocess
    subp = subprocess.Popen(['progname']

그리고 슬립 루프를 돌면서 타임아웃을 체크한다.

    import psutil, time

    TIMEOUT = 60 * 60 # 1 hour

    p = psutil.Process(subp.pid)
    while 1:
        if (time.time() - p.create_time) > TIMEOUT:
            p.kill()
            raise RuntimeError('timeout')
        time.sleep(5)

또는

    import psutil

    p = psutil.Process(subp.pid)
    try
        p.wait(timeout=60*60)
    except psutil.TimeoutExpired:
        p.kill()
        raise

또는

    import subprocess
    proc = subprocess.Popen(['progname']

로 생성한 `proc` 을...

    import time

    def wait_timeout(proc, seconds)
        start = time.time()
        end = start + seconds
        interval = min(seconds / 1000.0, .25)

        while True:
            result = proc.poll()
            if result is not None:
                return result
            if time.time() >= end:
                raise RuntimeError("Process timed out")
            time.sleep(interval)
