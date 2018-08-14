---
layout: post
title: Forcing a Split Tunnel in Windows 10
---

I recently got setup with a new system and a new VPN. The new VPN uses the built-in 
Windows functionality; previously I'd been on a 3rd party system which included it's own 
client. I don't do that much work on the VPN, as I use SSH or just web tools mostly. I noticed that 
with the new setup when ever I connected to the VPN any audio conference, 
or SSH tunnel would drop. When I looked into it I found that the VPN was being setup to take up 
the entire IP4 space (0.0.0.0 to 255.255.255.255). Here are the steps that I took to help resolve this - luckily I have local admin access.

First, you have to get the current settings for your VPN while connected and 
hope that they are stable enough to use (likely they will be). ipconfig from a command line will do the trick:

```bash
>ipconfig
```

Result might look something like:

```bash
PPP adapter VPN Name:
  IPv4 Address.................... : 192.168.62.**
  Subnet Mask .................... : 255.255.255.255
  Default Gateway: .............. : 192.168.51.1
```  

So basically the VPN need the entire 192.168.*.* address space, 
e.g. 192.168.0.0/16. So first thing was I changed my local router 
to be a 10.*.*.* internal network so that there wouldn't be any confusion.

Then I followed some instructions such as 
<https://superuser.com/questions/954801/how-can-i-disable-use-default-gateway-for-remote-networks-setting-in-windows-1> 
to turn off the use of the remote gateway.

At this point I can connect to the VPN, but nothing happens because the other 
VPN settings which assume the presence of a remote gateway, 
my system can't find the remote file server or the database I want to connect to via VPN.

After some poking around, I find that I can run "route print", 
locate the number assigned to the VPN and then from an admin command prompt run 
something like: `route add 192.168.0.0 mask 255.255.0.0 0.0.0.0 if 10`

I add this to a .bat file, create a shortcut and set the shortcut to run as admin and at first it works. Then it 
doesn't. I poke around and determine that the issue is that the number assigned to the 
VPN is variable and can't be controlled. There really isn't any way to work on this 
via windows .bat other than potentially parsing the results of the route print. However, 
I find that there is a command in PowerShell that will do what I want.

Not being familiar with powershell, I have to poke around through a number of respources before 
I can create a .ps1 file that works. Basically:

```
Write-Output 'Starting VPN Connection!'Start-Process "c:\Windows\System32\rasphone.exe" -ArgumentList '-d "VPN NAME"' -waitAdd-VpnConnectionRoute -ConnectionName "VPN NAME" -DestinationPrefix 192.168.0.0/16<
Pause
```

Then I update the .bat to launch this, which might seem like a bit of extra work
 but makes running as admin easier (no need to set it on the shortcut):

``` 
PowerShell.exe -Command "&amp; {Start-Process PowerShell.exe -ArgumentList '-ExecutionPolicy Bypass -File ""%~dpn0.ps1""' -Verb RunAs}"
```

So now the .bat file starts powershell which then starts 
another powershell with admin privileges which runs the .ps1 script. The ps1 script 
then launches the rasphone to dial the VPN by name and waits for it to 
return. Finally any routes are added and I pause so that if there are any error messages I can see them.

The reverse is pretty much the same at this point (to close the VPN and cleanup):

```
Write-Output 'Closing VPN Connection!'Remove-VpnConnectionRoute -ConnectionName "VPN NAME" -DestinationPrefix 192.168.0.0/16Start-Process "c:\Windows\System32\rasphone.exe" -ArgumentList '-h "VPN NAME"' -waitPause
```

So there you go. With this new setup I can connect during an audio conference and with open SSH shells without dropping everything.


## Update

So someone managed to run a PowerShell based virus and as a result 
everyone's PowerShell got taken away.  So now I'm back to just dealing with 
connections closing, etc.. when I start/stop a VPN session.