# Automate-MongoDB-Backup

Amazon S3 is an object storage service that offers industry-leading scalability, data availability, security, and performance.
Store and protect any amount of data for a range of use cases, such as data lakes, websites, cloud-native applications, backups, archive, machine learning, and analytics.

![image](https://github.com/user-attachments/assets/990b8e22-2789-4b86-a4fe-9001e460d30e)

Requirements:-AWS Account,MongoDB. AWS Services:-EC2,S3,IAM(Role).

Steps:-
1.	S3:-Create a bucket and inside that bucket create a backup folder.
Login to aws account>Go to management console>search for s3>Bucket>Create bucket.
 ![image](https://github.com/user-attachments/assets/66ed87f5-6171-4909-8a0f-d66f136a0bd5)


Give name of the bucket>Create bucket.
 ![image](https://github.com/user-attachments/assets/46be35ec-208f-40f3-b0f5-2907449f3081)

 
Now click on the bucket which you have created>Create Folder.
 ![image](https://github.com/user-attachments/assets/5f964430-aafb-4148-a9d6-4d69d00ab80b)

Give the name of folder>Create.
 ![image](https://github.com/user-attachments/assets/bee65fb4-fc56-47b0-9ff1-29c166f5bc66)

 
2.	IAM(Role):-Create a role.
Search for IAM>Role>Create role.
![image](https://github.com/user-attachments/assets/1af1d987-814c-43c3-9a02-32fb607a953a)

Choose use case as EC2 and keep rest of settings as it is(default)>Next.
![image](https://github.com/user-attachments/assets/9922ba87-a2c9-477b-a8e2-f9add007f633)

 
Search for S3FullAccess in permission policies select that policy>Next.
 ![image](https://github.com/user-attachments/assets/30a2e945-4357-4e84-9b2c-53d3fec5f584)

Give the name of role>Create role.
![image](https://github.com/user-attachments/assets/7283eda4-eeec-4854-8fe1-9d5bcab27924)

 
3.	EC2:-Launch EC2 Instance.
Search for EC2>Instance>Launch Instance.
![image](https://github.com/user-attachments/assets/ab7ee7f9-6efd-4bc5-b2fe-7bb02587d9a0)

Give the name of instance>Choose AMI as Amazon Linux2
![image](https://github.com/user-attachments/assets/c2303460-187d-45c8-ab31-d2c963156f12)

Select Instance type as t2.micro.
![image](https://github.com/user-attachments/assets/6f41b65e-c932-48b7-8690-d6e654868ab6)

Create new key pair>Give key pair name>select key pair type as.ppk>Create.
![image](https://github.com/user-attachments/assets/4ae694b6-92dd-47a7-9ae3-e02ab943e68b)
![image](https://github.com/user-attachments/assets/d8d8da68-9abe-4cfa-85e9-6f115a3c3e2c)

Advanced details>Select the role which you created>Launch Instance.
![image](https://github.com/user-attachments/assets/769577c6-3f60-479b-932d-56a9ddba32ab)

 
Wait until the status check pass gets 2/2 once status check gets passed then connect it.
![image](https://github.com/user-attachments/assets/184962a5-08ed-42d4-a0d7-abb9bf39a9f0)

EC2 Instance connect>Connect.
![image](https://github.com/user-attachments/assets/513c812b-1d5f-4387-8e72-816183c2a063)

 
4.	Installation of MongoDB and creating a database.
Linux terminal will get open when you connect the instance>write the following commands
sudo su	(Enter to root user)
yum update -y	(Will update the new dependency if available)
![image](https://github.com/user-attachments/assets/23b64a70-a53e-4b86-b076-b688ae0c8cba)

sudo vi /etc/yum.repos.d/mongodb-org-4.4.repo	(Vim editor will get open)
 ![image](https://github.com/user-attachments/assets/6b795b8a-8f84-458d-857d-5b87e25af2fd)

In vim editor press I write the following
[mongodb-org-4.4] name=MongoDB Repository
baseurl=https://repo.mongodb.org/yum/amazon/2/mongodb-org/4.4/x86_64/ gpgcheck=1
enabled=1 gpgkey=https://www.mongodb.org/static/pgp/server-4.4.asc
 ![image](https://github.com/user-attachments/assets/c9d68698-5ac0-42ee-8655-76229bd7058e)

After writing save it by pressing Esc- shift + : - wq
sudo yum install -y mongodb-org	(MongoDB Packages will installed)

 ![image](https://github.com/user-attachments/assets/b0403f18-c99e-4771-bb30-0cfa4faf95dc)

sudo systemctl start mongod	(MongoDB will start)
sudo systemctl status mongod	(It will show that mongodb is active or not)
 

 ![image](https://github.com/user-attachments/assets/18e77e4e-ef8e-4171-a4da-2bf13924ad31)

Now go to Mongo terminal and create a database. mongo	(mongo terminal will open)

![image](https://github.com/user-attachments/assets/15c44569-b510-4841-bda5-b94d946d2966)


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

![image](https://github.com/user-attachments/assets/f7b50d9e-8369-4991-85ab-975affc24887)

![image](https://github.com/user-attachments/assets/dc462b84-9bbe-42a8-be79-5b6c1534436e)


Save the script by pressing Esc – shift + : - wq
chmod 700 backupfile.sh (It will give all permission to file)
./backupfile.sh	(It will execute the file)

 ![image](https://github.com/user-attachments/assets/35570ed2-dd5f-40d6-9ee6-f129d76e244d)

 
Now we have to schedule the file so that the backup file will be automatically backuped in every one minute
Write the following command crontab -e

![image](https://github.com/user-attachments/assets/0ea12dfd-8dc3-4131-b61f-96b367dbd60d)



* * * * * /home/ec2-user/give your backupfile name (i.e .sh file)

![image](https://github.com/user-attachments/assets/134a425c-48da-45cc-9c7f-be4d901c1ff0)


Save it by pressing Esc – shift + : - wq
Again execute the file
./backupfile.sh
 
Go to s3>bucket>folder you will see that the files have been backuped in every 1 min

![Uploading image.png…]()



Hence this can be one of the solution to automate backup of MongoDB database.
