### systemd 执行环境配置

在 systemd 的服务单元文件中可以设置一些进程执行环境的相关参数，比如进程的运行用户和组、最大打开文件数、进程优先级、进程的环境变量等等。这些配置项可适用于 service、 socket、 mount、 swap 等类型的服务单元

官方 systemd 原文链接: [systemd.exec — Execution environment configuration](https://www.freedesktop.org/software/systemd/man/systemd.exec.html) 

#### systemd 通用选项列表

- `WorkingDirectory` 进程的工作目录

- `RootDirectory`       进程的根目录，类似使用 chroot 设置根目录

- `RootImage`               进程的根目录，类似使用 chroot 设置根目录
