debian-docker-runit
===================
A `runit` configuration to start up docker daemon 0.9.0 on Debian jessie/sid.

As of Docker 0.9.0, cgroups should be mounted individually e.g.:

    $ mount
    cgroup on /sys/fs/cgroup type tmpfs (rw,relatime,mode=755)
    cgroup on /sys/fs/cgroup/cpuset type cgroup (rw,relatime,cpuset)
    cgroup on /sys/fs/cgroup/cpu type cgroup (rw,relatime,cpu)
    cgroup on /sys/fs/cgroup/cpuacct type cgroup (rw,relatime,cpuacct)
    cgroup on /sys/fs/cgroup/memory type cgroup (rw,relatime,memory)
    cgroup on /sys/fs/cgroup/devices type cgroup (rw,relatime,devices)
    cgroup on /sys/fs/cgroup/freezer type cgroup (rw,relatime,freezer)
    cgroup on /sys/fs/cgroup/net_cls type cgroup (rw,relatime,net_cls)
    cgroup on /sys/fs/cgroup/blkio type cgroup (rw,relatime,blkio)
    cgroup on /sys/fs/cgroup/perf_event type cgroup (rw,relatime,perf_event)

I found this out when I upgraded from 0.8.1, couldn't even run `docker run -i -t busybox /bin/sh` and kept getting errors like this:

    ...
    [error] container.go:784 Error running container: mkdir /sys/fs/devices: no such file or directory
    ...
    [error] server.go:951 Error: bad file descriptor                         
    [error] server.go:86 HTTP Error: statusCode=500 bad file descriptor

I hopped onto #docker on freenode where crosbymichael and tianon pointed me in the right direction.
[#docker logs](https://botbot.me/freenode/docker/msg/11935770/)

It seems that cgroups mounting doesn't have all the kinks quite worked
out of it on Debian.  So I made use of tianon's scripts.  Here are the
steps that I used to get this up and running:

### Remove existing cgroup mount point
1. Stop your old docker daeamon
1. Remove cgroup mount point from /etc/fstab if you have it e.g.

    cgroup     /sys/fs/cgroup     cgroup     defaults     0 0

1. `sudo umount /sys/fs/cgroup`
1. You may need to `apt-get remove systemd`
1. You may need to reboot.

### Install runit and cgroupfs-mount script
This will start the docker daemon up and running and it will come back after reboots.

    $ sudo -i
    $ apt-get install runit
    $ cd /usr/local/bin 
    $ wget https://raw.github.com/tianon/cgroupfs-mount/master/cgroupfs-mount
    $ chmod a+x cgroupfs-mount
    $ cd /etc/sv
    $ git clone https://github.com/ubergarm/debian-docker-runit
    $ ln -s /etc/sv/debian-docker-runit/docker /etc/service/docker

### More commands
    $ man sv
    $ sv status docker
    $ sv stop docker
    $ sv start docker

Look mom, no [Ghost containers on daemon restart](https://github.com/dotcloud/docker/blob/release/CHANGELOG.md)!
Did anyone else notice that searching for Ghost containers is difficult due to the popular blogging platform namespace collision? ;p

### Customize
1. You can edit the /etc/sv/docker/run script with your preferred arguments.
1. Logs combine stdout/stderr and default to /var/log/docker/current 
1. Customize logs by editing /etc/sv/docker/log/{config,run} and `man svlogd`

## References
* [runit](http://smarden.org/runit/)
* [#docker freenode irc channel logs](https://botbot.me/freenode/docker/msg/11935770/)
* [tianon/cgroupfs-mount](https://github.com/tianon/cgroupfs-mount)
* [cgroupfs-mount debian package](https://ftp-master.debian.org/new/cgroupfs-mount_1.0.html)
