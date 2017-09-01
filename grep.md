# grep,egrep

### Find lines in file1 but not file2
```
grep -v -f file2 file1
```

But it is slow when files are big.  Two other options, you need to `sort` them first. 
```
diff file2 file1 | grep '^>' | sed 's/^>\ //'

diff --new-line-format="" --unchanged-line-format=""  file1 file2
```

### find multiple strings in a file
```
egrep "string1|string2|string3" file
```

### grep recursively
```
grep -r "string" /dir
```

### grep list line number
```
grep -n "string" file
```

### grep list file names only
```
grep -l "string" *
```

### Show with/without filename
```
# without filename
grep -h "string" *

# with filename
grep -H "string" *
```
