AWS Console
↓
EC2
↓
Launch Instance

Name            : sonarqube-server
AMI             : Amazon Linux 2
Instance Type   : t3.medium
Key Pair        : Create New / Existing
Storage         : 30 GB gp3

Security Group:
SSH   (22)   -> My IP
TCP   (9000) -> 0.0.0.0/0

Launch Instance
↓
Copy Public IP


# Install Java
sudo dnf install java-17-amazon-corretto -y

# Verify Java
java -version

# Go to /opt
cd /opt

# Download SonarQube
wget https://binaries.sonarsource.com/Distribution/sonarqube/sonarqube-8.9.10.61524.zip

# Unzip
unzip sonarqube-8.9.10.61524.zip

# Rename
mv sonarqube-8.9.10.61524 sonar

cd /sonar
cd /bin
cd linux-x86-64/
cd ..
cd ..
cd conf
ll
sonar.properties to change port number if required and db config
cd ..
ll
cd lib
cd ..
cd logs
cd ..

cd /opt

# Create sonar user
useradd sonar
cd id sonar

# Change ownership
chown -R sonar:sonar sonar
ll
cd sonar
ll
cd ..


# Create service file
cd etc/systemd/system
ll
vim sonar.service
or 

cat > /etc/systemd/system/sonar.service <<EOF
[Unit]
Description=sonar 8.9.10 service
After=network.target

[Service]
Type=forking
LimitNOFILE=65536

User=sonar
Group=sonar

ExecStart=/opt/sonar/bin/linux-x86-64/sonar.sh start
ExecStop=/opt/sonar/bin/linux-x86-64/sonar.sh stop

Restart=Always

[Install]
WantedBy=multi-user.target
EOF


cd etc/systemd/system
cd chmod 777 sonar.service
ll
cd ..
cd ..
move to root

 
# Enable service
systemctl enable sonar.service

# Start service
systemctl start sonar.service

# Check status
systemctl status sonar.service

# Check process
ps -ef | grep sonar

# Check port
netstat -tulpn | grep 9000

# Open firewall if required
iptables -I INPUT -p tcp --dport 9000 -j ACCEPT
service iptables save

# Access SonarQube
# http://<EC2-Public-IP>:9000

# Default Credentials
# Username: admin
# Password: admin