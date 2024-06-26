### Story of Organic India Corp ( some random organization )

* This organization runs an e-commerce application for selling organic fmcg
* This organization has it's Datacenter in Hyderabad

![alt text](shots/1.PNG)

#### Problems

* From the 1st to 5th of every month, orders increase. Right now Organic India Corp has sixteen VMs where they are running backend services. Inspite of this, one of every five customers are facing latency issues and slow responses
* Right now this organization is running only in one Datacenter which is a single point of failure, so they have one more Datacenter with same number of servers in noida
* So, the architect of Organic India Corp has suggested to move to AWS ( cloud )

#### How can AWS solve these problems ?

* Let's understand basic merits of the AWS cloud :
    * _**Global presence**_
    * _**Elasticity**_ : Automatically increasing and decreasing the resources based on some dynamic metrics
    * _**Disaster Recovery**_ : It is easier to setup
    * _**Pay as you go**_

### Service Models : IaaS  v/s PaaS  v/s SaaS 

![alt text](shots/2.PNG)

#### How to move to AWS ?

* _**Migration**_ : i.e. move the applications / data on your physical servers into AWS ( cloud )
* Migration based on workloads is classified into three types :
 1. Server Migration
 2. Database Migration
 3. Storage Migration

### Organic India Corp Application Architecture :

![alt text](shots/3.PNG)

### Order of Migration

* Always from least dependent to most dependent
* In this case :
    * Storage Layer and DB Layer ( Least Dependent )
    * Application Layer
    * Presentation Layer ( Most Dependent )

### Migration Strategies => _**7R’s of Migration**_

 1.  _**RE-HOST**_ : Lift and shift ( running on On-premises = running in the Cloud )

 ![alt text](shots/4.PNG)

 2. _**RE-PLATFORM**_ : Lift tinker and shift ( lifting with approximately required changes )

 ![alt text](shots/5.PNG)

 3. _**RE-FACTOR / RE-ARCHITECT**_ : Re-write the complete application in a modern cloud native way ( example : shifting to microservices )

 4. _**RETAIN**_ ( postponing migration )

 5. _**RRETIRE**_ ( stopping migration )

 6. _**RE-PURCHASE**_ ( for using of internal tools )

 7. _**RE-LOCATE**_ ( having similar platform of work, both in On-premises and Cloud )

 ![alt text](shots/6.PNG)

### HYPERVISOR Types

* Hypervisors are of two types :

 1. Type - I hypervisor

    ![alt text](shots/7.PNG)

 2. Type - II hypervisor

    ![alt text](shots/8.PNG)

* Virtualization

  ![alt text](shots/9.PNG)

#### What does migration mean in the Context of Server ?

* Contents of the disk are replicated to AWS which will eventually create _**Amazon Machine Images (AMI)**_

![alt text](shots/10.PNG)

* We will be migrating unique application hosted servers to AWS

### Migration approaches

* We have two approaches :
 1. One-time migration / replication
 2. On-going migration / replication 

### What is supported ways by AWS migration ?

* AWS Partner Network
* Agent V/S Agentless

![alt text](shots/15.PNG)

### Assumptions

1. We are not seeking help from partner network
2. We are not using any third party applications, we are using AWS tools

### Migration Steps

* In AWS we are going to perform :
    * P2V ( Physical to Virtual)
    * V2V ( Virtual to Virtual )

### Simulating On-premises

* Since we don't have physical infra, we will be simulating this on cloud ( AWS / Azure )

![alt text](shots/11.PNG)

### Migration Steps

* Workflow 

    [ Refer Here : https://docs.aws.amazon.com/mgn/latest/ug/migration-workflow-gs.html ]

* Limits 

    [ Refer Here : https://docs.aws.amazon.com/mgn/latest/ug/MGN-service-limits.html ]

* Supported OS 

    [ Refer Here : https://docs.aws.amazon.com/mgn/latest/ug/Supported-Operating-Systems.html ]

* Steps :

![alt text](shots/12.PNG)

### Source Environment Prepartion

* For this activity we will be installing nopCommerce
* Overview of architecture :

![alt text](shots/13.PNG)

* Create two AWS ubuntu linux instances _**( nop, dbserver )**_ of size `t2.micro` or `t3.micro` and security group opened for both `3306` and `5000` ports _**( open all - all tcp, all traffic )**_

* Consider one VM to be database server and install mysql
    * Steps: 
    
    [ Refer Here : https://www.digitalocean.com/community/tutorials/how-to-install-mysql-on-ubuntu-22-04 ]

    ```
    sudo apt update
    sudo apt install mysql-server -y
    sudo systemctl start mysql.service
    ```
    * In the mysql shell try to execute following commands 
    `username : nop , password : nop12345` :
    ```
    sudo mysql
    CREATE USER 'nop'@'localhost' IDENTIFIED BY 'nop12345';
    GRANT ALL PRIVILEGES ON *.* TO 'nop'@'localhost';
    FLUSH PRIVILEGES;
    exit
    ```
    ![alt text](shots/32.PNG)

    * To verify the `nop` execute `mysql -u nop -p` and you should be allowed in sql shell
    ```
    show databases;
    Select user();
    ```
    ![alt text](shots/33.PNG)
    ![alt text](shots/34.PNG)
    ![alt text](shots/16.PNG)

    * _**NOTE**_ : we need to fix the issue with external connectivity to the database

* _**Application : nopCommerce**_ 

    [ Refer Here : https://docs.nopcommerce.com/en/installation-and-upgrading/installing-nopcommerce/installing-on-linux.html ]

* _**Steps**_ :

    * Install .net core 7 
    
    [ Refer Here : https://learn.microsoft.com/en-us/dotnet/core/install/linux-ubuntu-install?pivots=os-linux-ubuntu-2004&tabs=dotnet8 ]

    ![alt text](shots/35.PNG)

    * Install `Unzip` and download the nopCommerce zip file
    ```
     sudo apt install unzip -y 
    ```
    ![alt text](shots/36.PNG)

    [ Refer here : https://docs.nopcommerce.com/en/installation-and-upgrading/installing-nopcommerce/installing-on-linux.html ]

    ```
    sudo mkdir /usr/share/nopCommerce
    cd /usr/share/nopCommerce/
    sudo wget https://github.com/nopSolutions/nopCommerce/releases/download/release-4.60.3/nopCommerce_4.60.3_NoSource_linux_x64.zip
    sudo unzip nopCommerce_4.60.3_NoSource_linux_x64.zip
    sudo mkdir bin
    sudo mkdir logs
    ```
    * To check whether application works type the command and expose the `http://<public_ip>:5000` over the browser
    ```
    sudo /usr/bin/dotnet Nop.Web.dll --urls "http://0.0.0.0:5000"
    ```
    ![alt text](shots/37.PNG)

* Create a user called as `nop`
```
sudo uadduser nop
password :nop12345
```
* Give full permissions to `/usr/share/nopCommerce` to `nop`
```
sudo chgrp -R nop /usr/share/nopCommerce/
sudo chown -R nop /usr/share/nopCommerce/
```
![alt text](shots/38.PNG)

* Create a file in `/etc/systemd/system/nopCommerce.service` with following content :
```
sudo vi /etc/systemd/system/nopCommerce.service
```
```
[Unit]
Description=Example nopCommerce app running on Xubuntu

[Service]
WorkingDirectory=/usr/share/nopCommerce
ExecStart=/usr/bin/dotnet /usr/share/nopCommerce/Nop.Web.dll
Restart=always
# Restart service after 10 seconds if the dotnet service crashes:
RestartSec=10
KillSignal=SIGINT
SyslogIdentifier=nopCommerce-example
User=nop
Environment=ASPNETCORE_ENVIRONMENT=Production
Environment=ASPNETCORE_URLS=http://0.0.0.0:5000
Environment=DOTNET_PRINT_TELEMETRY_MESSAGE=false

[Install]
WantedBy=multi-user.target
```
![alt text](shots/39.PNG)

* Now execute with the following commands :
```
sudo systemctl enable nopCommerce.service
sudo systemctl start nopCommerce.service
sudo systemctl status nopCommerce.service
```
![alt text](shots/40.PNG)

* Now access the application using `http://<public_ip>:5000`

![alt text](shots/17.PNG)

* Enabling mysql connections from anywhere 

    [ Refer Here : https://www.techrepublic.com/article/create-mysql-8-database-user-remote-access-databases/ ]

* Step-1 :

* The first thing we must do is configure MySQL for remote connections. To do this, log into your MySQL database server and open the configuration file with the command :
```
sudo nano /etc/mysql/mysql.conf.d/mysqld.cnf
```
* In that file, look for the line :
```
bind-address = 127.0.0.1
```
* Change that line to :
```
bind-address = 0.0.0.0
```
![alt text](shots/41.PNG)

* Save and close the file. Restart the MySQL service with :
```
sudo systemctl restart mysql
sudo systemctl status mysql
```
![alt text](shots/42.PNG)

* Step-2 : Launch mysql shell `sudo mysql`
```
CREATE USER 'nop'@'%' IDENTIFIED BY 'nop12345';
GRANT ALL PRIVILEGES ON *.* to 'nop'@'%';
FLUSH PRIVILEGES;
exit
```
![alt text](shots/43.PNG)

* Let's configure ecommerce application in webpage `http://<ip-address>:5000` and create some sample data

![alt text](shots/44.PNG)
![alt text](shots/45.PNG)

* Enter database details and credentials `nop / nop12345`
* Restart and wait for few minutes for the page to be reloaded

![alt text](shots/18.PNG)

### AWS Migration P2V 

* For the doc's :

    [ Refer Here : https://docs.aws.amazon.com/mgn/latest/ug/what-is-application-migration-service.html ] 

* For the first time setup guide :

    [ Refer Here : https://docs.aws.amazon.com/mgn/latest/ug/first-time-setup-gs.html ]

* Steps :

=> Navigate to Services => Migration & Transfer => AWS Application Migration Service => Set up Application => Set up service

![alt text](shots/19.PNG)

=> Source servers => Add server

![alt text](shots/20.PNG)

* Create an IAM user with `AWS Application Migration Agent Installation Policy` or `Administrator Access`

* Now follow the instructions as mentioned in the page :

=> AWS Replication Agent installation => Select operating system : Linux => Select replication preferences : Replicate all disks => provide required credentials => IAM access key ID :   => IAM secret access key :  

![alt text](shots/21.PNG)

=> Run the installer command on the dbserver => Run the input command for the source server to install the Replication Agent => Back

![alt text](shots/22.PNG)

=> After logging into _**dbserver ( Linux machine )**_ => resize the machine atleast to `t2.small` => run the commands mentioned below
```
wget ...
ls
sudo python3 aws-replication-installer-init.py --region ap-south-1 --aws-access-key-id '' --aws-secret-access-key '' --no-prompt 
```
![alt text](shots/23.PNG)

* We also set up the Replication Agent on the nop machine ( Linux Instance) too with the same steps

![alt text](shots/24.PNG)

* Check for the results at the Application Migration Service

![alt text](shots/25.PNG)
![alt text](shots/26.PNG)
![alt text](shots/27.PNG)

* we need to wait for initial sync to complete and then start replication

![alt text](shots/28.PNG)

* We were able to launch test instances, finalize testing, launch cutover instances and finalize cutover
* If the machines aren't working, just try rebooting  and restart them 

=> Application Migration Service => After it reaches the `Ready for testing phase` => Select the machine => Navigate to Launch template => Unable `Active instance ritesizing` => Default instance size : t2.micro => Save template => goto Test and cutover => Launch test instances 

* After the launch of instances we proceed to the cutover and automatically the testing instances get terminated 
* And if the launch fails we again go through the testing

=> Test and cutover => Mark as "Ready for cutover" => Launch cutover instances OR Revert to "Ready for testing"

![alt text](shots/46.PNG)

#### Lifecycle states 

 1. _**Not ready**_ – The server is undergoing the Initial Sync process and is not yet ready for testing
    * Data replication can only commence once all of the Initial Sync steps have been completed

 2. _**Ready for testing**_ – The server has been successfully added to AWS Application Migration Service and data replication has started
    * Test or cutover instances can now be launched for this server

 3. _**Test in progress**_ – A test instance is currently being launched for this server

 4. _**Ready for cutover**_ – This server has been tested and is now ready for a cutover instance to be launched

 5. _**Cutover in progress**_ – A cutover instance is currently being launched for this server

 6. _**Cutover complete**_ – This server has been cutover
    *  All of the data on this server has been migrated to the cutover instance

* The lifecycle always displays the Launch status, Last test, and Cutover status for each server that has undergone these stages

* After the complete process we can delete all the created instances and if needed we can also archive

=> Test and cutover => Terminate launched instances => ok

=> Actions => mark as Archive

* Here `Disk = volume` and `Snapshot = Backup of volume`

### AWS Database Migration Service

* Overview

![alt text](shots/14.PNG)

* We will be migrating from on-prem mysql to AWS rds for mysql
* Let's create an AWS rds for mysql

=> Navigate to Services => Select RDS => create databse => Create method : Standard create => Engine typr : Mysql => Templates : Free tier => DB instance identifier : qtnopsrvmigrated => Master username : nop => Master password : nop12345 => confirm password => Unable Enable storage autoscaling => Public access : yes => security group with 3306 port open => Create database

* Let's create replication instances

=> Navigate to Services => Select Database Migration Service => Replication instances => Create replication instance => Name : qtdbmigrate => Description: For migration => instance class : dms.t3.micro => Allocated storage 50GB => Select public accessible => Security group with 3306 port open => choose failover instance : Dev or test => Create replication instance

* Wait untill they get available

=> Goto endpoints => Create endpoint => select Source endpoint => Endpoint identifier : SourceDBid => Source engine : MYSQL => Access to endpoint database : Provide access information manually => Server name : <public_ip> of ec2 instance => Port : 3306 => Username : nop => Password : nop12345 => Test endpoint connection : Run test => Create endpoint

* Now selecting target point

=> => Goto endpoints => Create endpoint => select Target endpoint => Select RDS DB instance => Endpoint identifier : qtnopsrvmigrated => Target engine : MYSQL => 
* Let's create a migration task

=> Select database mifration tasks => Create task => Task identifier : migrate => select the created replication instance => Source database endpoint : sourcedbid => target database endpoint : qtnopsrvmigrated => Migration type : Migrate existing data

=> Selection rules => Schema : Enter a schema => Source name : nopCommerce => Source table name : % => Strart migration task : Automatically on create => Create task 

* Wait till the migration gets completed

### Storage Migration

* AWS supports two types of storage migrations :
    * _**Online**_ :
        * Snowball edge
        * Storage Gateway ( only for a specific type of data ) with local tape gateway, file gateway and volume gateway
    * _**Offline**_ : To perform offline transfers we have AWS Snow Family devices ( One time activity )
        * Snowcone
        * Snowball
        * Snowmobile

### Modernization

* After workloads are migrated to AWS, we can modernize them to Cloud Native Architectures
* Popular options
    * Microservices
        * Event based Micro-services
    * Serverless
    * Data architectures
        * Data lakes
        * Data Analytics
        * Integration with AI( Artificial Intelligence ) / ML ( Machine Learning )

### AWS Backup and Disaster Recovery

* BCDR (Business Continuity and Disaster Recovery)

![alt text](shots/29.PNG)

### AWS Backup Service

* Supported Resources 

    [ Refer Here : https://docs.aws.amazon.com/aws-backup/latest/devguide/whatisbackup.html#supported-resources ]

#### Creating a Backup for EC2 instance

* Create an ec2 instance and install something ( java , apache , nginx )
```
sudo apt update
sudo apt install nginx -y
```
* Let's create a backup plan

=> Navigate through Services => Select AWS Backup => Create Backup plan

=> select Start with a template => Daily-35day-Retention => Backup plan name : Daily => Create a new vault => vault name : qt => Encription : (default) => create vault

=> Backup rules : Daily backups => Edit => Backup frequency : Daily => Retention period : 2 Days => Create plan 

=> Assign resource : ec2 => Include specific resource types => Assign resources => continue

![alt text](shots/30.PNG)

* Ensure you choose resources filtered by tags

tags => Env Equals Prod => assign

* Wait for the schedule to see the backup or we can create an on-demand backup

![alt text](shots/31.PNG)
