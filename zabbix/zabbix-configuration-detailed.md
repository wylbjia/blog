# Zabbix配置文件详解

## zabbix_server.conf (服务端)
   
```
ListenPort=10051
```
   
```
SourceIP=
```
   
```
LogType=file
```
   
```
LogFile=/tmp/zabbix_server.log
```
   
```
LogFileSize=1
```
   
```
DebugLevel=3
```
   
```
PidFile=/tmp/zabbix_server.pid
```
   
```
DBHost=localhost
```
   
```
DBName=zabbix
```
   
```
DBSchema=
```
   
```
DBUser=zabbix
```
   
```
DBPassword=
```
   
```
DBSocket=/tmp/mysql.sock
```
   
```
DBPort=3306
```
   
```
StartPollers=5
```
   
```
StartIPMIPollers=0
```
   
```
StartPollersUnreachable=1
```
   
```
StartTrappers=5
```
   
```
StartPingers=1
```
   
```
StartDiscoverers=1
```
   
```
StartHTTPPollers=1
```
   
```
StartTimers=1
```
   
```
StartEscalators=1
```
   
```
JavaGateway=
```
   
```
JavaGatewayPort=10052
```
   
```
StartJavaPollers=0
```
   
```
StartVMwareCollectors=0
```
   
```
VMwareFrequency=60
```
   
```
VMwarePerfFrequency=60
```
   
```
VMwareCacheSize=8M
```
   
```
VMwareTimeout=10
```
   
```
SNMPTrapperFile=/tmp/zabbix_traps.tmp
```
   
```
StartSNMPTrapper=0
```
   
```
ListenIP=127.0.0.1
```
   
```
HousekeepingFrequency=1
```
   
```
MaxHousekeeperDelete=5000
```
   
```
SenderFrequency=30
```
   
```
CacheSize=8M
```
   
```
CacheUpdateFrequency=60
```
   
```
StartDBSyncers=4
```
   
```
HistoryCacheSize=16M
```
   
```
HistoryIndexCacheSize=4M
```
   
```
TrendCacheSize=4M
```
   
```
ValueCacheSize=8M
```
   
```
Timeout=3
```
   
```
TrapperTimeout=300
```
   
```
UnreachablePeriod=45
```
   
```
UnavailableDelay=60
```
   
```
UnreachableDelay=15
```
   
```
AlertScriptsPath=${datadir}/zabbix/alertscripts
```
   
```
ExternalScripts=${datadir}/zabbix/externalscripts
```
   
```
FpingLocation=/usr/sbin/fping
```
   
```
Fping6Location=/usr/sbin/fping6
```
   
```
SSHKeyLocation=
```

```
LogSlowQueries=3000
```
   
```
TmpDir=/tmp
```
   
```
StartProxyPollers=1
```
   
```
ProxyConfigFrequency=3600
```
   
```
ProxyDataFrequency=1
```
   
```
AllowRoot=0
```
  
```
User=zabbix
```
   
```
Include=/usr/local/etc/zabbix_server.conf.d/*.conf
```
  
```
SSLCertLocation=${datadir}/zabbix/ssl/certs
```
   
```
SSLKeyLocation=${datadir}/zabbix/ssl/keys
```
   
```
SSLCALocation=
```
   
```
LoadModulePath=${libdir}/modules
```
   
```
LoadModule=
```
   
```
TLSCAFile=
```
   
```
TLSCRLFile=
```
   
```
TLSCertFile=
```
   
```
TLSKeyFile=
```
   
## zabbix_proxy.conf (代理端)
   
```
ProxyMode=0
```
   
```
Server=127.0.0.1
```
   
```
ServerPort=10051
```
   
```
Hostname=Zabbix proxy
```
   
```
HostnameItem=system.hostname
```

```
ListenPort=10051
```

```
SourceIP=
```

```
LogType=file
```

```
LogFile=/tmp/zabbix_proxy.log
```
   
```
LogFileSize=1
```
   
```
DebugLevel=3
```
  
```
PidFile=/tmp/zabbix_proxy.pid
```
   
```
DBHost=localhost
```
   
```
DBName=zabbix_proxy
```
   
```
DBSchema=public
```
   
```
DBUser=zabbix
```
   
```
DBPassword=
```
   
```
DBSocket=/tmp/mysql.sock
```
   
```
DBPort=3306
```
   
```
ProxyLocalBuffer=0
```
   
```
ProxyOfflineBuffer=1
```
   
```
HeartbeatFrequency=60
```
   
```
ConfigFrequency=3600
```
   
```
DataSenderFrequency=1
```
   
```
StartPollers=5
```
   
```
StartIPMIPollers=0
```
   
```
StartPollersUnreachable=1
```
   
```
StartTrappers=5
```
   
```
StartPingers=1
```
   
```
StartDiscoverers=1
```
   
```
StartHTTPPollers=1
```
   
```
JavaGateway=
```
   
```
JavaGatewayPort=10052
```
   
```
StartJavaPollers=0
```
   
```
StartVMwareCollectors=0
```
   
```
VMwareFrequency=60
```
   
```
VMwarePerfFrequency=60
```
   
```
VMwareCacheSize=8M
```
   
```
VMwareTimeout=10
```
   
```
SNMPTrapperFile=/tmp/zabbix_traps.tmp
```
   
```
StartSNMPTrapper=0
```
   
```
ListenIP=0.0.0.0
```
   
```
HousekeepingFrequency=1
```
   
```
CacheSize=8M
```
  
```
StartDBSyncers=4
```
      
```
HistoryCacheSize=16M
```
   
```
HistoryIndexCacheSize=4M
```
   
```
Timeout=3
```
  
```
TrapperTimeout=300
```
   
```
UnreachablePeriod=45
```
   
```
UnavailableDelay=60
```
   
```
UnreachableDelay=15
```
   
```
ExternalScripts=${datadir}/zabbix/externalscripts
```
   
```
FpingLocation=/usr/sbin/fping
```
   
```
Fping6Location=/usr/sbin/fping6
```
   
```
SSHKeyLocation=
```
   
```
LogSlowQueries=3000
```
   
```
TmpDir=/tmp
```
   
```
AllowRoot=0
```
   
```
User=zabbix
```
   
```
Include=/usr/local/etc/zabbix_proxy.conf.d/*.conf
```
   
```
SSLCertLocation=${datadir}/zabbix/ssl/certs
```
   
```
SSLKeyLocation=${datadir}/zabbix/ssl/keys
```
   
```
SSLCALocation=
```
 
```
LoadModulePath=${libdir}/modules
```
   
```
LoadModule=
```
   
```
TLSConnect=unencrypted
```
   
```
TLSAccept=unencrypted
```
   
```
TLSCAFile=
```
   
```
TLSCRLFile=
```
   
```
TLSServerCertIssuer=
```
   
```
TLSServerCertSubject=
```
   
```
TLSCertFile=
```
   
```
TLSKeyFile=
```
   
```
TLSPSKIdentity=
```
   
```
TLSPSKFile=
```
   
## zabbix_agentd.conf (客户端)

```
PidFile=/tmp/zabbix_agentd.pid
```
   
```
LogType=file
```

```
LogFile=/tmp/zabbix_agentd.log
```   

```
LogFileSize=1
```   

```
DebugLevel=3
```   

```
SourceIP=
```   

```
EnableRemoteCommands=0
```   

```
LogRemoteCommands=0
```   

```
Server=127.0.0.1
```   

```
ListenPort=10050
```   

```
ListenIP=0.0.0.0
```   

```
StartAgents=3
```   

```
ServerActive=127.0.0.1
```   

```
Hostname=Zabbix server
```   

```
HostnameItem=system.hostname
```   

```
HostMetadata=
```   

```
HostMetadataItem=
```   

```
RefreshActiveChecks=120
```   

```
BufferSend=5
```   

```
BufferSize=100
```   

```
MaxLinesPerSecond=20
```   

```
Timeout=3
```   

```
AllowRoot=0
```   

```
User=zabbix
```   

```
Include=/usr/local/etc/zabbix_agentd.conf.d/*.conf
```   

```
UnsafeUserParameters=0
```   

```
UserParameter=
```   

```
LoadModulePath=${libdir}/modules
```   

```
LoadModule=
```   

```
TLSConnect=unencrypted
```   

```
TLSAccept=unencrypted
```   

```
TLSCAFile=
```   

```
TLSCRLFile=
```   

```
TLSServerCertIssuer=
```   

```
TLSServerCertSubject=
```   

```
TLSCertFile=
```   

```
TLSKeyFile=
```   

```
TLSPSKIdentity=
```   

```
TLSPSKFile=
```

----------------------------------------------------------------------------------------

By typefo typefo@qq.com Update: 2017-04-29 本文档使用 CC-BY 4.0 协议 ![by](../img/by.png)
