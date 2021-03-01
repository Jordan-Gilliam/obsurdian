# quantifiers

* `?` - zero or one time
* `*` - zero or more times
* `+` - one or more times

```bash
$ echo 'dog and doge' | sed 's/doge\?/DOGE/g'
DOGE and DOGE
$ echo 'beep bp beeeeep' | sed 's/be*p/BEEP/g'
BEEP BEEP BEEP
$ echo 'beep bp beeeeep' | sed 's/be\+p/BEEP/g'
BEEP bp BEEP
```


#bash #regex 