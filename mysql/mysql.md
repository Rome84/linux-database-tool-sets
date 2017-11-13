# MYSQL COMMAND LINE

### run sql file with command line

```
mysql -u "username" -p"password" -h "hostname" < query.sql
```

### mysql sql to csv
```
mysql -u "username" -p"password" -h "hostname" -B -N < export.sql  > export.csv
```

`-N` is a shortcut option of `--skip-column-names` , this option allows dump to csv **without header**.

`-B` is a shortcut option of `--batch` , print out `tab-delimited` output.

### batch exports in parallel
Assume, there is a partitioned monthly table, key `data_date` is always the end of each month.  We want to extract each partition into an individual csv file. The script belows can do. 

```
#!/bin/sh
TABLE_NAME=$1
SQL=$TABLE_NAME.sql
s=`cat $SQL`
OUT=$TABLE_NAME

# initial extract date
START_DATE=2010-01-01
# year loop
for j in {0..7}
do
(
    DATE=$(date -d "$START_DATE $j year" +%Y-%m-%d)
    # month loop
    for i in {0..11}
    do
       START_MONTH_DATE=$(date -d "$DATE $i month" +%Y-%m-%d)
       NEXT_MONTH_DATE=$(date -d "$DATE $(($i+1)) month" +%Y-%m-%d)
       END_MONTH_DATE=$(date -d "$NEXT_MONTH_DATE -1 day" +%Y-%m-%d)
       MONTH_CONVENTION=$(date -d "$START_MONTH_DATE" +%Y%m)
       #echo "this $START_MONTH_DATE"
       #echo "next $END_MONTH_DATE"
       #echo "month conversion: $MONTH_CONVENTION"
       OUT_PART=$OUT\_$MONTH_CONVENTION.sql
       OUT_CSV_PART=$OUT\_$MONTH_CONVENTION.csv
       echo "$OUT_CSV_PART"
       echo "$s" | sed "s/END_MONTH_DATE/$END_MONTH_DATE/" > $OUT_PART
       mysql -u "username" -p"password" -h "hostname" -B -N < $OUT_PART > $OUT_CSV_PART
    done
) &
done

wait
echo "All exports finished!!"

```

`TABLE_NAME.sql`  is like this

```
select *
from TABLE_NAME 
where data_date = 'END_MONTH_DATE'
```

Replace `*` with real fields list. 

# Import data

### import data from csv
Suppose, we have a partitioned table, each partition has a exported-csv file. We want to import all of them into another mysql database. Create `load.sql` template as below. Hard code the `DATABASE_NAME.TABLE_NAME`. `[INPUT_LOCAL_FILE_NAME]` is the string you need to replace. 

```
LOAD DATA LOCAL INFILE '[INPUT_LOCAL_FILE_NAME]'
INTO TABLE DATABASE_NAME.TABLE_NAME
FIELDS TERMINATED BY '\t'
LINES TERMINATED BY '\n'
(
    field1,
    field2,
    field3,
    field4,
    field5
);
```

The loading script is like below. 

```
#!/bin/bash

host=SERVER_HOST_NAME
username=USER_NAME
password=USER_PASSWORD
table_name=TABLE_NAME

EXPORT_CSV_FILES=`ls $table_name_*.csv`
for f in $EXPORT_CSV_FILES
do
   cat load.sql | sed "s/[INPUT_LOCAL_FILE_NAME]/$f/" > load_$f.sql   
   mysql -h $host -u $username -p$password < load_$f.sql &
done

# wait for all process finished
echo "Importing .........."

wait

echo "All import processes finish.........."
```
