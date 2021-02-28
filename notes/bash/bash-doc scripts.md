# scripts

Whenever you find yourself typing the same sequence of commands several times, consider making a script!

Just put the commands you would normally type into a file and add `#!/bin/bash` to the top of the file:

```bash
#!/bin/bash
mkdir wow
cd wow
echo "yay" \> zing.txt
```
---

Now make your script file executable:

```bash
~ $ chmod +x yourscript.sh
```

And now you can do:

```bash
~ $ ./yourscript.sh
```

to run the commands from your file!

---


# script arguments

When you execute a script with arguments on the command-line, special environment variables `$1`,`$2`, `$3`... will be defined for each argument.

For example, if our script is:

```bash
#!/bin/bash
echo first=$1
echo second=$2
```
---

Then we print out the first and second arguments:

```bash
~ $ ./yourscript.sh beep boop
first=beep
second=boop
```

---

There is a special variable `$*` that contains all the arguments separated by spaces. For a script of:

```bash
#!/bin/bash
echo The arguments are: $\*
```
---

And now we can get at all the arguments in one place:

```bash
~ $ ./args.sh cats dogs ducks lizards
The arguments are: cats dogs ducks lizards
```

#unix #bash