# Tooling Website Deployment Automation with Continuous Integration

## Introduction to Jenkins- 102


## Step 1 - Install Jenkins Server

### 1.1. Create an AWS EC2 Server
- **Operating System**: Ubuntu Server 20.04 LTS
- **Instance Name**: Jenkins

### 1.2. Install JDK
Jenkins is a Java-based application, so you need to install JDK:
```bash
sudo apt update
sudo apt install default-jdk-headless
```

### 1.3. Install Jenkins
1. Add Jenkins repository key:
    ```bash
    wget -q -O - https://pkg.jenkins.io/debian-stable/jenkins.io.key | sudo apt-key add -
    ```

2. Add Jenkins repository:
    ```bash
    sudo sh -c 'echo deb https://pkg.jenkins.io/debian-stable binary/ > /etc/apt/sources.list.d/jenkins.list'
    ```

3. Update package index and install Jenkins:
    ```bash
    sudo apt update
    sudo apt-get install jenkins
    ```

### 1.4. Verify Jenkins Installation
Ensure Jenkins is running:
```bash
sudo systemctl status jenkins
```

By default, Jenkins uses TCP port 8080. Open this port by creating a new inbound rule in your EC2 Security Group.

![alt text](<Security Group Jenkins.png>)
### 1.5. Perform Initial Jenkins Setup
1. Access Jenkins in your browser:
    ```
    http://<Jenkins-Server-Public-IP-Address-or-Public-DNS-Name>:8080
    ```
    
![alt text](<Unlock Jenkins.png>)
2. Retrieve the default admin password:
    ```bash
    sudo cat /var/lib/jenkins/secrets/initialAdminPassword
    ```

3. Install suggested plugins and create an admin user.
   
![alt text](<Customize Jenkins.png>)
![alt text](<Screenshot 2024-08-04 150525.png>)

## Step 2 - Configure Jenkins to Retrieve Source Code from GitHub Using Webhooks

### 2.1. Enable Webhooks in GitHub
1. Go to your GitHub repository settings and enable webhooks.
   
![alt text](<Screenshot 2024-08-05 153939.png>)
### 2.2. Create a Jenkins Job
1. In Jenkins, click **"New Item"** and create a **"Freestyle project"**.
   
![alt text](<Screenshot 2024-08-04 152409.png>)
3. Configure Git repository:
    - Provide the URL of your GitHub repository.
    - Add credentials if required.

![alt text](<Screenshot 2024-08-04 155910.png>)
4. Save the configuration and manually trigger the build by clicking **"Build Now"**.

![alt text](<Screenshot 2024-08-04 210445.png>)
5. Check the build status under **"Build History"** and view **"Console Output"** for success.

![alt text](<Screenshot 2024-08-04 203550.png>)
### 2.3. Automate Builds
1. Configure the job to trigger from GitHub webhook:
    - Go to **"Configure"** and add a **"Build Trigger"** for GitHub webhook.

2. Configure **"Post-build Actions"** to archive build artifacts.

![alt text](<Screenshot 2024-08-04 155942.png>)
### 2.4. Test Automation
1. Make a change to any file in your GitHub repository (e.g., `README.md`) and push the changes.

![alt text](<Screenshot 2024-08-04 210445.png>)
2. Verify that a new build is triggered automatically and artifacts are saved on the Jenkins server.
```
ls /var/lib/jenkins/jobs/tooling_github/builds/<build_number>/archive/
```
## Step 3 - Configure Jenkins to Copy Files to NFS Server via SSH

### 3.1. Install "Publish Over SSH" Plugin
1. In Jenkins, go to **"Manage Jenkins"** > **"Manage Plugins"**.

2. Search for **"Publish Over SSH"** and install it.

### 3.2. Configure SSH Plugin
1. Go to **"Manage Jenkins"** > **"Configure System"**.

2. Scroll to the **"Publish over SSH"** section and configure:
    - Provide the path to your private key file (`.pem`).
    - Enter arbitrary name, hostname (NFS server private IP), username (`ec2-user`), and remote directory (`/mnt/apps`).

![alt text](<Screenshot 2024-08-04 223202.png>)
3. Test the SSH configuration to ensure it returns success. Ensure TCP port 22 on the NFS server is open.

![alt text](<Screenshot 2024-08-04 223202.png>)
### 3.3. Configure Job for SSH Transfer
1. Open the Jenkins job configuration page and add a **"Post-build Action"** for SSH transfer.

2. Configure it to send all build files to the remote directory. Use `**` to include all files and directories.

### 3.4. Verify SSH Transfer
1. Change a file in your GitHub repository and push the changes.

2. Confirm that a new build is triggered and files are transferred to `/mnt/apps` on the NFS server.

3. Check the updated file via SSH/Putty:
    ```bash
    cat /mnt/apps/README.md
    ```

![alt text](<Screenshot 2024-08-04 231433.png>)
If you see the changes made in your GitHub repository, the job is working as expected!










# hello
