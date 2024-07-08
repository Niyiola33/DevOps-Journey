# Setting Up WordPress on an EC2 Instance with RedHat OS

## Prerequisites

- An AWS account with the ability to launch EC2 instances.

## Notes

- For this project, we'll use RedHat instead of Ubuntu.
- When connecting to the RedHat instance via SSH, use the `ec2-user` user.
- The connection string will look like: `ec2-user@<Public-IP>`.

## Step 1: Prepare a Web Server

### Launch an EC2 Instance

1. **Launch an EC2 Instance:**
   - Select RedHat as the AMI.
   - Choose an appropriate instance type (e.g., t2.micro).
   - Configure the instance with the required security group settings.

2. **Create and Attach EBS Volumes:**
   - Create 3 EBS volumes of 10 GiB each in the same Availability Zone as your EC2 instance.
   - Attach all three volumes to your EC2 instance.
   
   ![alt text](WORDPRESS/Volumes.png)

### Configure the Server

1. **Connect to the EC2 Instance:**
   Open a terminal and connect via SSH:

   ```bash
   ssh ec2-user@<Public-IP>
   ```

2. **Inspect Block Devices:**
   Use the `lsblk` command to see attached block devices:

   ```bash
   lsblk
   ```

   ![alt text](WORDPRESS/lsblk.png)

3. **Create Partitions:**
   Use the `gdisk` utility to create a single partition on each of the 3 disks:

   ```bash
   sudo gdisk /dev/xvdf
   # Follow the on-screen instructions to create a partition
   ```

   Repeat the partitioning steps for `/dev/xvdg` and `/dev/xvdh`.

   ![alt text](WORDPRESS/Screenshot 2024-06-09 203543.png)

4. **Install LVM2:**
   Install the LVM2 package:

   ```bash
   sudo yum install -y lvm2
   ```

5. **Create Physical Volumes:**
   Mark each of the 3 disks as physical volumes:

   ```bash
   sudo pvcreate /dev/xvdf1
   sudo pvcreate /dev/xvdg1
   sudo pvcreate /dev/xvdh1
   ```

6. **Create a Volume Group:**
   Create a volume group named `webdata-vg`:

   ```bash
   sudo vgcreate webdata-vg /dev/xvdf1 /dev/xvdg1 /dev/xvdh1
   ```

   ![alt text](WORDPRESS/Screenshot 2024-06-09 204320.png)

7. **Create Logical Volumes:**
   Create two logical volumes: `apps-lv` and `logs-lv`:

   ```bash
   sudo lvcreate -n apps-lv -L 14G webdata-vg
   sudo lvcreate -n logs-lv -L 14G webdata-vg
   ```

   ![alt text](WORDPRESS/Screenshot 2024-06-09 204530.png)

8. **Format Logical Volumes:**
   Format the logical volumes with the ext4 filesystem:

   ```bash
   sudo mkfs -t ext4 /dev/webdata-vg/apps-lv
   sudo mkfs -t ext4 /dev/webdata-vg/logs-lv
   ```

   ![alt text](WORDPRESS/Screenshot 2024-06-09 204801.png)

9. **Create Directories:**
   Create directories for the website and logs:

   ```bash
   sudo mkdir -p /var/www/html
   sudo mkdir -p /home/recovery/logs
   ```

10. **Mount Logical Volumes:**
    Mount the logical volumes to the created directories:

    ```bash
    sudo mount /dev/webdata-vg/apps-lv /var/www/html/
    sudo rsync -av /var/log/ /home/recovery/logs/
    sudo mount /dev/webdata-vg/logs-lv /var/log
    sudo rsync -av /home/recovery/logs/ /var/log
    ```

    ![alt text](WORDPRESS/Screenshot 2024-06-09 205001.png)

11. **Update /etc/fstab:**
    Update the `/etc/fstab` file to ensure the mount configuration persists after a reboot:

    ```bash
    sudo blkid
    sudo vi /etc/fstab
    ```

    Add the entries for the logical volumes using their UUIDs.

    ![alt text](WORDPRESS/Screenshot 2024-06-09 205734.png)

12. **Reload Daemon and Verify Setup:**
    Test the configuration and reload the daemon:

    ```bash
    sudo mount -a
    sudo systemctl daemon-reload
    ```

    Verify your setup:

    ```bash
    df -h
    ```

    ![alt text](WORDPRESS/Screenshot 2024-06-09 203409.png)

## Step 2: Install and Configure WordPress

1. **Install Apache and PHP:**
   Install the required packages:

   ```bash
   sudo yum install -y httpd php php-mysqlnd
   ```

2. **Start and Enable Apache:**
   Start the Apache service and enable it to start on boot:

   ```bash
   sudo systemctl start httpd
   sudo systemctl enable httpd
   ```

3. **Download WordPress:**
   Download and extract WordPress:

   ```bash
   wget https://wordpress.org/latest.tar.gz
   tar -xzvf latest.tar.gz
   ```

4. **Configure WordPress:**
   Copy the WordPress files to the web server directory and configure permissions:

   ```bash
   sudo cp -R wordpress/* /var/www/html/
   sudo chown -R apache:apache /var/www/html/
   sudo chmod -R 755 /var/www/html/
   ```

5. **Create a WordPress Configuration File:**
   Rename the sample configuration file:

   ```bash
   sudo cp /var/www/html/wp-config-sample.php /var/www/html/wp-config.php
   ```

   Edit the `wp-config.php` file to set up your database connection.

6. **Restart Apache:**
   Restart Apache to apply the changes:

   ```bash
   sudo systemctl restart httpd
   ```

## Step 3: Prepare the Database Server

### Launch and Configure the DB Server

1. **Launch a RedHat EC2 Instance:**
   - Follow the same steps as for the Web Server EC2 instance.

2. **Create and Attach EBS Volumes:**
   - Create 3 EBS volumes of 10 GiB each and attach them to the DB Server EC2 instance.

3. **Configure the DB Server:**
   - Follow the same steps as for the Web Server, but instead of `apps-lv`, create `db-lv` and mount it to `/db` instead of `/var/www/html/`.

## Step 4: Install MySQL on your DB Server EC2

1. **Update and Install MySQL:**

    ```bash
    sudo yum update
    sudo yum install mysql-server
    ```

2. **Start and Enable MySQL:**

    ```bash
    sudo systemctl restart mysqld
    sudo systemctl enable mysqld
    ```

## Step 5: Configure DB to Work with WordPress

1. **Log in to MySQL:**

    ```bash
    sudo mysql
    ```

2. **Create Database and User:**

    ```sql
    CREATE DATABASE wordpress;
    CREATE USER `myuser`@`<Web-Server-Private-IP-Address>` IDENTIFIED BY 'mypass';
    GRANT ALL ON wordpress.* TO 'myuser'@'<Web-Server-Private-IP-Address>';
    FLUSH PRIVILEGES;
    SHOW DATABASES;
    exit
    ```

## Step 6: Configure WordPress to Connect to Remote Database

1. **Open MySQL Port:**
   Open MySQL port 3306 on DB Server EC2. Allow access ONLY from your Web Server's IP address.

2. **Install MySQL Client:**

    ```bash
    sudo yum install mysql
    ```

3. **Test MySQL Connection:**

    ```bash
    sudo mysql -u myuser -p -h <DB-Server-Private-IP-address>
    ```

    Verify the connection:

    ```sql
    SHOW DATABASES;
    ```

4. **Change Apache Permissions:**
   Ensure Apache can use WordPress.

5. **Enable TCP Port 80:**
   Enable TCP port 80 in Inbound Rules configuration for your Web Server EC2.

6. **Access WordPress:**
   Open your browser and navigate to `http://<Web-Server-Public-IP-Address>/wordpress/`. Fill out your DB credentials.

7. **Verify Connection:**
   If you see a success message, your WordPress has successfully connected to your remote MySQL database.

   ![alt text](WORDPRESS/Screenshot 2024-06-12 105343.png)
   ![alt text](WORDPRESS/Screenshot 2024-06-12 110442.png)
   ![alt text](WORDPRESS/Screenshot 2024-06-12 110633.png)

## Important

Do not forget to STOP your EC2 instances after completion of the project to avoid extra costs.

---

Congratulations! You have learned how to configure the Linux storage subsystem and have deployed a full-scale web solution using WordPress CMS
