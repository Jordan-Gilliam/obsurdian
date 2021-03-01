# slashes
---
## sometimes you need the slashes

```bash
$ echo cat cabbage | sed 's/a/@/g'
c@t c@bb@ge
```

---
## sometimes the slashes are implied

```bash
$ echo -e 'one\ntwo\nthree' | grep ^t
two
three
```

---
## sometimes the slashes are not slashes at all

```bash
$ echo 'xyz party' | sed 's!xyz!cat!'
cat party
```

#bash #regex

