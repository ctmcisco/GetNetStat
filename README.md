# GetNetStat
Home of the cross-plat `Get-NetStat` replacement for the old `netstat.exe` utility on Windows

## Objectives

The Windows utility `netstat.exe` is very useful to examine connections and ports but it is text-based and OS-specific.

The cmdlet `Get-NetTCPConnection` applies to TCP only, is not available on .NET Core and has no DNS resolution.

That's why I implemented this **PowerShell** module and `Get-NetStat`: it is based on .NET Core and cross-platform, supports TCP and UDP connections, returns rich objects, and features a very fast multi-threaded DNS resolution.

Use the tab "Discussion" to join a discussion if you find issues or have suggestions.

## Requirements

Requires *PowerShell* Version 5.1 or better. Runs on *Windows PowerShell* and *PowerShell*. Runs on all platforms supported by *PowerShell*.

## Installation

To install the module, run this:

```powershell
Install-Module -Name GetNetStat -Scope CurrentUser
```

Once the module is installed, you can use the new command `Get-NetStat`:

### View All Connections

```powershell
Get-NetStat
```

```
PS> Get-NetStat

Protocol LocalIp          LocalPort RemoteIp        RemotePort           State   Pid PidName
-------- -------          --------- --------        ----------           -----   --- -------
TCP      0.0.0.0                135 0.0.0.0                  0       Listening  1148 svchost
TCP      0.0.0.0                445 0.0.0.0                  0       Listening     4 System
TCP      0.0.0.0               5040 0.0.0.0                  0       Listening  4932 svchost
TCP      0.0.0.0               5985 0.0.0.0                  0       Listening     4 System
TCP      0.0.0.0               7680 0.0.0.0                  0       Listening  9128 svchost
```

### Outgoing HTTPS: Connections (to Port 443)

Add the parameter *-Resolve* to resolve IP addresses and display host names instead:

```powershell
Get-NetStat -RemotePort 443 -Resolve
```

```
Protocol LocalIp          LocalPort RemoteIp        RemotePort           State   Pid PidName
-------- -------          --------- --------        ----------           -----   --- -------
TCP      DELL7390.spe...      49704 lb-140-82-11...        443     Established  9560 chrome
TCP      DELL7390.spe...      49729 server-13-22...        443     Established  9560 chrome
TCP      DELL7390.spe...      49730 server-143-2...        443     Established  9560 chrome
TCP      DELL7390.spe...      49454 51.103.5.186           443     Established  5464 svchost
TCP      DELL7390.spe...      49726 52.113.206.137         443     Established 13736 Teams
TCP      DELL7390.spe...      49742 40.126.1.145           443        TimeWait     0 Idle
```

### Check Port Availability

This example checks whether PowerShell Remoting port 5983 is actually listening (checking whether local host has PowerShell Remoting enabled):

```powershell
(Get-NetStat -LocalPort 5985).State -eq [NetStat+State]::Listening
```

```
True
```

### Check Applications

Show connections for specific running applications only, i.e. connections maintained by the **Chrome** browser:

```powershell
Get-NetStat -PidName chrome -Resolve
```

```
Protocol LocalIp          LocalPort RemoteIp        RemotePort           State   Pid PidName
-------- -------          --------- --------        ----------           -----   --- -------
TCP      DELL7390.spe...      49704 lb-140-82-11...        443     Established  9560 chrome
TCP      DELL7390.spe...      49696 wb-in-f188.1...       5228     Established  9560 chrome
TCP      DELL7390.spe...      49832 fra16s07-in-...        443     Established  9560 chrome
UDP      0.0.0.0               5353 DELL7390.spe...          0          Closed 14740 chrome
```

### TCP and Resolve

Show TCP-only connections with an *established* connection, and resolve host names:

```powershell
Get-NetStat -State Established -Resolve -TCP
```

```
Protocol LocalIp          LocalPort RemoteIp        RemotePort           State   Pid PidName
-------- -------          --------- --------        ----------           -----   --- -------
TCP      DELL7390.spe...      49696 wb-in-f188.1...       5228     Established  9560 chrome
TCP      DELL7390.spe...      49704 lb-140-82-11...        443     Established  9560 chrome
TCP      DELL7390.spe...      49770 storage3.local        5510     Established  5356 Synology Active Backup...
TCP      DELL7390.spe...      49832 fra16s07-in-...        443     Established  9560 chrome
TCP      DELL7390.spe...      57116 storage3.local        5510     Established  5356 Synology Active Backup...
TCP      DELL7390.spe...      49771 STORAGE3              5510     Established  5356 Synology Active Backup...
TCP      DELL7390.spe...      49772 STORAGE3              5510     Established  5356 Synology Active Backup...
TCP      DELL7390.spe...      49454 51.103.5.186           443     Established  5464 svchost
TCP      DELL7390.spe...      49726 52.113.206.137         443     Established 13736 Teams
```

## Notes

DNS-Resolution is a particularly expensive and slow task. `Get-NetStat` sports a very fast multi-threaded DNS resolution when you specify the parameter `-Resolve`.

To make DNS resolution versatile and reusable, it is performed by a separate cmdlet named `Resolve-HostNameProperty`. You can use this command separately to resolve any IP address in any list of objects.

Simply pipe the objects to `Resolve-HostNameProperty`, and specify the names of the properties that need to be resolved.

The following example demonstrates this by creating dummy objects with dummy properties with dummy IP addresses. Next, `Resolve-HostNameProperty` is resolving all IP addresses in all properties specified by using up to 80 parallel threads:

```powershell
1..255 | 
ForEach-Object { [PSCustomObject]@{IP1 = "192.168.2.$_"; IP2="40.112.72.$_"}} |
    Resolve-HostNameProperty -Property IP1, IP2
```

## Participate!

If you have additional useful examples or use-cases, or if you find issues, or if you have ideas, please participate in our [discussions](https://github.com/TobiasPSP/GetNetStat/discussions)!



