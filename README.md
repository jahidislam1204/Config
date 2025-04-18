# Install and configure SonarQube on AWS EC2 ubuntu 22.04 and 20.04 (Full Setup)



Server Specification (Please choose as per your requirement):

OS = Ubuntu 22.04 LTS

CPU: 2 vCPU

RAM: 4 GB

Storage: 20 GB

or 

use -> t2 micro


security group edit----->> 

i) HTTP = 80

ii) HTTPS = 443

iii) SSH = 22 (open to only required IP)

iv) Custom port = 9000

## .......Follow the step carefully......

**1) SSH into the AWS EC2 Ubuntu Server**
```bash 
ssh -i bastion-1.pem  ubuntu@43.217.7.7
```
**2) Let's rejuvenate our Ubuntu server**
```bash 
sudo apt update
sudo apt upgrade -y
```
**3) Install OpenJDK 17**

**i) Install OpenJDK 17 (needed for the latest version of SonarQube (version 10.0).**
```bash 
sudo apt install -y openjdk-17-jdk
```
**ii) Let's check the installed version of Java. VALIDATION IS IMPORTANT.**
```bash 
java -version
```

















