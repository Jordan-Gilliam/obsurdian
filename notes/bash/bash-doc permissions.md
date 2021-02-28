# permissions

Each file on a UNIX system belongs to a user and a group.

users are accounts on the system, like the one you log in with. groups are collections of users.

---

To see what groups you belong to, just type `groups`:

```bash
~ $ groups
substack cdrom floppy audio dip video plugdev
```

---

To inspect the permissions on a file, use `ls -l`:

```bash
~/doc $ ls -l b.txt
-rw-r--r-- 1 substack whatever 14 Dec 28 00:29 b.txt
```

---

Here we can see that the file `b.txt` is owned by the user `substack` and the group `whatever`. There's also this part on the left:

```bash
-rw-r--r--
```

This string describes the permissions of the file.

---

The first character is reserved for some fancy uses, but after that there are 3 groups of 3 characters:

```bash
rwxrwxrwx
```

---

Each character describes a permission: (r)ead, (w)rite, and e(x)ecute. A `-` in place of one of those letters means the permission is not available.

If the e(x)ecute bit is enabled on a file for a user, it means the user can execute the file.

If the e(x)ecute bit is enabled on a directory for a user, it means the user can list the files in that directory.

---

-   The first `rwx` says what the owner can do.
-   The second `rwx` says what users in the group can do.
-   The third `rwx` says what everyone else can do.

These three categories are called user (u), group (g), and other (o).

#unix #bash #linux #config