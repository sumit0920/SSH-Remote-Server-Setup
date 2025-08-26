# SSH-Remote-Server-Setup
Secure SSH access with multiple keys and Fail2Ban protection — setup guide and documentation.

# 🔗 Project URL:
```https://roadmap.sh/projects/ssh-remote-server-setup```

# Project Goal
- Configure a remote Linux server (AWS EC2 used here)
- Add two different SSH keys and verify login with both
- Optionally configure ~/.ssh/config for easy SSH alias
- Stretch goal: Install Fail2Ban to protect against brute force attacks

# Steps Taken
1. Provisioned a Remote Linux Server
2. Created an al2023 EC2 instance on AWS
3. Allowed SSH (port 22) in the AWS security group firewall

<img width="1588" height="729" alt="Screenshot 2025-08-26 175549" src="https://github.com/user-attachments/assets/a2f38dc2-3634-40c4-b662-c739479b9dd1" />

# 2. Generated Two SSH Key Pairs

On my local system:

```
ssh-keygen -t rsa -b 4096 -C "key1 for server" -f ~/.ssh/server_key1
ssh-keygen -t rsa -b 4096 -C "key2 for server" -f ~/.ssh/server_key2
```


This created:
```
~/.ssh/server_key1 + ~/.ssh/server_key1.pub

~/.ssh/server_key2 + ~/.ssh/server_key2.pub
```

# 3. Added Both Keys to the Server

Logged into AWS using the first key (created during EC2 launch):

ssh -i ~/.ssh/aws_default.pem ec2-user@<server-ip>

Then added both new public keys to ~/.ssh/authorized_keys:
```
cat <<EOF >> ~/.ssh/authorized_keys
# key1 for server
<contents-of-server_key1.pub>

# key2 for server
<contents-of-server_key2.pub>
EOF
```
# Set correct permissions:
```
chmod 600 ~/.ssh/authorized_keys
chmod 700 ~/.ssh
```

# 4. Verified Login with Both Keys
```
ssh -i ~/.ssh/server_key1 ec2-user@<server-ip>
ssh -i ~/.ssh/server_key2 ec2-user@<server-ip>
```
Both worked successfully. 🎉

# 5. Configured SSH Alias (Optional)
On local machine added ~/.ssh/config:
```
Host myserver-key1
    HostName <server-ip>
    User ubuntu
    IdentityFile ~/.ssh/server_key1

Host myserver-key2
    HostName <server-ip>
    User ubuntu
    IdentityFile ~/.ssh/server_key2

```
Now I can simply run:
``
ssh myserver-key1
ssh myserver-key2
``
# 6. Installed Fail2Ban (Stretch Goal)

On the AWS server:
```
sudo apt update
sudo apt install fail2ban -y
sudo systemctl enable fail2ban
sudo systemctl start fail2ban
```

# Enabled SSH protection:
```
sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local
sudo nano /etc/fail2ban/jail.local
# In [sshd] section:
# enabled = true
```

Checked status:
```
sudo systemctl status fail2ban
sudo fail2ban-client status sshd
```

Usage

Connect with first key:
```
ssh -i ~/.ssh/server_key1 ubuntu@<server-ip>
```

Connect with second key:
```
ssh -i ~/.ssh/server_key2 ubuntu@<server-ip>
```

Or use simple aliases:
```
ssh myserver-key1
ssh myserver-key2
```
# Repository Note

No private keys or sensitive files are included. This repository only contains documentation (README.md) explaining the setup steps.

# License

This project is licensed under the MIT License. You are free to use or modify these instructions.

# Credits
Project idea from roadmap.sh Implementation by Sumit (@sumit0920)
