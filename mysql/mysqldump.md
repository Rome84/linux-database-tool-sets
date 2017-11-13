# MYSQLDUMP

### dump whole database data ignoring some tables 
```
mysqldump --user="username" --password="password" --skip-triggers --compact --no-create-info database_name --ignore-table="database_name.table1" --ignore-table="database_name.table2" > "database_name_data_backup.sql"
```

### dump one table data
```
mysqldump database_name table_name --user="username" --password="password" --skip-triggers --compact --no-create-info > table_name_2017-01-01.sql
```

You can also include `where clause` in the command like below. 

```
mysqldump database_name table_name --user="username" --password="password" --where="data_date = '2017-01-01'" --skip-triggers --compact --no-create-info > table_name_2017-01-01.sql
```

Here is a very good example, we have a partitioned archive table - archive_table_name table. It is partitioned by day. We need to import to another database. This shell script allows you to export each day's data starting from `2017-01-01` in parallel of 10. 

```
#!/bin/sh

#### mysqldump archive  -u root --where="data_date = '2017-01-01'" --skip-triggers --compact --no-create-info > test.sql

# initial extract date
START_DATE=2017-01-01
# month loop
for j in {0..9}
do
(
    # current month
    DATE=$(date -d "$START_DATE $j month" +%Y-%m-%d)
    # frist day of next month
    NEXT_MONTH_FIRST_DATE=$(date -d "$DATE +1 month" +%Y-%m-%d)
    # day loop
    for i in {0..31}
    do
       CURRENT_DATE=$(date -d "$DATE $i day" +%Y-%m-%d)
        if [ "$(date -d "$CURRENT_DATE" +%Y%m%d)" -lt "$(date -d "$NEXT_MONTH_FIRST_DATE" +%Y%m%d)" ]
        then
            echo "Processing: ${CURRENT_DATE}"
            echo File name: archive_table_name_${CURRENT_DATE}.sql
            time mysqldump archive archive_table_name -u root --where="data_date = '"${CURRENT_DATE}"'" --skip-triggers --compact --no-create-info > archive_table_name_${CURRENT_DATE}.sql
        fi
    done
) &
done

wait
echo "All export finish!!"
```

### mysqldump to csv

You can also use mysqldump to generate csv file directly, but  **NOT RECOMMENDED** because `NULL` value will become `\N`

```
mysqldump -u root --skip-triggers --compact --no-create-info --fields-optionally-enclosed-by='' --fields-terminated-by=',' --tab . database_name table_name
```

The command above will generate table_name.txt under directory defined in `--tab` option.  For example, 

```
 mysqldump -u root --skip-triggers --compact --no-create-info --fields-optionally-enclosed-by='' --fields-terminated-by=',' --tab . --where="data_date = '2017-01-01'" archive archive_table_name
```


### mysqldump ddl only
```
mysqldump -d -u <username> -p<password> -h <hostname> <dbname>
```

`-d` option, shortcut of `--no-data`. This command only dumps tables' ddls, no functions/procedures. 


Get DDL for only one table.
```
show create table <database_name>.<table_name>;
```

### mysqldump store procedures/functions/triggers only, no data
mysqldump backup by default all the triggers but NOT the stored procedures/functions. 
+ `--routines` – FALSE by default
+ `--triggers` – TRUE by default

Use command below to dump all stored procedures/functions/triggers of a  database_name

```
mysqldump --user="username" --password="password" --routines --no-create-info --no-data --no-create-db --databases database_name > database_name_procedures.sql
```

If you want to skip triggers use `--skip-triggers` option. 

### dump multiple databases together. 

```
mysqldump --user="username" --password="password" --routines --no-create-info --no-data --no-create-db --databases database1 database2 database3 > procedures.sql
```

# MYSQLDUMP entire mysql instance

## database level

Get all databases list based on this query `get_databases.sql`: 
```
SELECT DISTINCT(schema_name) FROM information_schema.schemata WHERE schema_name NOT IN ('information_schema','mysql', 'performance_schema');
```

You can run this directly to get database lists and save into a file `databases_list.txt `
```
mysql -h hostname -u username -p"password" -A --skip-column-names < get_databases.sql  > databases_list.txt 
```

The script below allows you to export and zip each individual database at the same time.
```
#!/bin/sh

while read line
do 
   echo "Exporting $line ......"
   mysqldump --user="username" --password="password" --routines $line | gzip > $line.sql.gz &
done < databases_list.txt 

wait 

echo "All exports are done"
```

If you have too many databases, do not export all at the same time, you can limit the number to 10 at the same time.  [Here is another script `multiThread.py` I wrote ](https://github.com/bennzhang/python-multithread-multiprocess/) to allow you to do that. Just create the `cmd_list` based on this and replace the `cmd_list` array with the console outputs.
```
#!/bin/sh

while read line
do 
   echo "mysqldump --user="username" --password="password" --routines $line |  gzip > $line.sql.gz"
done < databases_list.txt 
```

## table level 
Same idea applies to table level export. Use this query to create table lists. 

```
SELECT CONCAT(table_schema,'.',table_name) FROM information_schema.tables WHERE table_schema NOT IN ('information_schema','mysql', 'performance_schema');
```

