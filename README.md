linux-server-config

ssh -i aws_secret_key.pem ubuntu@ec2-13-127-52-134.ap-south-1.compute.amazonaws.com

sudo apt update
sudo apt upgrade
sudo adduser grader
sudo nano /etc/sudoers.d/grader
	grader ALL=(ALL) NOPASSWD:ALL

su - grader
sudo mkdir .ssh
sudo nano .ssh/authorized_keys
sudo chmod 700 .ssh
sudo chmod 644 .ssh/authorized_keys
sudo chown -R grader:grader /home/grader/.ssh
sudo service ssh restart

ssh -i grader grader@ec2-13-127-52-134.ap-south-1.compute.amazonaws.com
passphere = grader
password = grader

sudo nano /etc/ssh/sshd_config
PasswordAuthentication no
Port 2200
PermitRootLogin no
sudo service ssh restart

