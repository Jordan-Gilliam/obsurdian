# metacharacters

* `.` matches any character
* `?` - zero or one time
* `*` - zero or more times
* `+` - one or more times

* `[]` - character class
* `^` - anchor at the beginning
* `$` - anchor to the end

* `(a|b)` - match a or b

* `()` - capture group
* `(?:)` non capture group

* `\d` - digit `[0-9]`
* `\w` - word `[A-Za-z0-9_]`
* `\s` - whitespace `[ \t\r\n\f]`

---
`.` matches any character

```bash
$ echo hello beep boop | sed 's/b..p/XXXX/g'
hello XXXX XXXX
```

---

## when to escape metacharacters

In some engines, you need to escape metacharacters
such as `+` and `?`. In others, you don't.

In javascript and perl, you generally don't need to
escape metacharacters. To use sed and grep in a
similar way, use:

* `sed -r`
* `grep -E`

#bash #regex