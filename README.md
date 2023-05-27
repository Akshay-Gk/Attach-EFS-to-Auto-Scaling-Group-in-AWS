# **Attach EFS to Auto Scaling Group**
### Description:
This would help someone who runs an application/website in AWS Autoscaling Group(ASG), who came across the issue that once the existing instance got terminated/stopped due to any reason  ASG automatically creates a new instance from the AMI or the Launch configuration but loss the data stored inside the instance/ebs. In this scenario, Elastic File System(EFS) can be a solution.
Amazon EFS provides shared file storage for use with compute instances in the AWS Cloud and on-premises servers.
Here I'm using a simple website to demonstrate the working.

### Diagram:
![image](https://github.com/Akshay-Gk/AWS-projects/assets/112197849/9ef0ca63-dde5-4fec-a8a2-fb60b9f2566f)


### Benefits of EFS:
* EFS can be mounted to multiple instances at a time
* Scalable network storage
* Cost efficient

### Steps:

1. Create an EFS 

a) *Log in to aws console > EFS > Create File System*
b) *Type name and select desired VPC then click customize*
 ![image](https://github.com/Akshay-Gk/AWS-projects/assets/112197849/ab3ace2b-8d60-4d0f-bb84-f9aa8685a37b)
 
c) *In Storage Class we can see two options*
- Standard - Data is stored across multiple AZ's
- One Zone - Data is stored in single AZ
> `Note: It would be ideal to select Standard since the data are stored in multiple AZ's. Data would not be lost in case of any issue with the AZ` 

d) *Select Automatic backups option as per need. Here im not selecting it*
 ![image](https://github.com/Akshay-Gk/AWS-projects/assets/112197849/049a5782-16fe-4fe0-a318-e4febd0f2d7e)
 
e) *Lifecycle management ,Amazon EFS supports two lifecycle policies*:
 - Transition into IA - It instructs lifecycle management when to transition files into the file system's Infrequent access storage class.
 - Transition out of IA - It instructs lifecycle management when to transition files out of IA storage.
 - 
> `#RRGGBBNote : Bill generated will differ as per the option you choose.To know more about this click`                            https://docs.aws.amazon.com/efs/latest/ug/lifecycle-management-efs.html
- Enable Encryption for the data if neccessery.
- Here im choosing 30 days in Transition into IA and Not enabling Encryption.
![image](https://github.com/Akshay-Gk/AWS-projects/assets/112197849/6e0c7a90-adce-4902-80aa-fc775f27f0ee)

f) *Perfomance Settings*
- Here we can choose the throughput and perfomace of the file system.
- Enhanced - For workloads with a range of performance requirements
- Bursting - For workloads with basic performance requirements.
- General Purpose - This is recommended performance mode for file systems. File systems with EFS One Zone storage classes always use this                     mode.
- Max I/O - This is designed for highly parallelized workloads
- Here we choose Bursting and General purpose.
![image](https://github.com/Akshay-Gk/AWS-projects/assets/112197849/9b8a44f7-16be-4648-abfa-3473e3d643e1)
- Add tags if needed.
 
> Bill amount differs as per the mode we choose. 
 
g) *Choose the desired VPC and Mount Targets ie, subnets*
- Modify the security groups for the subnets as per need. Here i choose all traffic, which is not recommended.
![image](https://github.com/Akshay-Gk/AWS-projects/assets/112197849/2eb54181-5f2c-4576-a240-250d0338bc87)

h) *File system policy (optional)*
- Select one or more of these common policy options, or create a custom policy as per need. Here im skipping this.
![image](https://github.com/Akshay-Gk/AWS-projects/assets/112197849/f11b2ce9-d035-46dd-8a8b-9828dbb023f6)

i) *Review the seleted settings and click create*
* Now we can see created EFS
![image](https://github.com/Akshay-Gk/AWS-projects/assets/112197849/f13a5d49-695b-49a7-b939-73fce7c8e6fd)
> Note: We would be using this highlighted file system ID in coming steps.


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
fs-0af3edb8356c16d77:/    /var/www/html/  efs  defaults,_netdev 0  0
Mount -a
```
4. Create a Launch configuration(LC) using below user data:
```
#!/bin/bash

yum install amazon-efs-utils httpd php  -y
echo "fs-0af3edb8356c16d77:/    /var/www/html/  efs  defaults,_netdev  0  0"  >> /etc/fstab
mount -a
systemctl restart httpd php-fpm
systemctl enable httpd php-fpm
```
> This bash script helps in mounting EFS to new instances automatically. 

5. Create an Autoscaling group(ASG) using the Launch configuration.
* Attach Loadbalancer if required. 
> We can use **Classic Load Balanacer(CLB)**, use **Route53 service** for domain name & hosting and **ACM service** for SSL certification if needed.
  
* click below link for more details:
* https://docs.aws.amazon.com/autoscaling/ec2/userguide/create-asg-launch-template.html
* https://docs.aws.amazon.com/elasticloadbalancing/latest/classic/elb-getting-started.html
* https://docs.aws.amazon.com/autoscaling/ec2/userguide/create-launch-config.html


### Conclusion:
Overall, what I have done is connected highly scalable EFS to instances in the autoscaling group.

Note: Please ensure to choose the appropriate settings while creating all the above as per requirement.



