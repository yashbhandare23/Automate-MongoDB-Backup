# Automate-MongoDB-Backup

Amazon S3 is an object storage service that offers industry-leading scalability, data availability, security, and performance.
Store and protect any amount of data for a range of use cases, such as data lakes, websites, cloud-native applications, backups, archive, machine learning, and analytics.


![image](https://github.com/user-attachments/assets/04626779-f164-4577-80b0-c82abed89d03)


Requirements:-AWS Account,MongoDB. AWS Services:-EC2,S3,IAM(Role).

Steps:-
1.	S3:-Create a bucket and inside that bucket create a backup folder.
Login to aws account>Go to management console>search for s3>Bucket>Create bucket.

![image](https://github.com/user-attachments/assets/53d30298-085e-415c-a0a4-7fbea09acbc2)



Give name of the bucket>Create bucket.

![image](https://github.com/user-attachments/assets/dafc6287-d2c7-4ac8-8a39-f43cff51f813)


 
Now click on the bucket which you have created>Create Folder.

![image](https://github.com/user-attachments/assets/a3b1ebdd-8f17-4fae-be32-1cf35da9d544)


Give the name of folder>Create.

![image](https://github.com/user-attachments/assets/9f55cf6a-3379-40d6-ad2b-7b70f656ee85)


 
2.	IAM(Role):-Create a role.
Search for IAM>Role>Create role.

![image](https://github.com/user-attachments/assets/748d5894-3317-4828-a1f6-9c8db438d67f)


Choose use case as EC2 and keep rest of settings as it is(default)>Next.

![image](https://github.com/user-attachments/assets/cf524845-a534-421e-95a5-b4218aaa88c6)


 
Search for S3FullAccess in permission policies select that policy>Next.

![image](https://github.com/user-attachments/assets/c98ec89d-5749-4550-8d00-f5f221c0a74e)


Give the name of role>Create role.

![image](https://github.com/user-attachments/assets/cdd6c27d-e833-469b-a98e-3c00bad7d492)


 
3.	EC2:-Launch EC2 Instance.
Search for EC2>Instance>Launch Instance.

![image](https://github.com/user-attachments/assets/e87f16f2-c1fc-41bf-9655-f962e711993a)


Give the name of instance>Choose AMI as Amazon Linux2

![image](https://github.com/user-attachments/assets/fb6ca379-0668-4270-baba-0150d7d0ee97)


Select Instance type as t2.micro.
![image](https://github.com/user-attachments/assets/6f41b65e-c932-48b7-8690-d6e654868ab6)

Create new key pair>Give key pair name>select key pair type as.ppk>Create.
![image](https://github.com/user-attachments/assets/4ae694b6-92dd-47a7-9ae3-e02ab943e68b)

![image](https://github.com/user-attachments/assets/5c5844e1-b818-465d-99ea-85aaa5f397e2)


Advanced details>Select the role which you created>Launch Instance.

![image](https://github.com/user-attachments/assets/05723020-639e-47c3-89a4-66ae819d8557)


 
Wait until the status check pass gets 2/2 once status check gets passed then connect it.

![image](https://github.com/user-attachments/assets/9b55a6ec-d785-4ea3-8924-2eecbe84495a)


EC2 Instance connect>Connect.

![image](https://github.com/user-attachments/assets/0eccac8d-ef1b-4201-81e3-edbbc5e782d8)


 
4.	Installation of MongoDB and creating a database.
Linux terminal will get open when you connect the instance>write the following commands
sudo su	(Enter to root user)
yum update -y	(Will update the new dependency if available)

![image](https://github.com/user-attachments/assets/0a63fde6-6875-4afc-a7a6-6c5ba83b0021)


sudo vi /etc/yum.repos.d/mongodb-org-4.4.repo	(Vim editor will get open)
 ![image](https://github.com/user-attachments/assets/6b795b8a-8f84-458d-857d-5b87e25af2fd)

In vim editor press I write the following
[mongodb-org-4.4] name=MongoDB Repository
baseurl=https://repo.mongodb.org/yum/amazon/2/mongodb-org/4.4/x86_64/ gpgcheck=1
enabled=1 gpgkey=https://www.mongodb.org/static/pgp/server-4.4.asc

![image](https://github.com/user-attachments/assets/fb0bc783-6b16-435c-9ed6-342b71c14371)


After writing save it by pressing Esc- shift + : - wq
sudo yum install -y mongodb-org	(MongoDB Packages will installed)

 ![image](https://github.com/user-attachments/assets/b0403f18-c99e-4771-bb30-0cfa4faf95dc)

sudo systemctl start mongod	(MongoDB will start)
sudo systemctl status mongod	(It will show that mongodb is active or not)
 

 ![image](https://github.com/user-attachments/assets/18e77e4e-ef8e-4171-a4da-2bf13924ad31)

Now go to Mongo terminal and create a database. mongo	(mongo terminal will open)


![image](https://github.com/user-attachments/assets/a3c5872b-36cf-41f1-b764-0b8564f506e0)


show dbs	(It will show the database which are present)
use mybackupDB	(Database will be created,you can use other name instead of mybackupDB)
db.createCollection("demoCollection")	(Collection of database will be created)
exit


![image](https://github.com/user-attachments/assets/c898193e-b4fd-4b23-b075-fa46a260172e)

mkdir DBbackups	(Directory will be created,you can replace DBbackups it by other name )
sudo vi backupfile.sh (vim editor will open and .sh file will be created)

![image](https://github.com/user-attachments/assets/f23b678e-e73f-4663-b361-d9daab11c023)


Editor will open press I write the following script #!/bin/bash
# Set the backup folder path
folder_name="/home/ec2-user/your directory name which you created/" # Set the backup file name components
file="backupStorage"
now=$(date +"%b%d-%Y-%H%M%S")
 
ext=".gzip" file_name="$file$now$ext"
backupPath="$folder_name$file_name" echo "Backup file path: $backupPath
# Perform the database backup
mongodump --db give your database name --archive="$backupPath" --gzip # Check if the backup was successful
if [ $? -eq 0 ]; then
echo "MongoDB backup successful."
# Copy the backup to an S3 bucket (replace 'your-s3-bucket-name' with your actual bucket name)
aws s3 cp "$backupPath" s3://your-s3-path # Check if the S3 upload was successful
if [ $? -eq 0 ]; then
echo "Backup uploaded to S3 successfully. # Remove the local backup file
rm "$backupPath"
echo "Local backup file removed." else
echo "Error: Failed to upload the backup to S3."
fi else
echo "Error: MongoDB backup failed."
fi
 
for S3 Path-Go to s3>bucket >folder>copy s3url


![image](https://github.com/user-attachments/assets/a0b379b8-7956-4642-896c-c568219facd1)



![image](https://github.com/user-attachments/assets/0d665cbf-7177-4a52-a2b7-1c9b137ab585)



Save the script by pressing Esc – shift + : - wq
chmod 700 backupfile.sh (It will give all permission to file)
./backupfile.sh	(It will execute the file)

 ![image](https://github.com/user-attachments/assets/35570ed2-dd5f-40d6-9ee6-f129d76e244d)

 
Now we have to schedule the file so that the backup file will be automatically backuped in every one minute
Write the following command crontab -e

![image](https://github.com/user-attachments/assets/0ea12dfd-8dc3-4131-b61f-96b367dbd60d)



* * * * * /home/ec2-user/give your backupfile name (i.e .sh file)


![image](https://github.com/user-attachments/assets/4617628a-4e54-46e8-8f80-7a3a81bbfb68)



Save it by pressing Esc – shift + : - wq
Again execute the file
./backupfile.sh
 
Go to s3>bucket>folder you will see that the files have been backuped in every 1 min

![image](https://github.com/user-attachments/assets/1a2284a8-5319-4234-be03-8ca1f293835d)



Hence this can be one of the solution to automate backup of MongoDB database.
