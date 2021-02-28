# grep

You can search for patterns in files or from stdin with the `grep` command.

The first argument is the pattern to search for as a regular expression.

Regular expressions are a language for pattern matching.

---

Here we can search for all lines matching "whaling" or "fishing":

```bash
~ $ grep -iE '(whal|fish)ing' mobydick.txt | tail -n5
Equatorial fishing-ground, and in the deep darkness that goes before the
the whaling season? See, Flask, only see how pale he looks--pale in the
preliminary cruise, Ahab,--all other whaling waters swept--seemed to
fixed upon her broad beams, called shears, which, in some whaling-ships,
years of continual whaling! forty years of privation, and peril, and
```

#unix #bash