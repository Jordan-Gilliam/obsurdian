# sed 

### Word Count Program
1. store our target url in variable
```bash
# store our target url in variable
URL="http://www.gutenberg.org/cache/epub/2701/pg2701.txt"
``` 
2. `curl -s` our target URL
3. pipe to `gunzip` to handle gzip compression
4. pipe to `sed -r 's/\s+/\n/g'` converts all whitespace into newlines
5. pipe to `grep -i` filters the output so that only lines that contain the word "whale" will be shown (`-i` makes the search case-insensitive)
6. pipe to `wc -l` to count occurances
```bash
curl -s $URL | gunzip | sed -r 's/\s+/\n/g' | grep -i whale | wc -l
```

#unix #bash