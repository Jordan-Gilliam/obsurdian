# loops

 ### while loop
```bash
# logs date every second
while true; do date; sleep 1; done
```


### for loop
```bash
# prints 1-10
for x in {0..10}; do echo $x; done
```

#### example
`read` each line of `std in` into an environment variable called `$LINE`. Pipe `wc -l` output into awk to print custom format `awk '{print $1}'` (prints first column and removes filename after wc)

```bash
ls -1 | while read LINE; do echo -n "$LINE "; wc -l $LINE  | awk '{print $1}'; done
```