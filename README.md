
# 🛠️ AWS EC2 Ansible Setup – Automated Configuration

This project demonstrates how to use Ansible to configure two AWS EC2 Amazon Linux instances. It covers secure SSH access, inventory setup, and running a playbook that performs real system configuration tasks.

---

## 📌 What This Project Does

With one playbook, Ansible will:

- ✅ Update all system packages
- 👤 Create a new user `devopsuser` with `sudo` access
- 🌐 Install and configure Nginx
- 🧾 Deploy a customized welcome page via Nginx on each machine

---

## 📁 Folder Structure

```
aws-ansible-ec2-setup/
├── inventory.ini           # Ansible inventory file with IPs and SSH key
├── aws_linux_setup.yml     # Main Ansible playbook
└── README.md               # This guide
```

---

## 🔧 Prerequisites

- Two AWS EC2 instances (Amazon Linux) running in eu-north-1b
- A valid `.pem` key pair assigned when launching the EC2 instances
- Ansible installed on your local system (Linux, WSL, or Mac)
- The `.pem` key must be securely stored with `chmod 400` permissions

---

## ✅ Step-by-Step Guide

### 🔑 Step 1: Copy Your Key and Set Permissions

```bash
cp "/mnt/c/Users/LAP TECH/Downloads/marwa-key.pem" ~/marwa-key.pem
chmod 400 ~/marwa-key.pem
```

### 🔌 Step 2: Verify SSH Access to Both Instances

```bash
ssh -i ~/marwa-key.pem ec2-user@13.51.180.123
ssh -i ~/marwa-key.pem ec2-user@16.171.25.40
```

Both should successfully connect to Amazon Linux 2023 instances.

### 🧾 Step 3: Create the Ansible Inventory File

📄 `inventory.ini`

```ini
[aws_linux]
machine1 ansible_host=13.51.180.123 ansible_user=ec2-user ansible_ssh_private_key_file=~/marwa-key.pem
machine2 ansible_host=16.171.25.40 ansible_user=ec2-user ansible_ssh_private_key_file=~/marwa-key.pem
```

### 🚨 Step 4: Test Ansible Connectivity

```bash
ansible -i inventory.ini aws_linux -m ping
```

Expected successful "pong" responses from both machines.

### 📝 Step 5: Create the Ansible Playbook

📄 `aws_linux_setup.yml`

```yaml
---
- name: Configure AWS Linux instances
  hosts: aws_linux
  become: yes
  tasks:

    - name: Update all system packages
      yum:
        name: '*'
        state: latest
      tags: update

    - name: Create a new user with sudo access
      user:
        name: devopsuser
        state: present
        groups: wheel
        append: yes
      tags: user

    - name: Install and start Nginx, and deploy a sample web page
      block:
        - name: Install nginx
          yum:
            name: nginx
            state: present

        - name: Start and enable nginx
          service:
            name: nginx
            state: started
            enabled: yes

        - name: Deploy sample index.html
          copy:
            dest: /usr/share/nginx/html/index.html
            content: "<h1>Welcome from Ansible on {{ inventory_hostname }}</h1>"
      tags: nginx
```

### 🚀 Step 6: Run the Playbook

```bash
ansible-playbook -i inventory.ini aws_linux_setup.yml
```

---

## ✅ Verification Steps

1. **New User Created**:  
   Confirmed in `/etc/passwd`:
   ```
   devopsuser:x:1001:1001::/home/devopsuser:/bin/bash
   ```

2. **Nginx Running**:  
   - Machine1: http://13.51.180.123 shows "Welcome from Ansible on machine1"  
   - Machine2: http://16.171.25.40 shows "Welcome from Ansible on machine2"

3. **System Updated**:  
   All packages were successfully updated via yum.

---

## 💡 Tips

- Ensure security groups allow inbound traffic on ports 22 (SSH) and 80 (HTTP)
- The playbook includes tags (`update`, `user`, `nginx`) for running specific tasks
- Check `/var/log/nginx/error.log` if the web page doesn't load

---

## 📸 Screenshot Proof

- Successful SSH connections to both instances (/screenshots/ssh-test.png)
- Ansible ping test returning "pong" (/screenshots/ping-test.png)
- Playbook execution with all tasks marked as "changed" or "ok" (/screenshots/playbook-run.png)
- Nginx welcome pages displaying on both machines (/screenshots/Nginx-test1.png),(/screenshots/Nginx-test2.png)
- New devopsuser visible in `/etc/passwd`(/screenshots/new-user-test.png),(/screenshots/new-user-test2.png)

---

## 🙋 About the Author

This project was created by **Marwa Elzoghby** to demonstrate real-world infrastructure automation using Ansible and AWS EC2.

GitHub: [marwa-elzoghby](https://github.com/marwa-elzoghby)  
LinkedIn: [Marwa Elzoghby](https://www.linkedin.com/in/marwa-elzoghby/)

---

> ✨ Feel free to fork this repo, modify the playbook, and use it in your own cloud automation workflows!
```
