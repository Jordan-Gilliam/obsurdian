# fold
> reformat with fold

```bash
head -n250 moby-dick.txt fold | fold -w 30
```

Sometimes it's handy to break long lines into shorter lines.

---

We can use the fold command to break some text at 30 characters:

```bash
~ $ head -n250 mobydick.txt | tail -n3 | fold -w 30
can see a whale, for the first
 discoverer has a ducat for hi
s pains....
I was told of a whale taken ne
ar Shetland, that had above a 
barrel of
herrings in his belly.... One 
of our harpooneers told me tha
t he caught
```

---

or to break on spaces instead, use `-s`:

```bash
~ $ head -n250 mobydick.txt | tail -n3 | fold -sw 30
can see a whale, for the 
first discoverer has a ducat 
for his pains....
I was told of a whale taken 
near Shetland, that had above 
a barrel of
herrings in his belly.... One 
of our harpooneers told me 
that he caught
```

#unix #bash