# job control
> Bash is built to handle multiple programs running in parallel

---
## time cat

Type `time cat` and then hit ctrl-c before one
second, as close as possible without going over:

    $ time cat
    ^C

    real    0m0.920s
    user    0m0.004s
    sys 0m0.000s

---
## ctrl-c

Terminate a process in the foreground.

---
## ctrl-z

Put a process in the background.

---
## fg JOB

Move a process from the background to the
foreground by its JOB.

    ~ $ cat
    ^Z
    [1]+  Stopped                 cat
    ~ $ echo wow
    wow
    ~ $ fg %1
    cat
    cool
    cool

---
## job syntax

When you background a process with ctrl-z, the
shell prints a message with `[N]`. `N` is the job
id. Use `%N` to refer to a particular job or:

* `%%` - the most recent job

---
## &

Another way to background a process is to use `&`:

```bash
$ ~ node &
    [1] 29877
```

The job id of `node` is 1 and the process id is
29877. Job ids are local to a shell session, but
process ids are global across the entire system.

---
```bash
~ $ perl &
    [1] 29870

# find the pid for perl
~ $ pgrep perl
    29870
	
# kill the process
~ $ kill %1
    [1]+  Terminated              perl
```

---
## pgrep

Search for a process by its name.

---
## kill ID

Kill a process by its process or job id.


#unix #bash 