---
layout: post
title: Execuate a script on shutdown with init.d and chkconfig
---

Referencing a number of sources, I recently implemented a simple trigger of a script on system shutdown using `init.d`.

## Init Script
The following steps are based on [a simplified `init.d` script](https://gist.github.com/marckysharky/69b4930fe087d47dd9d8b7018d40b44c#file-etc_init-d_on-shutdown). For the purposes of this example, we will name the script `on-shutdown`.

This should be created within the `/etc/init.d/` directory, with owner `root:root`, and permissions of `0755`.

### Key Points

#### [chkconfig](https://gist.github.com/marckysharky/69b4930fe087d47dd9d8b7018d40b44c#file-on-shutdown-sh-L5)
```
# chkconfig: 12345 95 02
```
This will be used later to declare the appropriate symlinks for `chkconfig` to create. Breaking this statement down:

- Start/Acticate on runlevels `1,2,3,4 and 5`
- Start set with a priority of `95` (low priority)
- Stopped/Deactivate with the priority of `02` (high priority)

#### [Start Function](https://gist.github.com/marckysharky/69b4930fe087d47dd9d8b7018d40b44c#file-on-shutdown-sh-L20-L24)
```
start() {
  touch /var/lock/subsys/on-shutdown
  touch /var/log/on-shutdown.log
  echo
}
```

##### [Lock File](https://gist.github.com/marckysharky/69b4930fe087d47dd9d8b7018d40b44c#file-on-shutdown-sh-L21)
```
touch /var/lock/subsys/on-shutdown
```

This command is important, as it informs the system to invoke the `stop` function on the `init` script named `on-shutdown` when the system is shutdown.

#### [Stop Function](https://gist.github.com/marckysharky/69b4930fe087d47dd9d8b7018d40b44c#file-on-shutdown-sh-L25-L29)
```
stop() {
  echo 'on-shutdown' >> /var/log/on-shutdown.log
  rm -rf /var/lock/subsys/on-shutdown
  echo
}
```

##### [Execution](https://gist.github.com/marckysharky/69b4930fe087d47dd9d8b7018d40b44c#file-on-shutdown-sh-L26)
```
echo 'on-shutdown' >> /var/log/on-shutdown.log
```

Insert/reference your desired actions on shutdown. In the example above we are simply logging to a file.

#### [Lock File Removal](https://gist.github.com/marckysharky/69b4930fe087d47dd9d8b7018d40b44c#file-on-shutdown-sh-L27)
```
rm -rf /var/lock/subsys/on-shutdown
```

Remove the lock file from the filesystem to notify the shutdown that we are complete, and to move forward.

## Registering the Init Script
When a system runlevel is started or stopped, the `/etc/rc*.d` directory is traversed to discover what needs to be invoked.

We require our script to be started on `runlevel 3` (default user mode) and stopped on `runlevel 0` (system shutdown).

To trigger the `start` function of the script, we require a symlink to the init script at the path `/etc/rc3.d/S95on-shutdown`:

- `rc3.d` - the `runlevel` of three (default user run level)
- `S` - the action (Start)
- `95` - the priority of the invocation (low)
- `on-shutdown` - the specific name of the script

To trigger the `stop` function of the script, we require a symlink to the init script at the path `/etc/rc0.d/K02on-shutdown`:

- `rc0.d` - the `runlevel` of zero (shutdown)
- `K` - the action (Kill)
- `02` - the priority of the invocation (high)
- `on-shutdown` - the specific name of the script

To achieve this we use the `chkconfig` command.

```
chkconfig --add on-shutdown
```

This will read our `/etc/init.d/on-shutdown` script, add use the declared [chkconfig](#chkconfighttpsgistgithubcommarckysharky69b4930fe087d47dd9d8b7018d40b44cfile-on-shutdown-sh-l5) attribute to create our desired symlinks.

A quick check of the directories should show the symlinks have been created:

```
ls -al /etc/rc0.d/ | grep on-shutdown
lrwxrwxrwx  1 root root   01 Jan 84 09:00 K02on-shutdown -> ../init.d/on-shutdown

ls -al /etc/rc3.d/ | grep on-shutdown
lrwxrwxrwx  1 root root   01 Jan 84 09:00 S95on-shutdown -> ../init.d/on-shutdown
```

## That's it
The system should now invoke the `stop` function of our `on-shutdown` script, when the system is shutdown.

The practical application of this has been on Amazon EC2 shutdown. In order the steps above I used `cloud-init`, which required me to invoke `service on-shutdown start`.

I have included an example `cloud-init` script in [the Gist](https://gist.github.com/marckysharky/69b4930fe087d47dd9d8b7018d40b44c#file-cloud-init).

## References
- [tldp.org - Starting Your Software Automatically on Boot](http://www.tldp.org/HOWTO/HighQuality-Apps-HOWTO/boot.html)
- [linux.com - An introduction to services, runlevels, and rc.d scripts](https://www.linux.com/news/introduction-services-runlevels-and-rcd-scripts)
- [lifewire.com - chkconfig](https://www.lifewire.com/chkconfig-linux-command-4091890)
- [Gist](https://gist.github.com/marckysharky/69b4930fe087d47dd9d8b7018d40b44c)
