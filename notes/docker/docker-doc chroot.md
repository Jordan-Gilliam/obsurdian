# chroot
 It's a Linux command that allows you to set the root directory of a new process. 

> "change root" 
 In our container use case, we just set the root directory to be where-ever the new container's new root directory should be.

Now the new container group of processes can't see anything outside of it.
- this eliminates our security problem because the new process has no visibility outside of its new root

---

### 1: Spin up an Ubuntu VM
- This example uses Docker **containers within containers 🤯** 

```bash 
# This will download the official Ubuntu container from Docker Hub 
# and grab the version marked with the bionic tag.
docker run -it --name docker-host --rm --privileged ubuntu:bionic
```

 `docker run` means we're going to run some commands in the container
 
  `-it` means we want to make the shell interactive 
  - so we can use it like a normal terminal

To see what version of Ubuntu you're using, **RUN:**
```bash
cat /etc/issue/
```

`cat` **reads** a file and **dumps** it into the output
> which means we can read it

 `/etc/issue` is a file that will tell us what distro we're using.

### 2: Attempt to use `chroot`  now

```bash 
# Make a new folder in your home directory
mkdir /my-new-root

# Inside that new folder 
echo "my super secret thing" >> /my-new-root/secret.txt
```

Now try to **RUN:**
```bash
chroot /my-new-root bash
```

You should see this ERROR: `chroot: failed to run command 'bash': No such file or directory`

This is because bash is a program and your new root wouldn't have bash to run (because it can't reach outside of its new root.)

So let's fix that! **RUN:**
```bash
mkdir /my-new-root/bin
cp /bin/bash /bin/ls /my-new-root/bin/
chroot /my-new-root bash
```

Still not working! The problem is that these commands rely on libraries to power them and we didn't bring those with us.

### 3: Fix errors + provision new root
 So let's do that too. **RUN:**
```bash 
ldd /bin/bash
```

This print out something like this:

`$ ldd /bin/bash
  linux-vdso.so.1 (0x00007fffa89d8000)
  libtinfo.so.5 => /lib/x86_64-linux-gnu/libtinfo.so.5 (0x00007f6fb8a07000)
  libdl.so.2 => /lib/x86_64-linux-gnu/libdl.so.2 (0x00007f6fb8803000)
  libc.so.6 => /lib/x86_64-linux-gnu/libc.so.6 (0x00007f6fb8412000)
  /lib64/ld-linux-x86-64.so.2 (0x00007f6fb8f4b000)`
  
These are the libraries we need for bash. Let's go ahead and copy those into our new environment. **RUN:**

```bash
mkdir /my-new-root/lib /my-new-root/lib64

# or you can do /my-new-root/lib{,64} if you want to be fancy
# Then we need to copy all those paths (ignore the lines that don't have paths) # into our directory. Make sure you get the right files in the right directory. # In the case above (yours likely will be different) it'd be two commands:

cp /lib/x86_64-linux-gnu/libtinfo.so.5 /lib/x86_64-linux-gnu/libdl.so.2 /lib/x86_64-linux-gnu/libc.so.6 /my-new-root/lib

cp /lib64/ld-linux-x86-64.so.2 /my-new-root/lib64

# Do it again for ls
ldd /bin/ls

# Follow the same process to copy the libraries for ls into our my-new-root.
cp /lib/x86_64-linux-gnu/libselinux.so.1 /lib/x86_64-linux-gnu/libpcre.so.3 /lib/x86_64-linux-gnu/libpthread.so.0 /my-new-root/lib

# Now, finally, run 
chroot /my-new-root bash 
# and run
ls 
```
You should successfully see everything in the directory. Now try pwd to see your working directory. You should see `/`. 

**You can't get out of here! This, before being called containers, was called a jail for this reason. **

> At any time, hit `CTRL+D` or run `exit` to get out of your chrooted environment.

#docker #linux #security