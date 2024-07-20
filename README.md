# config2tftp - copy the running configuration of a Hewlett Packard ProCurve switch to a TFTP server

The `config2tftp.exp` expect script copies the running config of a Hewlett Packard ProCurve switch to
a TFTP server.

## Pre-requisites

The first pre-requisite is that it is possible to run the `ssh` command and login
to the HP switch using a username and password. The HP switch I have tested the
`config2ftp.exp` expect script on is a HP ProCurve 2620-24 Switch (HP part number J9623A)
and the username used to ssh into the switch is `manager`.

The second pre-requisite is that the `expect` command must be installed
and be in a directory listed in the PATH environment variable.

If running the command:

```
which expect
```

does NOT display any ouput then the `expect` command is either not
installed or not in one of the directories in the PATH environment
variable.

If it is not installed then try one of:

```
sudo apt install expect
sudo dnf install expect
sudo yum install expect
```

Failing that you may need to compile `expect` from source. For help with this contact me via email at:

```
andy [[At]] cranstonhub [[Dot]] com
```

The third pre-requisite is that there is a TFTP server available that
the HP switch can use to copy it's running configuration to.
The TFTP server must allow for clients to write new files to it.

There are plenty of TFTP servers to choose from - use your favourite search engine to see
what the options are and select the best one for your environment.

I use this one:

[TFTP daemon by H. Peter Anvin](https://mirrors.edge.kernel.org/pub/software/network/tftp/tftp-hpa/)

I downloaded the file:

```
tftp-hpa-5.2.tar.gz
```

With `expect` installed and a writeable TFTP server available to your HP
switch you are all set.


## Quick start

Copy the `config2tftp.exp` expect script to a directory listed in the PATH
environment variable.

Make it executable by all users by running this command:

```
chmod a+x config2tftp.exp
```

Next run the command like this but subsitute the relevant command line argument
values for your environment:

```
config2tftp.exp --ip 192.168.1.10 --user manager --pass Secret001 --tftp 192.168.1.253
```

In the example command change the following:

+ Change `192.168.1.10` to the IP address of the HP switch that a ssh (secure shell) connection can be made to
+ Change `manager` to the user name for a ssh connection to the HP switch
+ Change `Secret001` to the password for a ssh connection to the HP switch
+ Change `192.168.1.253` to the IP address of the TFTP server that a new file can be written to

Here is a typical output from the above `config2tftp.exp` command:

```
Switch IP .........: "192.168.1.10"
Switch user .......: "manager"
Switch password ...: "*********"
TFTP server IP ....: "192.168.1.253"
*** Sending password ***
*** Logged in successfully ***

tty=ansi hp2620# show version
Image stamp:    /ws/swbuildm/rel_richmond_qaoff/code/build/xform(swbuildm_rel_richmond_qaoff_rel_richmond)
                Jan 25 2017 15:05:41
                RA.16.01.0012
                744
Boot Image:     Primary

Boot ROM Version:    RA.15.11
tty=ansi hp2620# copy running-config tftp 192.168.1.253 switch-192.168.1.10-running-config.txt
TFTP download in progress.
tty=ansi hp2620#
*** Starting logout process ***
*** Successfully logged out ***
```

Looking on the TFTP server at IP address `192.168.1.253` there is a file called:

```
switch-192.168.1.10-running-config.txt
```

and the content of this file is:

```
; J9623A Configuration Editor; Created on release #RA.16.01.0012
; Ver #0d:10.80.34.41.14.28.c3.84.9c.63.ff.37.2f:8b
hostname "hp2620"
console terminal ansi
ip route 0.0.0.0 0.0.0.0 192.168.1.254
ip routing
snmp-server community "public" unrestricted
aaa authentication ssh login public-key
vlan 1
   name "DEFAULT_VLAN"
   no untagged 1-16,22-24
   untagged 17-21,25-28
   no ip address
   exit
vlan 100
   name "HOME"
   untagged 1-16
   tagged 21
   ip address 192.168.1.10 255.255.255.0
   ip helper-address 192.168.1.254
   exit
vlan 200
   name "DEV"
   untagged 22-24
   tagged 21
   ip address 10.1.1.2 255.255.0.0
   exit
no dhcp config-file-update
password manager
```

The running configuration of the HP ProCurve switch has been successfully
copied to the TFTP server. Should that switch need to be replaced (for
example because of a hardware failure) then the configuration to apply
to the replacement switch is avialable on the TFTP server.

----------------
End of README.md
