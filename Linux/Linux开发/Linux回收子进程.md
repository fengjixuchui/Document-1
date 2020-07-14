# Linux回收子进程 

## 孤儿进程

父进程先于子进程结束，则子进程成为孤儿进程，子进程的父进程成为init进程，称为init进程领养孤儿进程。



## 僵尸进程

子进程终止，父进程尚未回收，子进程残留资源(PCB)存放于内核中，变成僵尸(Zombie)进程。

父进程通过PCB给子进程报仇。



另辟蹊径：

连续使用两次fork避免僵尸进程。

![image-20200616171551843](https://raw.githubusercontent.com/supermanc88/ImageSources/master/image-20200616171551843.png)



> fork出子进程后，子进程马上fork出孙进程，子进程马上关闭，那么孙进程就会被`init`进程领养，就不会存在僵尸进程了。

### wait函数

功能：

1.  阻塞等待子进程能出
2. 回收子进程残留资源
3. 获取子进程结束状态（退出原因，在Linux中所有的异常退出都是因为收到信号 ）

查看子进程结束状态：

1. WIFEXITED 为非0 进程正常结束
2. WEXITSTATUS 如上宏为真，使用此宏，获取进程退出状态(exit的参数)
3. WIFSIGNALED 为非0 进程异常终止
4. WTERMSIG 如上宏为真，使用此宏，获取进程终止的那个信号



如果有多个子进程，哪个子进程先死，wait就等到哪个进程。



### waitpid函数

作用同wait，但可指定PID进程清理，可以不阻塞。

```c
       #include <sys/wait.h>

       pid_t waitpid(pid_t pid, int *stat_loc, int options);
```

参数pid：

1. \>0 回收指定子进程
2. =-1 回收任意子进程
3. =0 回收和当前调用waitpid同一组的所有子进程
4. <-1 回收指定进程组的任意子进程

`options`可以指定非阻塞`WNOHANG`





wait和waitpid每次调用只能回收一个子进程