# backticks
> allow the output of a program in the arguments list of another

## logging example
```bash
echo some log data > foo-`date +%F`.log
```

## arithmetic
```bash
echo Greetings from the year $((`date +%Y`+1000))

# alternative to backticks is xargs
date +%Y | xargs echo Greetings from the year
```

#unix #bash