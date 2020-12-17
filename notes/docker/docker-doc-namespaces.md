# namespaces
> useful to hide processes, networks, and other configurations from other environments

While chroot is a pretty straightforward,
namespaces
and cgroups are a bit more nebulous to understand but no less important.

**Both of these next two features are for security and resource management.**

### why? 
Let's say you're running a big server that's in your home and you're selling space to other people (that you don't know) to run their code on your server. What sort of concerns would you have? Let's say you have Alice and Bob who are running e-commerce services dealing with lots of money. They themselves are good citizens of the servers and minding their own business. But then you have Eve join the server who has other intentions: she wants to steal money, source code, and whatever else she can get her hands on from your other tenants on the server. If just gave all three them root access to server, what's to stop Eve from taking everything? Or what if she just wants to disrupt their businesses, even if she's not stealing anything?

Your first line of defense is that you could log them into chroot'd environments and limit them to only those. Great! Now they can't see each others' files. Problem solved? Well, no, not quite yet. Despite the fact that she can't see the files, she can still see all the processes going on on the computer. She can kill processes, unmount filesystem and potentially even hijack processes.

### Enter namespaces.
Namespaces allow you to hide processes from other processes. If we give each chroot'd environment different sets of namespaces, now Alice, Bob, and Eve can't see each others' processes (they even get different process PIDs, or process IDs, so they can't guess what the others have) and you can't steal or hijack what you can't see!

There's a lot more depth to namespaces beyond what I've outlined here. The above is describing just the UTS (or UNIX Timesharing) namespace. There are more namespaces as well and this will help these containers stay isloated from each other.

The problem with chroot alone
Now, this isn't secure. The only thing we've protected is the file system, mostly.

chroot in a terminal into our environment
In another terminal, run docker exec -it docker-host bash. This will get another terminal session #2 for us (I'll refer to the chroot'd environment as #1)
Run tail -f /my-new-root/secret.txt & in #2. This will start an infinitely running process in the background.
Run ps to see the process list in #2 and see the tail process running. Copy the PID (process ID) for the tail process.
In #1, the chroot'd shell, run kill <PID you just copied>. This will kill the tail process from inside the chroot'd environment. This is a problem because that means chroot isn't enough to isolate someone. We need more barriers. This is just one problem, processes, but it's illustrative that we need more isolation beyond just the file system.
	

## configure new namespace
> Bootstrap bare min debian pkgs `apt-get install debootstrap -y`

### This will create a new environment that's isolated on the system with its own PIDs, mounts (like storage and volumes), and network stack. Now we can't see any of the processes!

```bash
# install debootstrap
apt-get update -y
apt-get install debootstrap -y
debootstrap --variant=minbase bionic /better-root

# head into the new namespace'd, chroot'd environment
unshare --mount --uts --ipc --net --pid --fork --user --map-root-user chroot /better-root bash # this also chroot's for us
mount -t proc none /proc # process namespace
mount -t sysfs none /sys # filesystem
mount -t tmpfs none /tmp # filesystem
```



#docker 