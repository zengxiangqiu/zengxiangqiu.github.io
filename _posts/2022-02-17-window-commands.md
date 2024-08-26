---
title: "window commands"
date:  2022-02-17 12:19:45 +0800
categories: [其他]
tags: [工具]
---

```powershell
# smb server
Get-SmbServerConfiguration | Select EnableSMB2Protocol,EnableSMB1Protocol
# detect
Get-WindowsOptionalFeature -Online -FeatureName SMB1Protocol
# detect service is running
sc.exe query mrxsmb10
```
[How to detect, enable and disable SMBv1, SMBv2, and SMBv3 in Windows](https://learn.microsoft.com/en-us/windows-server/storage/file-server/troubleshoot/detect-enable-and-disable-smbv1-v2-v3?tabs=server)

[How to Check, Enable or Disable SMB Protocol Versions on Windows?](https://woshub.com/smb-1-0-support-in-windows-server-2012-r2/)

Windows 7 SP1 does not contain any major improvements; it’s basically a rollup of updates that have been released for the operating system since it went to manufacturing July 22 2009. If you have been diligently updating your computer through Windows Update since then, you basically have all that SP1 has to offer.


[在 Windows 10 上查找 CPU 内核数的 4 种方法](https://www.top-password.com/blog/find-number-of-cores-in-your-cpu-on-windows-10/)

[如何在 Linux 中从命令行查找 CPU 核心数](https://ostechnix.com/find-number-cpu-cores-commandline-linux/#:~:text=To%20find%20out%20the%20CPU%20cores%2C%20run%20top,CPU%20core%20details%20from%20%22%20%2Fproc%2Fcpuinfo%20%22%20file.)


`w32tm /config /manualpeerlist:"ntp1.aliyun.com" /syncfromflags:manual /reliable:yes /update`

同步系统时钟


[Enable TLS 1.2 on Windows 7](https://windowsreport.com/enable-tls-1-2-windows-11/#:~:text=What%20is%20the%20command%20to%20check%20the%20TLS,the%20following%20command%3A%20Get-TlsCipherSuite.%204%20Press%20Enter.%20Y)


