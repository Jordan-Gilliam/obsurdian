# chmod

To change the permissions on a file, first figure out which capabilities you want to grant or revoke (rwx) from which categories of users (ugo).

---

To allow the owner of a file to execute a script you can do:

```bash
~ $ chmod u+x script.sh
```

which is the same as:

```bash
~ $ chmod +x script.sh
```

because the `u` is implied if not specified.

---

You can also revoke permissions with a `-`. To make it so that other users can't write to a file:

```bash
~ $ chmod o-w wow.txt
```

---

You can grant and revoke permissions at the same time. Here we're adding read and execute permissions to the user while simultaneously revoking read and write from the group:

```bash
~ $ chmod u+rxg-rw status.sh
```

You can change the owner of a file with `chown` and the group with `chgrp`.

#unix #bash #linux #config