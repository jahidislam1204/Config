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
ssh -i jahid.pem  ubuntu@Public_ip
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
**4) Install and Configure PostgreSQL**

**i) Add the PostgreSQL repository.**
```bash
sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt/ `lsb_release -cs`-pgdg main" /etc/apt/sources.list.d/pgdg.list'
```

**ii) Add the PostgreSQL signing key.**
```bash
wget -q https://www.postgresql.org/media/keys/ACCC4CF8.asc -O - | sudo apt-key add -

```
**iii) Install PostgreSQL.**
```bash
sudo apt install postgresql postgresql-contrib -y
```
**iv) Enable the database server to start automatically on reboot.**
```bash
sudo systemctl enable PostgreSQL
```
**v) Start the database server.**
```bash
sudo systemctl start postgresql
```
**vi) Check the status of the database server**
```bash
sudo systemctl status postgresql
```
**vii) Let’s check the installed version of the install Postgres DB. VALIDATION IS IMPORTANT.**
```bash
psql --version
```
**viii) Switch to the Postgres user.**
```bash
sudo -i -u postgres
```
**ix) Create a database user named ddsonar.**

Note: You can provide the name of your own but make a note of it, as we will be needing this name in further steps.
```bash
createuser ddsonar
```
**x) Log in to PostgreSQL.**
```bash
psql
```
**xi) Set a password for the ddsonar user. Use a strong password in place of my_strong_password.**
```bash
ALTER USER [Created_user_name] WITH ENCRYPTED password 'my_strong_password';
```
For example (In my case it will be):
```bash
ALTER USER ddsonar WITH ENCRYPTED password 'mwd#2%#!!#%rgs';
```
**xii) Create a SonarQube database and set the owner to ddsonar.**
```bash
CREATE DATABASE [database_name] OWNER [Created_user_name];
```
For example (In our case it will be) — feel free to give an awesome database name as per your requirement:
```bash
CREATE DATABASE ddsonarqube OWNER ddsonar;
```
**xiii) Grant all the privileges on the ddsonarqube database to the ddsonar user.**
```bash
GRANT ALL PRIVILEGES ON DATABASE ddsonarqube to ddsonar;
```
**xiv) Let's check the created user and the database.**

**a) To check the created database**
```bash
\l
```
**b) To check the created database user**
```bash
\du
```
**xv) Exit PostgreSQL.**
```bash
\q
```
**xvi) Return to your non-root sudo user account.**
```bash
exit
```

**5) Download and Install SonarQube**
**i) Install the zip utility, which is needed to unzip the SonarQube files.**
```bash
sudo apt install zip -y
```

**ii) Locate the latest download URL from the SonarQube official download page.**

Download the SonarQube distribution files. (you can download the latest SonarQube distribution using the following link)

https://www.sonarsource.com/products/sonarqube/downloads/

Here we are installing the latest version of SonarQube 10.0 community edition (free one)
```bash
sudo wget https://binaries.sonarsource.com/Distribution/sonarqube/sonarqube-10.0.0.68432.zip
```

**iii) Unzip the downloaded file.**
```bash
sudo unzip sonarqube-10.0.0.68432.zip
```
**iv) Move the unzipped files to /opt/sonarqube directory**
```bash
sudo mv sonarqube-10.0.0.68432 sonarqube
```
**6) Add SonarQube Group and User**
```bash
Create a dedicated user and group for SonarQube, which can not run as the root user.
```
Note: You can give any name for the sonar user and group. I have here given the user and group name to be the same i.e ddsonar.

**i) Create a sonar group.**
```bash
sudo groupadd ddsonar
```
**ii) Create a sonar user and set /opt/sonarqube as the home directory.**
```bash
sudo useradd -d /opt/sonarqube -g ddsonar ddsonar
```
**iii) Grant the sonar user access to the /opt/sonarqube directory.**
```bash
sudo chown ddsonar:ddsonar /opt/sonarqube -R
```

**7) Configure SonarQube**
**i) Edit the SonarQube configuration file.**
```bash
sudo vi /opt/sonarqube/conf/sonar.properties
```
**a) Find the following lines:**
```bash
#sonar.jdbc.username=

#sonar.jdbc.password=
```
**b) Uncomment the lines, and add the database user and Database password you created in Step 4 (xi and xii). For me, it’s:**
```bash
sonar.jdbc.username=ddsonar

sonar.jdbc.password=mwd#2%#!!#%rgs
```
**c) Below these two lines, add the following line of code.**
```bash
sonar.jdbc.url=jdbc:postgresql://localhost:5432/ddsonarqube
```
Here, ddsonarqube is the database name created.

d) Save and exit the file.

**ii) Edit the sonar script file.**
```bash
sudo vi /opt/sonarqube/bin/linux-x86-64/sonar.sh
```
**a) Add the following line**
```bash
RUN_AS_USER=ddsonar
```
Here, ddsonar is the name of the user that we have created in step number 6 (ii).

d) Save and exit the file.

**ii) Edit the sonar script file.**
```bash
sudo vi /opt/sonarqube/bin/linux-x86-64/sonar.sh
```
**a) Add the following line**
```bash
RUN_AS_USER=ddsonar
```
Here, ddsonar is the name of the user that we have created in step number 6 (ii).


iii) Save and exit the file.

**iv) Enable the SonarQube service to run at system startup.**
```bash
sudo systemctl enable sonar
```
**v) Start the SonarQube service.**
```bash
sudo systemctl start sonar
```
**vi) Check the service status.**
```bash
sudo systemctl status sonar
```

**vii) Hurray, It's up and running.**

**9) Modify Kernel System Limits**
SonarQube uses Elasticsearch to store its indices in an MMap FS directory. It requires some changes to the system defaults.

**i) Edit the sysctl configuration file.**
```bash
sudo vi /etc/sysctl.conf
```
**ii) Add the following lines.**
```bash
vm.max_map_count=262144
fs.file-max=65536
ulimit -n 65536
ulimit -u 4096
```

iii) Save and exit the file.

**iv) Reboot the system to apply the changes.**
```bash
sudo reboot
```
**10) Access SonarQube Web Interface**
**i) Access SonarQube in a web browser at your server’s IP address on port 9000.**
```bash
For example, http://IP:9000
```
Note: the default username and password are admin and admin respectively.



**ii) Change the Old password with a New one.**

Log in with username admin and password admin. In the next step, SonarQube will prompt you to change your password. CHANGE THE PASSWORD.


**iii) And yes, we have successfully installed the SonarQube Community version 10.0 on AWS EC2 Ubuntu 22.**


# Congratulations, Our SonarQube has been installed successfully.
















