---
title: "理解 linux 里的 ps"
blogType: "tutorial"
weight: 1
bookToc: true
---

# 理解 linux 里的 ps

## 引入

一切的一切都得从我在使用 OpenCV 处理视频的时候说起...

```python

import cv2

while True:
    success, frame = cap.read()

    if not success:
        print("End of video or error occurred.")
        break

    # process frame...


cap.release()  # Release the video capture object
cv2.destroyAllWindows()

```

看似这个代码没有什么大问题, 但是在多次调试反复运行之后程序突然崩溃😱, 一检查发现硬盘占用高达 96 GB! 这显然不合理.

一问 ChatGPT, 突然发现, 这原来是因为我一直把 Ctrl+Z 当成了 终止 terminal 来使用, 然而实际上, 它是 suspend terminal. 当使用 `fb` 指令后, 突然发现, 一大堆之前的 程序 又回来了!

所以笔者痛定思痛, 决定恶补这一块的知识...

---
## 键盘快捷键 与 终端

既然已经搞错了 Ctrl+Z 的意思, 那么就更应该看看其它快捷键的意思了:

Terminal/Bash Shortcuts 
* Ctrl+C: Kill current running command.
* Ctrl+Z: Suspend/pause current command.
* Ctrl+L: Clear the screen.

可以发现, 我一直把 Ctrl+C 和 Ctrl+Z 搞混了...

---
## 查看后台

那么, 我们应该如何查看所有运行着的进程呢?

```bash
ps
```

输出:

```bash
  PID TTY           TIME CMD
95734 ttys000    0:00.02 -zsh
92610 ttys001    0:00.03 /bin/zsh -il
92736 ttys002    0:00.02 /bin/zsh -i
93228 ttys003    0:00.04 /bin/zsh -il
95847 ttys005    0:00.03 /bin/zsh -il
```
---

看到这里又纳闷了, `tty` 是什么?

> A TTY (teletypewriter) in Linux is a terminal interface that allows users to interact with the system by sending input and receiving output.

当我们在mac 默认终端输入 `tty` 时, 输出:

```bash
/dev/ttys000
```
而在其它 vscode 里的终端输入 `tty` 时, 分别输出了:

```bash
/dev/ttys001
/dev/ttys002
/dev/ttys003
/dev/ttys005
```

好好好, 原来我开了这么多 tty 😒.

---
那么, 我们有没有办法查看每个 tty 占用的资源?

```bash
top
```

输出:

```bash
PID    COMMAND      %CPU  TIME     #TH   #WQ  #PORTS MEM    PURG   CMPRS  PGRP
92585  Code Helper  174.3 72:25.10 19    1    277    288M+  0B     66M    92578
93624  Code Helper  127.7 50:39.71 19/1  1    212+   298M+  0B     41M    92578
92604  Code Helper  24.6  12:15.70 24    1    132    549M+  0B     187M   92578
97723  top          4.8   00:01.83 1/1   0    28     8144K  0B     0B     97723
0      kernel_task  4.6   42:22:52 573/8 0    5      40M    0B     0B     0

```

* 按 q 退出
* 按 k + PID 终止进程

---
## fb 与 bg

那么我们回到原本的任务, 当我按下 Ctrl+Z 时, 

用 `jobs` 指令, 会输出:

```bash
[1]  + suspended  /Users/lzc/Documents/my_proj/.venv/bin/python 
```

可以发现这个 job 确实被挂在了后台, 且暂停, 但是并没有被“杀死”! (好恐怖的词语...)

那么应该如何把它拉回来呢? -> 用 `fb`

```bash
[1]  + continued  /Users/lzc/Documents/my_proj/.venv/bin/python 
```

