---
layout: post
title:  "Unpublished template"
published: false
---

## Build Docker container that's running multiple processes

### Despite...

...the official Docker documentation, which is 100% in alignment with the
best practices of Micro-Services design, says:

> A container’s main running process is the `ENTRYPOINT` and/or `CMD` at the end
of the `Dockerfile`. It is generally recommended that you separate areas of
concern by using one service per container. That service may fork into
multiple processes (for example, Apache web server starts multiple worker
processes). It’s ok to have multiple processes, but to get the most benefit
out of Docker, avoid one container being responsible for multiple aspects of
your overall application.

### ...in practice...

...when benefits from encapsulation and abstracting away from
internal complexity outweighs the previous rule of thumb, we may want to bundle
several processes (apps/services) into one container.

### Problem #1

The way that Docker containers are run, is that we specify the command to run in
the `Dockerfile` and when the runtime figures out that that command finished
execution it stops the container as well.

That is exactly the reason why in container we need to start apps like
`appached` in a **foreground** that are normally ran in daemon mode (background)
in non-container environments.

### Problem #2

When docker container runs one process and the process terminates, the container
runtime (depending on the configuration) will restart the container and the
process/service is available again.

This approach would not help if there was another (not main and therefore not
monitored) process in the background that failed.

### Supervisor to the rescue

> Supervisor is a client/server system that allows its users to monitor and
control a number of processes on UNIX-like operating systems

http://supervisord.org/ - is going to allow to run multiple processes in a
container and monitor them.

The following sample `Dockerfile` and `supervisord.conf` can be used as a sample:
```
FROM centos:7.3.1611

RUN yum -y install python-setuptools
RUN easy_install supervisor

ADD ./supervisord.conf /etc/supervisord.conf

RUN mkdir -p /opt/apps/logs
ADD ./app1.sh /opt/apps/app1.sh
ADD ./app2.sh /opt/apps/app2.sh

CMD supervisord -n
```

```
[supervisord]
logfile=/tmp/supervisord.log ; (main log file;default $CWD/supervisord.log)
logfile_maxbytes=50MB        ; (max main logfile bytes b4 rotation;default 50MB)
logfile_backups=10           ; (num of main logfile rotation backups;default 10)
loglevel=info                ; (log level;default info; others: debug,warn,trace)
pidfile=/tmp/supervisord.pid ; (supervisord pidfile;default supervisord.pid)
nodaemon=false               ; (start in foreground if true;default false)
minfds=1024                  ; (min. avail startup file descriptors;default 1024)
minprocs=200                 ; (min. avail process descriptors;default 200)

[rpcinterface:supervisor]
supervisor.rpcinterface_factory = supervisor.rpcinterface:make_main_rpcinterface

[supervisorctl]
serverurl=unix:///tmp/supervisor.sock ; use a unix:// URL  for a unix socket

[program:app1]
user=root
command=/opt/apps/app1.sh
stdout_logfile=/opt/apps/logs/%(program_name)s.log
stderr_logfile=/opt/apps/logs/%(program_name)s.log

[program:app2]
user=root
command=/opt/apps/app2.sh
stdout_logfile=/opt/apps/logs/%(program_name)s.log
stderr_logfile=/opt/apps/logs/%(program_name)s.log
```

For more configuration parameters check http://supervisord.org/configuration.html
