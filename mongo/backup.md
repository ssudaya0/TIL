## MongoDB Backup Script in Docker

To create a backup of your MongoDB database in docker, you can use the following script:

```bash
#!/bin/bash
# prohibit script at specific time
current_hour=$(TZ=Asia/Seoul date +%H)
echo $current_hour
if [ $current_hour -ge 5 ] && [ $current_hour -lt 8 ]; then
    echo "Script execution prohibited. Current time: $current_hour" >> ~/cron_logs/log.txt
    exit 0
fi

# load docker env
. ~/docker/.env

# set backup directory
out_dir="/home/ubuntu/mongodb/mongodump"
container_name="mongodb_1"
current_time=$(TZ=Asia/Seoul date +"%Y-%m-%dT%H:%M:%SZ")
mongodump_name="mongodump_$current_time.gz"
dump_dir=$out_dir/$mongodump_name

# dump mongodb in gz
echo "dump_dir: $dump_dir"
sudo docker exec -i $container_name mongodump --authenticationDatabase=$AUTH_DB --username=$MONGO_USER --password=$MONGO_PASSWORD --gzip --archive|cat >$dump_dir
echo "done dumping file to dump_dir"

# upload zip file to s3 bucket with setting expire date
sleep 2
s3uri=s3://s3_url/
expire_time=$(date -d "$current_time + 15 days" -u +"%Y-%m-%dT%H:%M:%SZ")

echo "uploading $mongodump_name to $s3uri ..."
aws s3 cp $dump_dir $s3uri --expires $expire_time --storage-class ONEZONE_IA
echo "done upload to s3"

# delete dump data
sleep 2
echo "dump_dir: $dumtp_dir"
sudo rm $dump_dir
echo "done deleting dump file"

exit 0
```

Make sure to replace `out_dir, dump_dir, s3_url` with the actual directory where you want to store the backups. Also, update the MongoDB connection details according to your setup.

Save the script with a `.sh` extension (e.g., `backup.sh`) and make it executable using the following command:

```bash
chmod +x backup.sh
```

You can then run the script using the following command:

```bash
./backup.sh
```

This will create a backup of your MongoDB database in the specified directory with a timestamped filename.
Remember to schedule regular backups to ensure the safety of your data.
