# find

### find all yesterday's file based on creation time
```
find . -daystart -ctime 1
```

### remove all yesterday's file
```
find . -daystart -ctime 1 -exec rm {} \;
```

### find all files after a specfic date based on creation date
```
DATE=2017-01-01
DATE_TIME=$(date -d "$DATE" +%Y%m%d0000)
touch -t $DATE_TIME /tmp/day_$DATE_TIME
# list all files after a specific date 
find . -type f -name "*.txt" -cnewer /tmp/day_$DATE -exec ls {} \;
```

### find all files between two dates
```
DATE1=2017-08-28
DATE_TIME1=$(date -d "$DATE1" +%Y%m%d0000)
DATE2=2017-08-30
DATE_TIME2=$(date -d "$DATE2" +%Y%m%d0000)
touch -t $DATE_TIME1 /tmp/day_$DATE_TIME1
touch -t $DATE_TIME2 /tmp/day_$DATE_TIME2
find . -type f -name "*.txt" -cnewer /tmp/day_$DATE_TIME1 ! -cnewer /tmp/day_$DATE_TIME2 -exec ls -l {} \;
```

### get all errors from all log files
```
  find . -name "*.log" -type f -exec grep -i "error" {} \;
```

### find last 15 minutes files until now
find . -type f -mmin -15
