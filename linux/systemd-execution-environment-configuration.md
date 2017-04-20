### systemd 服务执行环境配置选项

在 systemd 的服务单元文件中可以设置一些进程执行环境的相关参数，比如进程的运行用户和组、最大打开文件数、进程优先级、进程的环境变量等等。这些配置项可适用于 service、 socket、 mount、 swap 等类型的服务单元

官方 systemd 原文链接: [systemd Execution environment configuration](https://www.freedesktop.org/software/systemd/man/systemd.exec.html) 

#### systemd 通用选项列表

- WorkingDirectory

进程的工作目录

- RootDirectory

进程的根目录，类似使用 chroot 设置根目录

- RootImage

进程的根目录，类似使用 chroot 设置根目录

- MountAPIVFS
- User
- Group
- DynamicUser
- SupplementaryGroups
- RemoveIPC
- Nice
- OOMScoreAdjust
- IOSchedulingClass
- IOSchedulingPriority
- CPUSchedulingPolicy
- CPUSchedulingPriority
- CPUSchedulingResetOnFork
- CPUAffinity
- UMask
- Environment
- EnvironmentFile
- PassEnvironment
- StandardInput
- StandardOutput
- StandardError
- TTYPath
- TTYReset
- TTYVHangup
- TTYVTDisallocate
- SyslogIdentifier
- SyslogFacility
- SyslogLevel
- SyslogLevelPrefix
- TimerSlackNSec
- LimitCPU
- LimitFSIZE
- LimitDATA
- LimitSTACK
- LimitCORE
- LimitRSS
- LimitNOFILE
- LimitAS
- LimitNPROC
- LimitMEMLOCK
- LimitLOCKS
- LimitSIGPENDING
- LimitMSGQUEUE
- LimitNICE
- LimitRTPRIO
- LimitRTTIME
- PAMName
- CapabilityBoundingSet
- AmbientCapabilities
- SecureBits
- ReadWritePaths
- ReadOnlyPaths
- InaccessiblePaths
- BindPaths
- BindReadOnlyPaths
- BindPaths
- PrivateTmp
- PrivateDevices
- PrivateNetwork
- PrivateUsers
- ProtectSystem
- ProtectHome
- ProtectKernelTunables
- ProtectKernelModules
- ProtectControlGroups
- MountFlags
- UtmpIdentifier
- UtmpMode
- SELinuxContext
- AppArmorProfile
- SmackProcessLabel
- IgnoreSIGPIPE
- NoNewPrivileges
- SystemCallFilter
- SystemCallErrorNumber
- SystemCallArchitectures
- RestrictAddressFamilies
- RestrictNamespaces
- Personality
- RuntimeDirectory
- RuntimeDirectoryMode
- MemoryDenyWriteExecute
- RestrictRealtime
