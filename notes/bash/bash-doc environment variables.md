# environment variables

Environment variables are defined by the shell and shell scripts.

To list the current environment variables, type `export`:

---

```bash
~ $ export
declare -x DISPLAY=":0"
declare -x HOME="/home/substack"
declare -x HUSHLOGIN="FALSE"
declare -x LANG="en_US.UTF-8"
declare -x LD_LIBRARY_PATH="/home/substack/prefix/lib:/usr/local/lib:/usr/lib/x86_64-linux-gnu:/usr/lib:/lib64:/lib"
declare -x LIBGL_DRIVERS_PATH="/usr/lib/i386-linux-gnu/dri:/usr/lib/x86_64-linux-gnu/dri"
declare -x LOGNAME="substack"
declare -x MAIL="/var/mail/substack"
declare -x OLDPWD="/home/substack/projects/workshops"
declare -x PATH="/home/substack/prefix/bin:/usr/local/bin:/usr/bin:/bin:/sbin:/usr/sbin:/usr/local/games:/usr/games"
declare -x PREFIX="/home/substack/prefix"
declare -x PWD="/home/substack"
declare -x ROXTERM_ID="0x43962f0"
declare -x ROXTERM_NUM="15"
declare -x ROXTERM_PID="2521"
declare -x SHELL="/bin/bash"
declare -x SHLVL="3"
declare -x TERM="xterm"
declare -x USER="substack"
declare -x WINDOWID="8684328"
declare -x WINDOWPATH="7"
declare -x XAUTHORITY="/home/substack/.Xauthority"
```

---

You can use any environment variable by refering to its `$NAME`.

For example to print the value of `$HOME` do:

```bash
~ $ echo $HOME
/home/substack
```

---

You can use environment variables in any command:

```bash
~ $ ls /home/$USER
doc  media  notes.txt  projects
```

---

To define your own environment variable, just put its name followed by an equal sign (with no spaces) followed by its value:

```bash
~ $ ANIMAL=cats
~ $ echo $ANIMAL
cats
```

---

Environment variables are almost always capitalized to distinguish them from variables in shell scripts but lower-case variables work too.

#unix #bash #config