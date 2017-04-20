### systemd 服务执行环境配置选项

在 systemd 的服务单元文件中可以设置一些进程执行环境的相关参数，比如进程的运行用户和组、最大打开文件数、进程优先级、进程的环境变量等等。这些配置项可适用于 service、 socket、 mount、 swap 等类型的服务单元

官方 systemd 原文链接: [systemd Execution environment configuration](https://www.freedesktop.org/software/systemd/man/systemd.exec.html) 

#### systemd 通用选项列表

- `WorkingDirectory`进程的工作目录
- `RootDirectory` 进程的根目录，类似使用 chroot 设置根目录
- `RootImage`
- `MountAPIVFS`
- `User` 指定运行用户，进程将以此用户身份运行
- `Group` 指定运行用户组，进程将以此用户组身份运行
- `DynamicUser`
- `SupplementaryGroups`
- `RemoveIPC`
- `Nice` 进程的运行优先级，优先级范围从高到低为 -20 到 19，数字越小优先级越高
- `OOMScoreAdjust`
- `IOSchedulingClass`
- `IOSchedulingPriority`
- `CPUSchedulingPolicy`
- `CPUSchedulingPriority`
- `CPUSchedulingResetOnFork`
- `CPUAffinity`
- `UMask` 设置进程创建文件或目录时的默认权限掩码，也就是新创建的文件或目录的默认权限
- `Environment` 设置进程运行期间的环境变量，可多次使用设置多个环境变量
- `EnvironmentFile` 从一个配置文件中读取环境变量参数
- `PassEnvironment`
- `StandardInput`
- `StandardOutput`
- `StandardError`
- `TTYPath`
- `TTYReset`
- `TTYVHangup`
- `TTYVTDisallocate`
- `SyslogIdentifier`
- `SyslogFacility`
- `SyslogLevel`
- `SyslogLevelPrefix`
- `TimerSlackNSec`
- `LimitCPU` 与 ulimit -t 相同，单位为秒
- `LimitFSIZE` 与 ulimit -f 相同，单位为字节
- `LimitDATA` 与 ulimit -d 相同，单位字节
- `LimitSTACK` 与 ulimit -s 相同，单位为字节
- `LimitCORE` 与 ulimit -c 相同，单位为字节
- `LimitRSS` 与 ulimit -m 相同，单位为字节
- `LimitNOFILE` 与 ulimit -n 相同，值类型为数字
- `LimitAS` 与 ulimit -v 相同，单位为字节
- `LimitNPROC` 与 ulimit -u 相同，值类型为数字
- `LimitMEMLOCK` 与 ulimit -l 相同，单位为字节
- `LimitLOCKS` 与 ulimit -x 相同
- `LimitSIGPENDING` 与 ulimit -i 相同
- `LimitMSGQUEUE` 与 ulimit -q 相同，单位为字节
- `LimitNICE` 与 ulimit -e 相同
- `LimitRTPRIO` 与 ulimit -r 相同
- `LimitRTTIME` 暂时没有对应参数说明
- `PAMName`
- `CapabilityBoundingSet`
- `AmbientCapabilities`
- `SecureBits`
- `ReadWritePaths`
- `ReadOnlyPaths`
- `InaccessiblePaths`
- `BindPaths`
- `BindReadOnlyPaths`
- `BindPaths`
- `PrivateTmp`
- `PrivateDevices`
- `PrivateNetwork`
- `PrivateUsers`
- `ProtectSystem`
- `ProtectHome`
- `ProtectKernelTunables`
- `ProtectKernelModules`
- `ProtectControlGroups`
- `MountFlags`
- `UtmpIdentifier`
- `UtmpMode`
- `SELinuxContext`
- `AppArmorProfile`
- `SmackProcessLabel`
- `IgnoreSIGPIPE`
- `NoNewPrivileges`
- `SystemCallFilter`
- `SystemCallErrorNumber`
- `SystemCallArchitectures`
- `RestrictAddressFamilies`
- `RestrictNamespaces`
- `Personality`
- `RuntimeDirectory`
- `RuntimeDirectoryMode`
- `MemoryDenyWriteExecute`
- `RestrictRealtime`

------------------------------
Author: typefo <typefo@qq.com>
