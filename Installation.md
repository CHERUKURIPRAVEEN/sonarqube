# sonarqube
### Soanrqube Installation 
#### Prerequisites

Deploy a fully updated Ubuntu 2X.04 LTS server at AWS EC2 with at least 2GB of RAM and 1 vCPU cores.
Create a non-root user with sudo access.

SSH to your Ubuntu server as a non-root user with sudo access.
#### Install OpenJDK 11.

```sh
sudo apt-get install openjdk-11-jdk -y
 ```
#### Install and Configure PostgreSQL

Add the PostgreSQL repository.
```sh 
sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt/ `lsb_release -cs`-pgdg main" >> /etc/apt/sources.list.d/pgdg.list'
```
Add the PostgreSQL signing key.
```sh 
wget -q https://www.postgresql.org/media/keys/ACCC4CF8.asc -O - | sudo apt-key add -
```
#### Install PostgreSQL.
```sh
sudo apt install postgresql postgresql-contrib -y
```
Enable the database server to start automatically on reboot.
```sh 
sudo systemctl enable postgresql
```
Start the database server.
```sh
sudo systemctl start postgresql
```
Change the default PostgreSQL password.
```sh
sudo passwd postgres
```
Switch to the postgres user.
```sh
su - postgres
```
Create a user named sonar.
```sh
createuser sonar
```
Log in to PostgreSQL.
```sh
psql
```
Set a password for the sonar user. Use a strong password in place of my_strong_password.
```sh
ALTER USER sonar WITH ENCRYPTED password 'my_strong_password';
```
Create a sonarqube database and set the owner to sonar.
```sh
CREATE DATABASE sonarqube OWNER sonar;
```
Grant all the privileges on the sonarqube database to the sonar user.

GRANT ALL PRIVILEGES ON DATABASE sonarqube to sonar;
Exit PostgreSQL.
```sh
\q
```
Return to your non-root sudo user account.
```sh
exit
```
#### Download and Install SonarQube
Install the zip utility, which is needed to unzip the SonarQube files.
```sh
sudo apt-get install zip -y
```
Locate the latest download URL from the SonarQube official download page.
Download the SonarQube distribution files.
```sh
sudo wget https://binaries.sonarsource.com/Distribution/sonarqube/sonarqube-<VERSION_NUMBER>.zip
```
Unzip the downloaded file.
```sh
sudo unzip sonarqube-<VERSION_NUMBER>.zip
```
Move the unzipped files to /opt/sonarqube directory
```sh
sudo mv sonarqube-<VERSION_NUMBER> /opt/sonarqube
```
#### Add SonarQube Group and User
Create a dedicated user and group for SonarQube, which can not run as the root user.
Create a sonar group.
```sh 
sudo groupadd sonar
```
Create a sonar user and set /opt/sonarqube as the home directory.
```sh
sudo useradd -d /opt/sonarqube -g sonar sonar
```
Grant the sonar user access to the /opt/sonarqube directory.
```sh
sudo chown sonar:sonar /opt/sonarqube -R
```
#### Configure SonarQube
Edit the SonarQube configuration file.
```sh
sudo nano /opt/sonarqube/conf/sonar.properties
```
Find the following lines:
```sh
#sonar.jdbc.username=
#sonar.jdbc.password=
```
Uncomment the lines, and add the database user and password you created in Step 2.
```sh
sonar.jdbc.username=sonar
sonar.jdbc.password=my_strong_password
```
Below those two lines, add the sonar.jdbc.url.
```sh
sonar.jdbc.url=jdbc:postgresql://localhost:5432/sonarqube
```
Save and exit the file.
Edit the sonar script file.
```sh
sudo nano /opt/sonarqube/bin/linux-x86-64/sonar.sh
```
About 50 lines down, locate this line:
```sh
#RUN_AS_USER=
```
Uncomment the line and change it to:
```sh
RUN_AS_USER=sonar
```
Save and exit the file.

#### Setup Systemd service
Create a systemd service file to start SonarQube at system boot.
```sh
sudo vi /etc/systemd/system/sonar.service
```
Paste the following lines to the file.
```sh
[Unit]

Description=SonarQube service
After=syslog.target network.target

[Service]

Type=forking
ExecStart=/opt/sonarqube/bin/linux-x86-64/sonar.sh start
ExecStop=/opt/sonarqube/bin/linux-x86-64/sonar.sh stop
User=sonar
Group=sonar
Restart=always
LimitNOFILE=65536
LimitNPROC=4096

[Install]

WantedBy=multi-user.target
```

Save and exit the file.

Enable the SonarQube service to run at system startup.
```sh
sudo systemctl enable sonar
```
Start the SonarQube service.
```sh
sudo systemctl start sonar
```
Check the service status.
```sh
sudo systemctl status sonar
```

#### Modify Kernel System Limits
SonarQube uses Elasticsearch to store its indices in an MMap FS directory. It requires some changes to the system defaults.

Edit the sysctl configuration file.
```sh
sudo nano /etc/sysctl.conf
```
Add the following lines.
```sh
vm.max_map_count=262144
fs.file-max=65536
ulimit -n 65536
ulimit -u 4096
```
Save and exit the file.

Reboot the system to apply the changes.
```sh
sudo reboot
```
