# sed 

### Remove
```
# Deleting lines
sed '5 d' file.txt

# Deleting lines with a matched String
sed -i '/string/d' filename

# Remove the last character of a file
sed -i '$ s/.$//' filename
```

### Replace
```
# In-place replace a string with another in a file 
sed -i 's/unix/linux/g' file.txt

# Replacing string
sed 's/unix/linux/' file.txt

# Replacing the nth occurrence of a pattern in a line.
sed 's/unix/linux/3' file.txt

# Replacing all the occurrence of the pattern in a line.
sed 's/unix/linux/g' file.txt

# Replacing from nth occurrence to all occurrences in a line.
sed 's/unix/linux/3g' file.txt

# Running multiple sed commands.
sed 's/unix/linux/' file.txt| sed 's/os/system/'  
or 
sed -e 's/unix/linux/' -e 's/os/system/' file.txt

# Replace specific line number
sed '3 s/unix/linux/' file.txt      # replace the line 3
sed '2,$ s/unix/linux/' file.txt    # replace line 2 and the last line

# Replace all file names:
for f in *; do mv "$f" "${f/7548688/7548689}"; done

```

#### Replace a string with a file content 
It is suitable for having a template file and want to replace a string `original_string` with the whole content of a file. 

```
 sed "/original_string/{          
          r filename
          d
         }" template_file > new_file

```

`r` reads the file contect from file `filename`; `d` is deleting the current line.


### Get logs between two timestamp
sed -n '/2015-07-16 12:59/,/2015-07-16 13:00/p' server.log

