# $PATH

There is a special environment variable called `$PATH`:

```bash
~ $ echo $PATH
/usr/local/bin:/usr/bin:/bin:/sbin:/usr/sbin:/usr/local/games:/usr/games
```

This variable contains a list of places separated by `:` that bash will look when you type a command.

---

If you put an executable file in one of the directories in your `$PATH`, you can make your own commands without needing to specify a relative or absolute path!

`/usr/local/bin` is the customary place to put system-specific scripts that are not managed by your system distribution. If you do:

```bash
~ $ sudo cp yourscript.sh /usr/local/bin
```

---

Then you'll be able to type `yourscript.sh` from any directory on the command-line!

You can rename that command that you type by renaming the file:

```bash
~ $ sudo mv /usr/local/bin/{yourscript.sh,whatever}
```

and now the command is called `whatever`.

#unix #bash