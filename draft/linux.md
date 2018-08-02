# linux 知识点

## 快捷键
Ctrl + C 中断正在运行的程序
Crtl + D 结束键盘输入
Ctrl + Z 将当前的任务暂停,挂起,放到后台
![ctrl+Z](http://ovt2bylq8.bkt.clouddn.com/ctrl+Z.png)

```
ctrl-c 发送 SIGINT 信号给前台进程组中的所有进程。常用于终止正在运行的程序。
ctrl-z 发送 SIGTSTP 信号给前台进程组中的所有进程，常用于挂起一个进程。
ctrl-d 不是发送信号，而是表示一个特殊的二进制值，表示 EOF。
```

引用自 https://blog.csdn.net/u012787436/article/details/39722583