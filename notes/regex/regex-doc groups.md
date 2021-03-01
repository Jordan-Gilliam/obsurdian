# groups

```bash
(a|b) # - match a or b
```

* `()` capture group
* `(?:)` non capture group

---
# capture groups in sed

```bash
$ echo 'hey <cool> whatever' | sed -r 's/<([^>]+)>/(\1)/g'
hey (cool) whatever
```


#bash #regex 