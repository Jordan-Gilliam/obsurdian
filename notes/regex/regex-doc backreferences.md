# back references in sed

```bash
$ echo 'hey cool cool beans' | sed -r 's/(\S+) \1/REPEATED/'
hey REPEATED beans
```


#bash #regex
