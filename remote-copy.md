# Remote Copy

List some remote copy commands between servers.

## scp 
```
scp -c arcfour /path/to/bigfile.txt username@remote_ip:path/to/dest
```

## tar+pigz+ssh 
If you don't have `pigz` installed. Install it via command below.
```
sudo yum install pigz 
```

Do copy 
```
tar -b 112 -cvf - /path/to/file | pv | pigz -p4 -4 | ssh -T -o Compression=no -x user_name@remote_ip "pigz -d |tar -b 112 -xvf - -C /path/to/dest"
```

`-b` block size of `N x 512` bytes

## nc
```
#!/bin/sh 

# use nc do copy
tar -cf - -C ${local_folder}/ . | pigz -4 | nc -l 4000 & ssh -c arcfour username@remote_ip "nc $FROM_LOCALHOST 4000 | pigz -d | tar xvf - -C /path/to/dest"
```

This command means: create a tar file based on $local_folder, compress it using pigz with level 4, use nc to transfer with port 4000, on remote host , nc listen to 4000 from sender host, pigz uncompress, untar it to destination folder on remote host. It is really an efficient remote copy command. 

+ `-cf` - create tar file 
+ `-C` - change directory 
+ `pigz -4` - compress level 4
+ `pigz -d` - decompress

## rsync
```
rsync -avzh /data/ username@remote_ip:/data/ 
```

+ `-a` enables archive mode, which preserves permissions, ownership, and modification times, among other things
+ `-v` enables verbose mode, which simply increase how much rsync prints to stdout
+ `-z` enables compression during transfer
+ `-h` outputs numbers in human-readable format (e.g. "36864 bytes" becomes "36 kilobytes")

## rsyn over ssh 

```
rsync -avz -e "ssh -o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null" --progress /path/to/bigfile.txt username@remote_ip:/path/to/dest
```

or 
```
sshpass -p "password" rsync -avzh /path/to/bigfile.txt username@remote_ip:/path/to/dest
```
