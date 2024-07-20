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

## It does not work - help !!!

As I have only tested the `config2tftp.exp` expect script against my own HP switch running at
a particular firmware version is it very likely you may have problems getting this script to
work.

DONT PANIC!

I am happy to help (see my email address above - courteous enquires are always welcome).

I am anticipating two major issues:

+ TFTP setup (e.g. the TFTP server is not allowing PUT operations)
+ The dialogue with the ssh connection to your HP switch is different to the one my switch has

Let's look at these in turn.

### TFTP Setup

Confirm you can write a file to the TFTP server.

A good way to do this is to create a small file and see if you can use the `tftp` command
to send it (i.e. `write` it) to the TFTP server.

For example on a Windows laptop you could try
opening a command prompt and running command similar to:

```
echo "This is a TFTP test" > tftptest.txt
tftp 192.168.1.253 PUT tftptest.txt
```

Change `192.168.1.253` to the IP address of your TFTP server.

Then look on the TFTP server and see if the file `tftptest.txt` has been successfully copied.

If the `tftptest.txt` file has not been successfully copied then this
needs to be fixed before the `config2tftp.exp` will work successfully.

### Dialogue issues

When I use the word `dialogue` I mean the output of the ssh command during the various
stages of the login and connection.

Common issues can be:

+ The prompts for the password may different to my HP switch
+ After login the session needs to be `elevated` with a switch command like `enable`
+ The process to logout from the switch might be different
+ The command to copy the running config to a TFTP server might be different on more recent firmware versions

The good news is that the the `config2tftp.exp` can be enhanced to handle all these and other issues.

If you are having any issue then please do following:

```
script /tmp/config2tftp.out
config2tftp.exp --ip 192.168.1.10 --user manager --pass Secret001 --tftp 192.168.1.253
exit
```

As before change the command line argument values for the switch IP etc. Send email me the `/tmp/config2tftp.out`
file (see above for my email address) and I will try and amend the `config2tftp.exp` expect script to also
work in your environment.

## Advanced usage

Once you get the `config2tftp.exp` script working on you HP switch here are some things to try:

+ Set up a cron job to get the config on a regular basis
+ Keep a history of the config
+ Compare the most recently retrieved config with the last one retrieved and raise an `alert` of some sort if it has been changed
+ Track changes found by the previous comparsion in a file so you have an "audit trail" of changes
+ Send an alert if the `config2tftp.exp` expect script cannot successfully ssh into the switch

If you have more than one HP switch and can implement all the above
then I would say you will now be in a better situation of managing
your HP switches. This will be an improvement that you can show to your
employer. Maybe that will be the time to ask for that overdue pay rise :-]

Talking pay - if you found any of this useful then just "pay it forward"
by letting someone else know about it. But if you really insist
that I should get a modest recogniton then I will be most, and very, grateful.



----------------
End of README.md
