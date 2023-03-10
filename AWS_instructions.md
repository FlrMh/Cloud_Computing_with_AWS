# Creating an AWS EC2 instance(scalable cloud VM) for our local monolith(our app and database VMs):



![](imges/EC2.jpeg)

## Creating our EC2 instance:

!!! Note: Make sure our account always has `Ireland(eu-west-01)` as location.

1. Search EC2.

2. Select EC2 and `launch an instance`.

### Configuring our instance:

3. Name your instance. Best practice: name_group_ReasonForUse

- in my case: `florina-tech201-app`(because we are creating an instance first, for our `app` VM).

4. Select the OS needed. In my case Ubuntu 18.04.

5. Select the instance type. In my case, `t2.micro`.

6. Select a key pair for this instance. In my case, select the key I received to be able to access this instance and that I further stored within the `.ssh` folder on my local host.

!!! Note: If a key matches a pair, the system copies the key and stores it on the other end point, so you do not need to check again.
 

7. Network settings:

1. Allow SSH Traffic - Only my IP.
2. Allow HTTP traffic from the internet(HTTPS - mainly used in production, when the site needs to be secure).

3. Go to Edit( Create a security group so we can use it another time as well): 
- VPC - default
- Subnet - Default subnet recommended.
- Auto-assign public IP - Enable (if it is an instance that should not be accessible- Disable)
- Firewall - Create security group.
- Security group name - FLORINA-TECH201-APP(we create this firewall for our app VM only)
- Security group description - The same as the Security group name + any other important things we need to mention.
- Sec rule 1 - ssh- My IP
- sec rule 2  - HTTP - Anywhere
- sec rule 3 (must be added -  Add Security group rule) - Custom TCP - Anywhere - 3000 - For Node App


8. Configure storage:
- Currently default

9. Summary:
- Need to make sure it matches the configuration we just made. 

10. Launch the instance.

11. Click the ID to get to the dashboard and see the created instance.

## Entering the EC2 instance we have just created



- We created the instance, and the system will run some checks in the background, especially checking if the connection code is `200`. If that is the case, the instance will show on the dashboard as `Running`.
- Now, we have to make sure we did the correct configurations and everything worked as expected. So we need to enter the instance through our local computer.
- So, in order to get in the machine, we need to be able to SSH in the machine.
1. Select your instance, and click `Connect` button at the top.
2. It will take us to a menu with some information and instructions.
3. Let`s go on the SSH Client tab and we can see the information re. our session ID, the key location, and commands to enter the VM through terminal.
4. Lets go to git bash terminal on our local host.
5. Go in the folder where we have the access key to connect to the instance: should be in the `.ssh` folder.
6. Run `chmod 400 YourKeyFile.pem` to ensure your key is not publicly viewable.
7. Then `ssh -i "YourKeyFile.pem" ubuntu@yourInstancePublicDNS.com`
8. Run `sudo apt-get update-y`
9. Run `sudo apt-get upgrade -y`
10. Run `sudo agt-get install nginx -y`
11. Copy and paste your I.P. address from the EC2 Instance Connect tab, into a web browser, and should be able to see the `Welcome to nginx!` web page.

![](images/nginx.PNG)


!! Note: If you get a `Connection timed out` error - it is a port 22 issue. That means, that your computer might have a Dynamic IP address.

![](images/timedout.PNG)

- Go to your Instance`s Security.
- Access your Security Group.
- Edit your security group configurations: On port 22: Switch to `My IP` (if not already configured like that).
- Should now work, and you should be able to `ssh` in the instance through your bash terminal.


## Migrating our `app` folder to our EC2 Instance and running our app on the cloud

- Migrating a folder to your EC2 AWS instance can be done by using the `scp` method.
- In order to do that, first open a `git bash` terminal, and go into the .ssh folder to run the following command.

```
scp -i devops-tech201.pem -r <absolute path of your folder> ubuntu@<your_instance_public_DNS>:/home/ubuntu
```

- It will take a very long time to copy everything, depending on how many files you have within the foder you are migrating.
- However, when it is done, you should simply be aple to run `ls` and see the folder you have just migrated.

![](images/app_folder.PNG)

- Now that our app folder is within out EC2 instance, we can check if nginx and node.js are installed:

```
nginx -v
node -v
```

- Make sure that our Reverse Proxy settings are properly set by accessing the default file :

```
sudo nano /etc/nginx/sites-available/default # check if the `location` configuration mentions the port :3000 for proxy_pass.
```
- Run: 
```
sudo apt-get install npm

node app.js 
```
- Now, you should be able to access your ap via the I.P. address of you EC2 instance, without having to mention the port number!

![](images/reverseproxy.PNG)

--- 

## Tier 2 architecture - Deploying our `database` VM on EC2

- In order to be able to deploy our 2tier architecture, we need to make sure our `database` EC2 instance (our second tier) is set properly on a separate instance than the `app`(our first tier), by following the next requirements. 
- If everything is set properly, we should be able to get the two tiers communicating and we then have successfully deployed a 2tier architechture from a local monolith.

![](images/requirement.png)

### Requirements
1. App tier deployed- available on public IP on port 3000(at least)


2. Now, let`s create 2nd tier with required dependencies:
a. Ubuntu 18.04LTs
b. Mongodb Installed
c. Changed configurtion for Mongo.db to 0.0.0.0
d. Security group for our DB - allow 27017 from anywhere - allow only from app instance.
e. Go back to the app, create the environment variable that allows the connection of the app to the database.
f . Relaunch the app.
g. Securing the database VE through a Firewall(security Group) : database can only be accessed through the app, the app can be accessed only through a secure shell(SSH).

- After we created the instance with the required OS, we can ssh within the `database` instance and continue with configuring the other dependencies.
```
 ssh -i key_file.pem ubuntu@<your instance public DNS>

```
- Once inside the EC2 instance, we need to get the provision file for the `database` VM we created locally, so we can provision our EC2 database instance with the same dependencies as our locally created VM.
- We can do that either by using the `scp` commad we used previously to migrate the `app` folder within our `app` EC2 instance, or we can copy the file in our `database` EC2 instance by cloning the repository that contains the specific file and run it within our EC2 instance. 
- If we want to use the `clone` method, simply run 
```
git clone <http of the repository where we have the database provision file>

# navigate within the repo to the folder that holds the provision file for database

# then change the permission for the provision file so it is executable

sudo chmod +x <provisionfile.sh>

# run the file so you can implement the dependancies

sudo ./provisionfile.sh

# check that mongodb is up and running by checking its status

sudo systemctl status mongod
```
- If everything went well, you should be getting `active` as a status.

![](images/activemongo.PNG)

!!! Note: You might encounter issues if your provision.sh file containt specific changes for the mongodb configuration. 

- For example, my provision file initially specified some changed to the network configurations for mongo.db. Those specific configuration changes need to be commented out in the provision.sh file, as they will create issues when running the provision file within our EC2 instance.
- So, before running the provision file, make sure you run:
```
sudo nano <provisionfile.sh> # in order to comment out any specific configuration changes and have only the basic installation and enablement for mongo.db
```
- As a result, your provision file, before running it, should look like this:

```
sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv D68FA50FEA312927

echo "deb https://repo.mongodb.org/apt/ubuntu xenial/mongodb-org/3.2 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-3.2.list

sudo apt-get update -y
sudo apt-get upgrade -y

sudo apt-get install -y mongodb-org=3.2.20 mongodb-org-server=3.2.20 mongodb-org-shell=3.2.20 mongodb-org-mongos=3.2.20 mongodb-org-tools=3.2.20

sudo systemctl restart mongod
sudo systemctl enable mongod

```
- If your provision file contains the previous commands, your mongodb shoud be up and running as expected when checked with:
```
sudo systemctl status mongod
```

- Lastly, we will have to change the configurations for mongo.db to allow the database to be accessed from anywhere, as long as it is accessed through the `27017` port. This would be the same as we did in ptovision file, only we will do it manually to make sure everything works as required. 

```
sudo nano /etc/mongod.conf
# go to the network interface section and replace the current bindIP with 0.0.0.0

```
- After we change the bidIP, we have to make sure we restart and re-enable mongo.db:

```
sudo systemctl restart mongod
sudo systemctl enable mongod
```
- Now, our database should be ready and accesible through port 27017.

---


## Tier 2 architecture - Connecting our `app` and `database` EC2 instances

- Now, that we have our `app` Ec2 instance set up and running properly, and the `database` EC2 instance with mongo.db active and allowing connections through port 27017, we only have to create an env variable wthin the `app` Ec2 instance that will connect us to the database.
- Just like we previously did locally, let`d first make sure the app is running properly.
```
ssh -i <key_file.pem> ubuntu@<your instance public DNS>

cd app/

npm install

node app.js

#these will allow us to check that first, our app is working as expected

```

- Now that our app is up and running, we need to create an env variable (at first we will not make it persistent, just because we want to make sure the connection is successful before we make the variable persistent).

```
export DB_HOST=mongodb://<database ip address>:27017/posts

printenv DB_HOST # to check the env variable has been set correctly

cd app/

npm install

node seeds/seed.js

node app.js
```
- If everything was set up correctly, you should be able to see the posts page.

![](images/workingposts.PNG)

---

## Creating an AM for our `app` EC2 instance

- Creating AMIs for our  Ec2 instances can help save the company money (in terms of storage) due to the fact that instead of having an EC2 instance that could be running while not used, we can create an AMI (an image of the state of the EC2 instance) that can allow us to launch an instance with the same template and at the same stage as our `app` instance, whenever we need to use it again.

- In order to create an AMI, we need to select the `Action` tab at the top of the AWS page of our instance that we want to crate an AMI for:

![](images/createAMI.PNG)

- Select `Image and templates` and further select `Create image`.
- This will takes us to a page similar to the page where we created an EC2 instance, where we have to mention the name of our AMI and the description.
- Best practice would be to name your AMI after your EC2 instance that you create an AMI for + AMI at the end.
- When it comes to the description, copy + paste the name you assigned it, and mention things like ports allowed, just to give information to the user about what the AMI is a template of. 
- When ready, simply press `Create image`. It will be in the `Pending` status for a while as it has to cpy all the information from the EC2 instance. 
- Once finished, feel free to delete the EC2 instance if no longer needed or in use. 
- To access all your created AMIs, simply navigate the left-side menu.

![](images/acessAMI.PNG)

---

## Creating an AMI for our `database` EC2 instance

- Exactly as we did for the `app` EC2 instance, we will now create an AMI for our `database` EC2 instance.
- So, once again, select `Image and templates` and further select `Create image`.
- Name the AMI accordingly e.g. "name-class-ami" and in the description copy and paste the name and add details that would be useful (e.g. litening on port 27017-in the case of the database).
- Select `Create image`. Once the status is `Avaialble`, feel free to terminate your `database` EC2 instance. 
- Together with the AMI from the `app` EC2 instance, if you now go to the Images tab in the left-side menu, you should be able to see your AMIs.

![](images/namingami.PNG)

---

## Launching instances from AMI (both for `app` and `database`)

![](images/launchingAMIinstance.PNG)


- Now that we have Images f our EC2 instances, and we have terminated both the `app` and `database` EC2 instances, we can recreate the instances by using the AMIs we just created.
- Go in the Images tab on the menu on the left hand side of the AWS web page.
- Selected the `app` AMI and click on the AMI ID.
- This will open a new web page with the details of the AMI. 
- Select `Launch instance from AMI`.

![](images/amiinstance.PNG)

- Just like when we created the original EC2 instance for our `app` virtual environment, selecting `Launch instance from AMI` will prompt us on a new web page where we have to select the key, the security group, etc.
- We will have to select a name, the OS will be already selected de to the fact that this EC2 instance is being created after a template.
- We will have to select a key, which will be the same key we used for the original EC2 instance.

![](images/AMIsettings.PNG)

- And, in terms of Network settings, we can simply select `Select existing Security group` and pick the security group we created for our `app` EC2 instance.
- Then, we can just `Launch Instance` and we will have a new EC2 instance created from an AMI.

### !!! Note: For our `database` AMI we should follow the same instructions in order to launch an EC2 instance after the `database` AMI.

---

## Launching and connecting our EC2 instances created from AMIs

- Now that we have created new EC2 instances from AMIs from our `app` and `database` initial EC2 instances, we only have to `ssh` into both instances, check that everything is set and running accordingly.

1. In our `database` EC2 instance:
- Check that our network configurations are correct and allow connection from port 27017:
```
sudo nano /etc/mongod.conf

#should say bandIP: 0.0.0.0
```
- Check that mongodb is up and running correctly :
```
sudo systemctl status mongod
```
![](images/activemongo.PNG)

2. In our `app` EC2 instance:
- Check that we have `nginx` and `node` installed and with the correct versions:
```
nginx -v

node -v

```

- Check that our reverse proxy is correctly set with the right cnfiguration:

```
sudo nano /etc/nginx/sites-available/default
```
- Check that we don t have an env variable and set it up:
```
export DB_HOST=mongodb://<database EC2 instance IP>:27017/posts
```
- Install and launch the app to make sure it is running and connecting to the database:
```
cd app/

npm install

node seeds/seed.js

node app.js
```

- If everything went right, you should be able to see the app running and connecting to the database:

![](images/AMIposts.png)

---

## Setting a monitoring and alerting alarm for our `app` EC2 instance.

- Setting up an Alarm for our system can be very beneficial in monitoring our app 24/7 and ensuring that everything is up an drunning as expected. We can set certain actions to be followed when the Alarm goes off, but for practice purposes, we will simply set an alarm to send us an email when the CPU Utilisation in our EC2  instance has reached a certain treshhold. 

![](images/monitoringAWS.PNG)

---

- In order to set up an alarm for our `app` EC2 instance, in my case that monitors the CPU-Utilisation, we simply need to follow some steps. 
First, we need to go into the dashboard of our instance, select `Monitoring`, and further Select `Manage detailed monitoring` where you will have to tick the `Enable` box. 

![](images/enablemonitoring.PNG)

[Creating a CPU alarm for an EC2 instance](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/US_AlarmAtThresholdEC2.html)

- First, we need to access the Amazon CloudWatch dashboard. 
- Go to `Alarms` tab on the left hand side of the dashboard.
- Further select the option `All alarms`.
- Select `Create Alarm` and proceed to follow the above instructions step by step.

!! Note: when prompted to select an SNS topic for alarm, please make sure to first create a new topic containing your Email as the communication endpoint in order to make sure that you receive the Alarm went it goes off. 
- When finished, please make sure to head back to the AWS CoudWatch Dashboard in order to check that everything went well in terms of set up.
- If everything went well, you will also receive an email from AWS mentioning your subscritption for the monitoring service as a confirmation for your alarm creation.
---

## Setting up an AWS S3 Bucket

- Setting up an AWS S3 bucket(folder) can be extremely useful for disaster recovery, social media, allowing public access to documents via cloud, etc.

![](images/s3diagram.PNG)

- As you can see on the diagram, in order to create and make use of a S3 bucket, we need to first create an EC2 instance. Please see the previous sections for intructions.

!! Note: The diagram mentions that we need "AWS Secret& Access keys" to be able to access the S3 instance. Please make sure you have these from your AWS admin before proceeding.

- Once our Ec2 instance is set, we need to SSH into it, as normally.
- The next step will be to make sure we have access and connection to internet by running
```
sudo apt-get update -y
sudo apt-get upgrade -y
```
- Once we established that we have an internet connection, the next step (see diagram) will be to install Python and its dependancies, as S3 is built on Python, so we need it to interact with this service.

```
# Install Python
sudo apt install python -y

# check Python version (will output python 2.7 = default used version. MUST be at least Python 3.6)
python --version


# changing to Python 3 (in this way we are telling the system to use Python 3)
alias python=python3


# checking the version again 
python --version
# will output python 3.6.9
# remember that the aliases disappear when the session ends.

# Install pip
sudo apt install python3-pip

# Install AWS CLI
sudo pip3 install awscli

aws configure
# this will prompt us to insert the AWS Access & Secure keys. 
# Further it will ask you to introduce the default region name (where the requests to the bucket will be sent from - S3 buckets are available globally, but we need to specify a location from where we will interract with it = default region name of your EC2 instance)

# Lastly, it will ask for the type of output you would like (in my case, JSON).

# If everything went well, you should be able to receive the buckets in a server when running the following command:
aws s3 ls
# if you received "access denied" it means that you have a typing error in the configuration process.

```
- We should now have a secure connection between our EC2 instance to the S3 service!

- We should now be able to perform minimum CRUD(create, read, update and delete) operations.

- Next step is to create our own bucket!
```
aws s3 mb s3://name-respecting-conventions
# mb = make bucket; we cannot use "_"; use explicit name for your bucket.

aws s3 ls # to confirm your bucket has been created


```
- You should be able to see your bucket on the AWS website, on the S3 dashboard.

![](images/flrs3.PNG)

- If we want to upload something in our bucket, we can do it via the dashboard, using the "Upload" button. 
- However, we can also do it through cml.
- In order to store some data within our bucket, let`s create a file.
```
sudo touch test.txt

ls # check the file has been created

sudo nano test.txt # add whatever text you would like to have in the file, then save it using Ctrl + X, Y and Enter

cat test.txt # to see the content of your file and make sure your changes have been changed

```
- In order to be able to upload it into our personal bucket, we must do the following:
```
aws s3 cp test.txt s3://name-of-your-bucket
```
- The file should now be successfully added to your bucket and should be available to view online on the dashboard. 

![](images/s3file.PNG)

- There is our file created!
- If we want to allow public access to our file through an URL, we only have to change the permissions of the file. 

!!! Note: Permission of the files within a bucket are dependable on the permissions of the bucket itself. However, we have the option to specific permissions for each file.
- Permissions for a file can be accessed by clicking on the file name, choosing 'Permissions', then 'Edit'.
- Within the 'Edit access control list' tick all the unticked Boxes (Read access).
- Thick the box that request your confirmation for making the changes, then simply press "Save changes".
- Now, if we click on the URL of the file, we should be able to see our file and the text within it.

![](images/testurl.PNG)

Happy days! We have created a file that can be accessed through an URL using S3 service on AWS.

- If we want to delete a file from out bucket, we have to use:
```
aws s3 rm s3://bucket-name/bucket file

# if we want to delete all the files in a bucket

aws s3 rm s3://fucket-name --recursive
```

!! Note: As S3 offers backup, we can download our file from the S3 bucket using:
```
aws s3 cp s3://bucket-name/file-to-copy name-of-the-copied-file-in-our-local-machine
```
- Now, if the bucket is empty and we want to remove our bucket from the S3 service, we can do it by using:
```
aws s3 rb s3://bucket-name
```
!!! Please, be advised that the bucket will not be removed unless all the files inside the bucket have been deleted. 

- If we want to automate the process of creating a bucket, adding a file to the bucket, deleting the file and deleting the bucket, we can do that by using `boto3`.
- Boto3 is an AWS SDK(Software development kit) for Python that allows us to create, configure and manage AWS services such as EC2 and S3.
- To be able to install and use Boto3, we have to make sure we have Python installed in our EC2, as well as pip.

```
#Installing boto3

sudo pip3 install boto3
```
- In my case, I have to create a Pyhton file that contain the instruction for fullfilling the following tasks: create a bucket, create a file that will be a reproduction of a file that exists within my local host, upload the file to the bucket, delete the filde, and lastly delete the bucket.
- Remember to `import boto3` at the beginning of your python file!!

---

## Creating an Autoscaling group and Load balancing

![](images/autoandload.png)

1. Create launch template for autoscaling group
- On the left side of the Ec2 dashboard, we have the Instances tab where we can find `Launch template`. 
- Select Launch template.
- Because we do not have a template created, we have to create one.
- Select `Create Launch template`.
- Name if using the same name convention : e.g. `florina-tech201-ASG-1`.
- Copy and paste the name in the description.
- Tick the box for `Autoscaling guidance`.
- Select `Ubuntu 18.04` for O.S. as we want all the instances in the Autoscaling group to have the same configuration.
- For Instance type select `t2.micro`, as this is what we used before and it works for us.
- Select the same key as before for the access, e.g. in my case `devops-tech201`.
- For Security group select the previously made security froup for your App instance.
- User data to automate provisioning to be used. We would like to make sure that all the machines have the same provisioning, so when the autoscaling launches it, it needs to facilitate it with the same functionality for all the users that use the app. So, we will write a script that does all the provisioning for us when the instance is launched. 
- On Advanced details, scroll down to the bottom, and stop when it says `user data`.
- Here is where we will do the bash scripting for the provisioning of our instance.

```
#!/bin/bash

sudo apt update -y
sudo apt upgrade -y

sudo apt install nginx -y
sudo systemctl restart nginx
sudo systemctl enable nginx
``` 

- **BE AWARE: if you have any typos, the user data provision will not work and everything will need to be done manually!!!**
- If the summary looks correct, select `Create launch template`.

Now, we have a launch template, but we do not have a load balancer, an autoscaling group, or a policy. So, so far we have done 25% of the work. 

- Now that we have a launch tamplate, we would like to use it to create an autoscaling group (find it at the bottom of the EC2 danshboard under `Auto Scaling`).
- Let`s create an Auto scaling group. 
- Select `Create an auto scaling group`.
- Same naming convention `florina-tech201-ASG-app`.
- For launch template, select the launch template we have just created earlier.
- Select `Next`.
- VPC will be default.
- As we want to create an instance in eu-west-1a, one in eu-west-1b, one in eu-west-1c. We will do that by selecting on availability zones `DevOpsStudent default 1a`, `DevOpsStudent default 1b`, respectively `DevOpsStudent default 1c`.
- Select `Next`.
- We need to create the Load balancer as mentioned in the diagram.
- We want an Application Load Balancer, by selecting `Attach a new load balancer` and select `Application load balancer`.
- The name will be automatically generated, e.g. `florina-tech201-ASG-app-1`.
- We need the load-balancer to be `Internet-facing` as it needs to facilitate internet users that access the app.
- We need to tell the load balancer to allow port 80 connection as our instances need that, and although our template does that, the load balancer needs to be instructed as well.
- So, on `Listeners and routing`, select for port 80- `Create a target group`.
- A name for the target group will be assigned automatically that will allow access via port 80 due to our template only having NGINX as provisioning.
- `Health checks` = IMPORTANT! - automatically replaces the instances that have failed. So, tick the box for `ELB`.
- Select `Next`.
- Policy setup! Why and when it should scale up! 
- Group size: Desired - 2, Minimum - 2, Maximum - 3.
- Scaling policies- Select `Target tracking Policy`.
- In the default, it will monitor the CPU Utilization, when it reaches 50 it will create a new instance as part of autoscaling.
- Select `Next`.
- Add notification - select `Next`.
- Add tags - Ideally ADD TAGS as the autoscaling group in connection to the load balancer will create 2 instances that need to be names so we can differentiate them from other instances. 
- Respect naming convention , for `key` type "Name", for `value` type `florina-tech201-HA-HS`.
- Lastly, `Create auto scaling group`. 
- Now, once the instances have been created, due to the provisioning file that installs nginx, if we simply copy and paste the IP addresses of the instances in our browser, we should be able to get the webpage `Welcome to nginx`. 

![](images/nginx.PNG)

!!! **NOTE**: If we want to automate the entire process, and have the app up and running every single time we launch an EC2 instance within our autoscaling group, we have to make sure the `launch template` is made using the AMI we created after our initial EC2 instance that was set with a reverse proxy and where the app was up and running.

We also have to make sure that the `user data` section in the configuration of our launch template runs a command that installs and launches our app, but runs it in the background, rather than in the foreground.

In this case, our `user data` section will look like this:

```
#!/bin/bash

sudo apt update -y 
sudo apt upgrade -y

cd /path/to/our/app/folder

nohup npm start 2>/dev/null 1>/dev/null&

```
Our app should be accessible only by copy and pasting the IP address of our instance in the browser, and without having to ssh within our EC2 instance!

![](images/reverseproxy.PNG)







