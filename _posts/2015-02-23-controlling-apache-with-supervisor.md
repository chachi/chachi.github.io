---
layout: post
title: Controlling Apache with Supervisord
---

[Supervisord](http://supervisord.org/) is a really simple way to control deployed and long-running processes in the cloud. Configuring a regular program to be run and monitored by Supervisor is dead simple. Supervisor only requires that your program stay in the foreground (not daemonize itself) and it will handle starting, stopping, logging, and monitoring your program for its lifetime.

The [Apache HTTP server](http://httpd.apache.org) is a bit different than a normal program, unfortunately. Apache is launched by calling `apache2ctl` as root (to bind to port 80), which then spawns `apache2` processes as the `www-data` user. This is A Good Thing, as it de-escalates the Apache server's user priviliges and preforks processes to handle incoming requests.

Unfortunately, under normal operation, when stopping Apache with supervisor, these child processes become orphaned: the parent process exits without stopping the children. To control the children, along with the `apache2ctl` process, add the following lines to your supervisor configuration:

```
killasgroup=true
stopasgroup=true
```

This will send `KILL` and `STOP` commands to the entire process group, rather than simply to the parent process.

Here's my entire Apache configuration:

```
[program:apache]
command=apache2ctl -c "ErrorLog /dev/stdout" -DFOREGROUND
autostart=true
autorestart=true
startretries=1
startsecs=1
redirect_stderr=true
stderr_logfile=/var/log/myapache.err.log
stdout_logfile=/var/log/myapache.out.log
user=root
killasgroup=true
stopasgroup=true
```
