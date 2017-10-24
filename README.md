# Fake Systemd

From https://github.com/kvaps/fake-systemd and using Debian like start-stop-deamon from https://github.com/daleobrien/start-stop-daemon written by Marek Michalkiewicz <marekm@i17linuxb.ists.pwr.wroc.pl>.

Instead of using original systemctl + dbus + priviliges + seccomp + x packages + conjunction of mercury and venus, a simple bash script using start-stop-daemon.

## Usage

The container will only compile start-stop-daemon and put the file in systemctl.
Build

    $ docker build -t ahmet2mir/fakesystemd .

Run interactive docker

    $ docker run --rm -it ahmet2mir/fakesystemd bash

Example with httpd

```
[root@ff6625414fd4 /]# yum install httpd -y
[root@ff6625414fd4 /]# systemctl start httpd

[root@ff6625414fd4 ~]# systemctl status httpd 
● httpd.service - The Apache HTTP Server
   Loaded: loaded (/usr/lib/systemd/system/httpd.service; disabled; vendor preset: not_implemented)
   Active: active (running) since Tue Oct 24 09:23:20 2017
/usr/bin/systemctl: line 236: [: man:httpd(8): binary operator expected
 Main PID: 5977 (httpd)
   Memory: 0.0%
   CGroup: /system.slice/httpd.service
           └─5977 /usr/sbin/httpd -DFOREGROUND

[root@ff6625414fd4 ~]# curl -XHEADER localhost
<!DOCTYPE HTML PUBLIC "-//IETF//DTD HTML 2.0//EN">
<html><head>
<title>501 Not Implemented</title>
</head><body>
<h1>Not Implemented</h1>
<p>HEADER to / not supported.<br />
</p>
</body></html>

[root@ff6625414fd4 ~]# systemctl stop httpd  
[root@ff6625414fd4 ~]# curl -XHEADER localhost
curl: (7) Failed connect to localhost:80; Connection refused

[root@ff6625414fd4 ~]# systemctl enable httpd
Created symlink from /etc/systemd/system/multi-user.target.wants/httpd.service to /usr/lib/systemd/system/httpd.service.
[root@ff6625414fd4 ~]# systemctl is-enabled httpd
enabled
[root@ff6625414fd4 ~]# systemctl is-active httpd
failed
[root@ff6625414fd4 ~]# systemctl status httpd
● httpd.service - The Apache HTTP Server
   Loaded: loaded (/usr/lib/systemd/system/httpd.service; enabled; vendor preset: not_implemented)
   Active: failed (Result: not_implemented)
[root@ff6625414fd4 ~]# systemctl start httpd
[root@ff6625414fd4 ~]# systemctl is-active httpd
active
```

Example with sshd

```
[root@ff6625414fd4 ~]# yum install -y openssh-clients openssh-server sshpass
[root@ff6625414fd4 ~]# systemctl start sshd
[root@ff6625414fd4 ~]# systemctl status sshd
● sshd.service - OpenSSH server daemon
   Loaded: loaded (/usr/lib/systemd/system/sshd.service; disabled; vendor preset: not_implemented)
   Active: active (running) since Tue Oct 24 09:27:30 2017
/usr/bin/systemctl: line 236: [: man:sshd(8): binary operator expected
 Main PID: 7189 (sshd)
   Memory: 0.0%
   CGroup: /system.slice/sshd.service
           └─7189 /usr/sbin/sshd -D

[root@ff6625414fd4 ~]# echo "root:docker" | chpasswd 
[root@ff6625414fd4 ~]# sshpass -p docker ssh -oStrictHostKeyChecking=no localhost uptime
 09:29:28 up 5 days, 18:14,  0 users,  load average: 0.00, 0.02, 0.05
```

## Currently supported actions

**Unit Commands:**

* start
* stop
* restart
* is-active
* status: works with the pidfile,  if a PIDfile options isn't defined, one will be created in /run/UNIT_NAME.pid.

**Unit File Commands:**

* enable
* disable
* is-enabled 

**Variables:**

* MAINPID

**Specifiers:**

* %i
* %I
* %n
* %N

**Unit Options:**

* Description
* Documentation

**Install Options:**

* WantedBy

**Service Options:**

* EnvironmentFile: if starting with a minus/dash (ex EnvironmentFile=-/etc/sysconfig/myconfig), errors are ignored
* ExecStart
* ExecStartPost
* ExecStartPre
* ExecStop
* ExecStopPost
* ExecStopPre
* PIDFile
* Type (oneshot, simple, notify and forking only)
* User
* WorkingDirectory

`Exec[Start|Stop][Post|Pre]` commands with `\` and multiples occurrence are supported well.
If a command starts with a minus/dash, the error is ignored.

# Licences

`Original fake-systemd` Copyright (c) 2017 kvaps Licence MIT https://opensource.org/licenses/MIT

`start-stop-daemon` is under public domain, see https://github.com/daleobrien/start-stop-daemon#notes

```
A rewrite of the original Debian's start-stop-daemon Perl script
in C (faster - it is executed many times during system startup).

Written by Marek Michalkiewicz <marekm@i17linuxb.ists.pwr.wroc.pl>,
public domain.  Based conceptually on start-stop-daemon.pl, by Ian
Jackson <ijackson@gnu.ai.mit.edu>.  May be used and distributed
freely for any purpose.  Changes by Christian Schwarz
<schwarz@monet.m.isar.de>, to make output conform to the Debian
Console Message Standard, also placed in public domain.  Minor
changes by Klee Dienes <klee@debian.org>, also placed in the Public
Domain.

Changes by Ben Collins <bcollins@debian.org>, added --chuid, --background
and --make-pidfile options, placed in public domain aswell.

Port to OpenBSD by Sontri Tomo Huynh <huynh.29@osu.edu>
               and Andreas Schuldei <andreas@schuldei.org>
Changes by Ian Jackson: added --retry (and associated rearrangements).

Modified for Gentoo rc-scripts by Donny Davies <woodchip@gentoo.org>:
 I removed the BSD/Hurd/OtherOS stuff, added #include <stddef.h>
 and stuck in a #define VERSION "1.9.18".  Now it compiles without
 the whole automake/config.h dance.
```
    