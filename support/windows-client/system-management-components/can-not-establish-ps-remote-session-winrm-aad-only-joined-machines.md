---
title: Can't establish a PowerShell remote session using WinRM
description: Helps to resolve the issue in which a PowerShell remote session using Windows Remote Management (WinRM) can't be established between machines that are joined to Azure Active Directory (Azure AD) only.
ms.date: 03/09/2023
author: v-lianna
ms.author: v-lianna
manager: dcscontentpm
audience: itpro
ms.topic: troubleshooting
ms.prod: windows-client
localization_priority: medium
ms.reviewer: kaushika, gbrag
ms.custom: sap:winrm, csstroubleshoot
ms.technology: windows-client-system-management-components
---
# Can't establish a PowerShell remote session using WinRM between Azure AD-only joined machines

This article helps to resolve the issue in which a PowerShell remote session using Windows Remote Management (WinRM) can't be established between machines that are joined to Azure Active Directory (Azure AD) only.

You have two machines on the same network. They aren't joined to a local domain and only joined to Azure AD with no on-premises synchronization.

When you try to establish a PowerShell remote session using WinRM between the two machines, you receive the following error messages:

- > Enter-PSSession : Connecting to remote server CLIENT01 failed with the following error message : The WinRM client cannot process the request. If the authentication scheme is different from Kerberos, or if the client computer is not joined to a domain, then HTTPS transport must be used or the destination machine must be added to the TrustedHosts configuration setting. Use winrm.cmd to configure TrustedHosts. Note that computers in the TrustedHosts list might not be authenticated. You can get more information about that by running the following command: winrm help config. For more information, see the about_Remote_Troubleshooting Help topic.

- > Enter-PSSession : Connecting to remote server CLIENT01 failed with the following error message : WinRM cannot process the request. The following error with errorcode 0x8009030e occurred while using Negotiate authentication: A specified logon session does not exist. It may already have been terminated.  
    Possible causes are:  
     -The user name or password specified are invalid.  
     -Kerberos is used when no authentication method and no user name are specified.  
     -Kerberos accepts domain user names, but not local user names.  
     -The Service Principal Name (SPN) for the remote computer name and port does not exist.  
     -The client and remote computers are in different domains and there is no trust between the two domains.  
    After checking for the above issues, try the following:  
    -Check the Event Viewer for events related to authentication.  
    -Change the authentication method; add the destination computer to the WinRM TrustedHosts configuration setting or use HTTPS transport.  
    Note that computers in the TrustedHosts list might not be authenticated.  
    -For more information about WinRM configuration, run the following command: winrm help config. For more
    information, see the about_Remote_Troubleshooting Help topic.

This issue occurs because of one of the following reasons:

- WinRM considers the Azure AD-only joined machines as workgroup machines. Therefore, implicit credentials can't be used.
- The WinRM default Service Principal Name (SPN) prefix HTTP prevents Azure AD authentication.

## Implicit credentials can't be used

To resolve this issue, on the client machine set the `TrustedHosts` value as follows:

|Value  |Description  |
|---------|---------|
|`*`     |This value allows reusing the implicit credentials for all target machines.         |
|`*.contoso.com`     |This value restricts the usage of credentials to the machines of a specific domain.         |

For example, you can use one of the following ways to set the `TrustedHosts` value to `*.contoso.com`:

- The PowerShell cmdlet:

    ```powershell
    set-item WSMan:\localhost\Client\TrustedHosts "*.contoso.com"
    ```

- The command prompt:

    ```console
    winrm s winrm/config/client @{TrustedHosts="*.contoso.com"}
    ```

- Add a registry entry from the command prompt:

    ```console
    reg add HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\WSMAN\Client /v trusted_hosts /t REG_SZ /d "*.contoso.com" /f
    ```

## Default SPN prefix HTTP prevents Azure AD authentication

To resolve this issue, on the client machine set the default SPN prefix to `HOST` by running the following command:

```console
reg add HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\WSMAN\Client /v spn_prefix /t REG_SZ /d "HOST" /f
```
