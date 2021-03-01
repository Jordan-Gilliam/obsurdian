# character class

`[...]`

Any characters inside the square brackets will
match.

For example, to match a vowel character: `[aeiou]`.

```bash
$ echo 'beep and boop' \
| sed 's/b[aeiou]\+p/BXXP/g'
BXXP and BXXP
```

---
## character class ranges

`[A-Z]`

You can use `-` to specify ranges.

```bash
$ echo 'beep and boop' | sed 's/[a-f]/X/g'
XXXp XnX Xoop
```

---
## negated character class

`[^...]`

Put a `^` after the opening square bracket in a
character class to negate it.

For example, to match a non-vowel character: `[^aeiou]`

```bash
$ echo 'beep boop' | sed 's/[^aeiou]/Z/g'
ZeeZZZooZ
```

---
## character class sequences

Regex engines provide many pre-defined character class sequences:

* `\w` - word character: `[A-Za-z0-9_]`
* `\W` - non-word character: `[^A-Za-z0-9_]`
* `\s` - whitespace: `[ \t\r\n\f]`
* `\S` - non-whitespace: `[^ \t\r\n\f]`
* `\d` - digit: `[0-9]`
* `\D` - non-digit: `[^0-9]`


#bash #regex