## Windows Checklist - Initial
* * *
### Passwords
* Get a list of local users/groups and remove/disable unnecessary ones
    * As admin:
    * `net user`, `net user <username>`
    * `net user <username> /active:no`
    * There IS a difference between `net ...` and `net ... /domain`
        * Unless you're the DC
* Change passwords
    * `net user <username> <newpass>`
    * `net user <username> /random` 

### Misc. lockdown
* powershelL: `set-executionpolicy restricted`
* powershell: `disable-PSRemoting -Force`
* Disable camera and mic drivers
    * Or print some sexy pictures to place in front of the webcam
* Change folder view options
    * Show hidden files
    * unhide extensions
    * unhide protected OS files


### Changes for Domain wide GPO
* Computer Configuration - Policies - Windows Settings - Security Settings
    * Local Policies - Security Options
    * Accounts: Rename administrator account
    * Accounts: Rename guest account
    * Interactive logon: Do not display last user name
    * Interactive logon: Number of previous logons to cache - 0
* Authentication
    * Account lockout, password policy, audit policy from Domain Wide GPO
        ```
Computer Configuration - Policies - Windows Settings - Security Settings
Account Policies - Password Policy
Account Policies - Account Lockout Policy
Local Policies - Audit Policy
```
* GPO
    * Refresh Interval from Domain Wide GPO
        ```
Computer Configuration - Policies - Administrative Templates - System - Group Policy
- Set Group Policy refresh interval for computers - 5 Mins - 5 Mins
- Set Group Policy refresh interval for Domain Controllers - 5 Mins - 5 Mins
```
* Task scheduler
    * Disable from domain wide GPO and enable from Domain controller GPO
```
Computer Configuration - Policies - Windows Settings - System services - Task scheduler - Automatic
```
    * Disable Server oeprators from task scheduling - Domain Wide GPO
```
Computer Configuration - Policies - Windows Settings - Security Settings - Local Policies - Security Options - Domain Controller: Allow server operators to schedule tasks - Disabled
```
* Add mimikatz Protection
    * download updates from `https://support.microsoft.com/en-us/kb/2871997
```
Computer Configuration - Preferences - Registry - Create 2 Registry Keys
    - HKLM\System\CurrentControlSet\Lsa
        - RunAsPPL - DWORD - 00000001 (Hex)
        - DisableRestrictedAdmin - DWORD - 00000000 (Hex)
    - HKLM\System\CurrentControlSet\Control\SecurityProviders\WDigest
        - UseLogonCredential - DWORD - 00000001 (Hex)
```
* Remote Powershell ISE
```
Computer Configuration - Policies - Administrative Templates - Windows Components - Windows Powershell - Turn on script execution - Disabled
```

### Active Directory User Management
* Create new user for other systems to login (non-administrator), also change the username of this account
* Add admins to the "Protected Users" group (if it exists)
* Remove unidentified `Users`, `Computers`, and `Domain Controllers`
* Look for other users with admin privileges
    * ```DomainAdmins,EnterpriseAdmins,SchemaAdmins,ServerOperators,NetworkConfigurationOperators,GroupPolicyCreatorOwners,BackupOperators,AccountOperators,ProtectedUsers,IIS_IURS,DnsAdmins,Replicator,HyperVAdministrators,CryptographicOperators,PrintOperators,RemoteManagementUsers```
    * Disable `Guest` and `Domain Guests`
* Disable RDP
    * If needed, enable it and restrict access by adding only the `Administrator` account to `Remote Desktop Users` group and Enable Secure RDP



### Take inventory
* `live.sysinternals.com` is your master
    * Ctrl-R(run): `\\live.sysinternals.com\tools\toolname.exe`
* Process Explorer
    * cmd, powershell, rundll32 are all suspicious 
        * Also look for extra DLLs in a normal process
    * Suspend processes if they're being restarted
    * Wrong icon, location, title
    * colors
        * Purple - encrypted/obfuscated
            * properties->verify->no signature present
        * red - just disappeared
        * pink - services
        * cyan - app container/extra secure
        * black - suspended
* Autoruns
    * Options - scan/verify code signatures, hide empty/verified/MS signed
    * Right click entry - jump to image/entry (exe, run key, etc)
    * Malware may monitor and re-add file. Double check it's gone
* Sigcheck
    * `sigcheck -e -h -u <folder>`
    * `sigcheck -vr -vt -h -u <folder>`
    * `-e` scan for exe
    * `-v[rs]` Submit to virustotal (hashes, s->file r->view report)
        * `-vt` accept virustotal terms of service so you don't get prompted
    * `-h` list hashes
    * `-u` only show unsigned or not-vt-cleared
* Old commands for services
    * `net start`
    * `sc query`, `sc qc`
* Sysmon
    * docs: `https://docs.microsoft.com/en-us/sysinternals/downloads/sysmon`
    * `sysmon -accepteula -i <config> -l -h md5 -n`
        * -i install [from config file]
        * -l loaded modules (DLLs)
        * -h list hashes
        * -n network monitoring
    * configs:
        * top google result `https://github.com/SwiftOnSecurity/sysmon-config`
        * one from the example is from a fork from ^
* Process Monitor
    * `Filter-Filter-Operation-Begins with- {TCP [Always], UDP[Rare]}`
* Look for Poison Ivy
    * Remove Files
        * `%AllUsersProfile%\random.exe`
        * `%AppData%\Roaming\Microsoft\Windows\Templates\random.exe`
        * `%AllUsersProfile%\Application Data\.dll`
    * Remove Registry entries
        * `HKCU\Software\Microsoft\Windows\CurrentVersion`
            * random `.exe`
            * run `Random`
            * Internet Settings `CertificateRevocation` =Random
                * Set it to DWORD = 1 (Hex)
        * `HKLM\Software\Microsoft\Windows NT\CurrentVersion\
            * Random

### Firewall
* Local management: DROP inbound, ALLOW outbound
* Firewall -> (right) New Rule->TCP,all->Block
* Newer Windows servers: ssh may be enabled (ew)
* Deny rundll32.exe in/out-bound access - prevent DLL beacons
    * `C:\Windows\System32\rundll32.exe`
    * `C:\Windows\SysWOW64\rundll32.exe`
    * `C:\Windows\SysSxS\...`
        * Multiple exe instances may exist, do a file search

### backups
* Snipping tool
* Save to file 
    * `autorunsc -ct -a [opts]`
    * opts: `s` for services, blank for all
    * `-ct` tab-delimited (`-c` for csv)
    * `-s` verify digital signatures
    * `-u` only show unsigned or not-vt-cleared