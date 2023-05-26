# **Attach EFS to Auto Scaling Group**
###Description:
This would help someone who runs an application/website in AWS Autoscaling Group(ASG), who came across the issue that once the existing instance gets terminated/stopped due to any reason  ASG automatically creates a new instance from the AMI or The Launch configuration but loss the data stored inside the instance/ebs. In this scenario, Elastic File System(EFS) can be a solution.
Amazon EFS provides shared file storage for use with compute instances in the AWS Cloud and on-premises servers.
Here I'm using a simple website to demonstrate the working.

### Diagram:
![image](https://github.com/Akshay-Gk/AWS-projects/assets/112197849/ad48eda7-b14a-4268-a28a-17d6058e1566)

###Benefits of EFS:
* EFS can be mounted in multiple instances at a time
* Scalable network storage
* Cost efficient

### Steps:

1. Create an EFS 
 * Click the link for More details(https://docs.aws.amazon.com/efs/latest/ug/gs-step-two-create-efs-resources.html)
2. Create and setup an instance with the following commands:
```
sudo -i

yum install httpd php git -y
systemctl restart php-fpm.service httpd.service
systemctl enable php-fpm.service httpd.service
```
This is a simple website created by Fuji sir, We clone this for study purposes.
```
git clone https://github.com/Fujikomalan/aws-elb-site.git  /var/website/   
cp -r /var/website/*  /var/www/html/
chown -R apache:apache /var/www/html/*
```
3. Mount EFS to Document root
* Edit fstab
> Give your EFS id instead of a device name or UUID 
```
fs-09c0f4c52907c3e4f:/    /var/www/html/  efs  defaults,_netdev 0  0
Mount -a
```
4.Create a Launch configuration(LC) using below user data:
```
#!/bin/bash

yum install amazon-efs-utils httpd php  -y
echo "fs-09c0f4c52907c3e4f:/    /var/www/html/  efs  defaults,_netdev  0  0"  >> /etc/fstab
mount -a
systemctl restart httpd php-fpm
systemctl enable httpd php-fpm
```
> is bash script helps in connecting EFS to new instances automatically. 

5.Create a Autoscaling group(ASG) with the LC
* Attach Loadbalancer if required
* click link for more details


### Conclusion:
Overall, what I have done is connected highly scalable EFS to instances in the autoscaling group.
Note: Please ensure to choose the appropriate settings while creating an EFS as per requirement.



