# 99 Windows Server

## 01 Config NTP Server

配置Windows Server的上级NTP时钟服务器

```bash
W32tm.exe /config /manualpeerlist:192.168.100.1 /syncfromflags:MANUAL
```

采用注册表配置，启用NTP Server配置

```bash
reg add HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\w32time\TimeProviders\NtpServer /v Enabled /t REG_DWORD /d 1 /f
```

查询相关配置是否生效

```bash
reg query HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\W32Time\TimeProviders\NtpServer
reg query HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\W32Time\Config
reg query HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\W32Time\Parameters

W32tm.exe /resync
W32tm.exe /config /update
W32tm.exe /query /configuration
W32tm.exe /query /status /verbose
```

NTP Server服务相关启动，停止操作

```bash
Stop-Service w32time
Start-Service w32time
net stop w32time && net start w32time
```

