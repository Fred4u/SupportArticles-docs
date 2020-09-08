---
title: The host's A record is registered in DNS after you choose not to register the connection's address
description: Fixes an issue where the IP address registers an A record for the host name in its primary DNS suffix zone after you clear the Register this connection's address in DNS check box under Advanced TCP/IP Settings for a network interface.
ms.date: 09/08/2020
author: delhan
ms.author: Deland-Han
manager: dscontentpm
audience: itpro
ms.topic: troubleshooting
ms.prod: windows-server
localization_priority: medium
ms.reviewer: kaushika
ms.prod-support-area-path: DNS
ms.technology: Networking
---
# The host's A record is registered in DNS after you choose not to register the connection's address

This article provides methods to fix an issue where the IP address registers an A record for the host name in its primary DNS suffix zone after you clear the **Register this connection's address in DNS** check box.

This article applies to Windows 2000. Support for Windows 2000 ends on July 13, 2010. The Windows 2000 End-of-Support Solution Center is a starting point for planning your migration strategy from Windows 2000. For more information, see the [Microsoft Support Lifecycle Policy](/lifecycle/).

_Original product version:_ &nbsp;Windows 2000  
_Original KB number:_ &nbsp;275554

## Symptoms

In Windows 2000, if you clear the **Register this connection's address in DNS** check box under Advanced TCP/IP Settings for a network interface, the IP address may register an A record for the host name in its primary DNS suffix zone.

For example, this behavior may occur if you have the following configuration:

- The DNS service is installed on the server.
- The DNS server zone is **example**.com, where the **example**.com zone can be updated dynamically.
- The server host name is Server1. **example**.com, where Server1 has two network adapters that have IP addresses of 10.1.1.1 and 10.2.2.2. If you click to clear the **Register this connection's address in DNS** check box on the network adaptor that has the IP address of 10.2.2.2 and then you delete the host record for Server1. **example**.com 10.2.2.2, the host record for Server1. **example**.com 10.2.2.2 is dynamically added back to the zone late. The unwanted registration of this record can be reproduced if you restart the DNS service on the server.

## Cause

By default, when the DNS service is installed on a computer that is running Windows 2000, it listens to all of the network interfaces that are configured by using TCP/IP. When DNS causes an interface to listen for DNS queries, the interface tries to register the host A record in the zone that matches its primary DNS suffix. The interface tries to register the host A record regardless of the settings that have been configured in the TCP/IP properties. This behavior is by design and can take place under the following circumstances:

- The DNS service is installed on the server whose configuration you're trying to change.
- The DNS zone that matches the primary DNS suffix of the server is enabled to update dynamically.

## Resolution

> [!NOTE]
> The resolution that is described in this article only works on member servers that run DNS in a domain. It does not resolve this issue on domain controller computers. For additional information about how to resolve this issue on a domain controller, click the article number below to view the article in the Microsoft Knowledge Base:

[292822](/EN-US/help/292822) Name Resolution and Connectivity Issues on Windows 2000 Domain Controller with Routing and Remote Access and DNS Installed  
To prevent a DNS server from registering an A record for a specific interface in its primary DNS suffix zone, use one of the following methods.

### Method 1

> [!IMPORTANT]
> This section, method, or task contains steps that tell you how to modify the registry. However, serious problems might occur if you modify the registry incorrectly. Therefore, make sure that you follow these steps carefully. For added protection, back up the registry before you modify it. Then, you can restore the registry if a problem occurs. For more information about how to back up and restore the registry, click the following article number to view the article in the Microsoft Knowledge Base: [322756](https://support.microsoft.com/help/322756) How to back up and restore the registry in Windows  

Configure the DNS service to publish specific IP addresses to the DNS zone. To do so, make the following registry modification:

```console
PublishAddresses
 HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\DNS\Parameters

Data type : REG_SZ
 Range : IP address [IP address]
 Default value : blank
```

This modification specifies the IP addresses that you want to publish for the computer. The DNS server creates A records only for the addresses in this list. If this entry doesn't appear in the registry, or if its value is blank, the DNS server creates an A record for each of the computer's IP addresses.

This entry is for computers that have multiple IP addresses, only a subset of which you want to publish. Typically, this prevents the DNS server from returning a private network address in response to a query when the computer has a corporate network address.

DNS reads its registry entries only when it starts. You can change entries while the DNS server is running by using the DNS console. If you change entries by editing the registry, the changes aren't effective until you restart the DNS server.

The DNS server doesn't add this entry to the registry. You can add it by editing the registry or by using a program that edits the registry.

### Method 2

Remove the interface from the list of interfaces that the DNS server listens on. To do so, follow these steps:

1. Start the DNS Management Microsoft Management Console (MMC).
2. Right-click the DNS server, and then click Properties.
3. Click the Interfaces tab.
4. Under **Listen on**, click to select the **Only the following IP addresses** check box.
5. Type the IP addresses that you want the server to listen on. Include only the IP addresses of the interfaces for which you want a host A record registered in DNS.
6. Click OK, and then quit the DNS Management MMC.

## Status

Microsoft has confirmed that this is a problem in the Microsoft products that are listed at the beginning of this article.

## More information

For additional information about how to disable dynamic registrations, click the article number below to view the article in the Microsoft Knowledge Base:

[246804](https://support.microsoft.com/help/246804) How to Enable/Disable Windows 2000 Dynamic DNS Registrations  
The registry key to disable dynamic update of the DHCP client service is:

```console
HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\Tcpip\Parameters\DisableDynamicUpdate

Data type : REG_DWORD
 Range : 0 - 1
 Default value : 0
```

> [!NOTE]
> This registry key does not resolve the issue that is outlined in this article. If the DNS server listens on a specific interface, the host A record for that interface is registered.

If you remove an IP address from the list of the DNS server's listening interfaces, the server no longer accepts DNS requests that are sent to that IP address. This option is sometimes used in situations where the DNS server is also a domain controller and has an interface that is connected to a disjointed network. For this configuration, make sure that Active Directory client computers don't direct any queries to an interface that they can't reach.