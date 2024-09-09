# Load Balancer Solution With Nginx and SSL/TLS-102

## DevOps/Cloud Engineering Load Balancer Solution With Nginx and SSL/TLS-102

### Status: Complete

---

## Part 1 - Configure Nginx As A Load Balancer

You can either uninstall Apache from the existing Load Balancer server or create a fresh installation of Linux for Nginx.

1. Create an EC2 VM based on Ubuntu Server 20.04 LTS and name it `Nginx LB`. (Do not forget to open TCP port 80 for HTTP connections and TCP port 443 for secured HTTPS connections.)

2. Update the `/etc/hosts` file for local DNS with Web Servers' names (e.g., Web1 and Web2) and their local IP addresses.

3. Install and configure Nginx as a load balancer to point traffic to the resolvable DNS names of the web servers.

    ```bash
    # Update the instance and install Nginx
    sudo apt update
    sudo apt install nginx
    ```

4. Configure Nginx LB using the Web Servers' names defined in `/etc/hosts`.

    **Hint:** Read this blog to learn more about `/etc/hosts`.

5. Open the default Nginx configuration file:

    ```bash
    sudo vi /etc/nginx/nginx.conf
    ```

6. Insert the following configuration into the `http` section:

    ```nginx
    upstream myproject {
        server Web1 weight=5;
        server Web2 weight=5;
    }

    server {
        listen 80;
        server_name www.domain.com;
        location / {
            proxy_pass http://myproject;
        }
    }
    ```

7. Comment out this line:

    ```nginx
    # include /etc/nginx/sites-enabled/*;
    ```

8. Restart Nginx and make sure the service is up and running:

    ```bash
    sudo systemctl restart nginx
    sudo systemctl status nginx
    ```
# Part 2 - Register a New Domain Name and Configure Secured Connection Using SSL/TLS Certificates

Let's make the necessary configurations to secure connections to our Tooling Web Solution!

## Step 1: Register a New Domain Name

To obtain a valid SSL certificate, you need to register a new domain name. You can do this through any domain name registrar. Some of the most popular ones are:

- [GoDaddy.com](https://www.godaddy.com)
- [Domain.com](https://www.domain.com)
- [Bluehost.com](https://www.bluehost.com)

### Instructions:
1. Register a new domain name with any registrar of your choice in any domain zone (e.g., .com, .net, .org, .edu, .info, .xyz, or any other).
2. Assign an Elastic IP to your Nginx LB server and associate your domain name with this Elastic IP.

    - Every time you restart or stop/start your EC2 instance, you receive a new public IP address. To avoid this, associate your domain name with a static IP address that does not change after a reboot. Elastic IP solves this problem. Learn how to allocate an Elastic IP and associate it with an EC2 server on [this page](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/elastic-ip-addresses-eip.html).

3. Update the A record in your domain registrar to point to Nginx LB using the Elastic IP address.
![alt text](<Screenshot 2024-08-25 161317.png>)

![alt text](<Screenshot 2024-08-25 173922.png>)

    - Learn how to associate your domain name with your Elastic IP on [this page](https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/routing-to-elastic-ip-address.html).

### Side Self-Study:
Read about different DNS record types and learn what they are used for.

4. Verify that your web servers can be reached from your browser using the new domain name via the HTTP protocol: `http://<your-domain-name.com>`.

## Step 2: Configure Nginx to Recognize Your New Domain Name

1. Update your `nginx.conf` file:

    ```bash
    sudo vi /etc/nginx/nginx.conf
    ```

2. Modify the `server_name` directive to use your new domain name:

    ```nginx
    server_name www.<your-domain-name.com>;
    ```
![alt text](<Screenshot 2024-08-25 173731.png>)
## Step 3: Install Certbot and Request an SSL/TLS Certificate

1. Ensure the `snapd` service is active and running:

    ```bash
    sudo systemctl status snapd
    ```

2. Install Certbot:

    ```bash
    sudo snap install --classic certbot
    ```

3. Request your SSL/TLS certificate:

    ```bash
    sudo ln -s /snap/bin/certbot /usr/bin/certbot
    sudo certbot --nginx
    ```

    - Follow the Certbot instructions. You will need to choose the domain for which you want the certificate issued. The domain name will be looked up from the `nginx.conf` file, so ensure it was updated in the previous step.

4. Test secured access to your web solution by trying to reach `https://<your-domain-name.com>`.

    - You should be able to access your website using the HTTPS protocol (which uses TCP port 443) and see a padlock icon in your browser's address bar. Click on the padlock icon to view the details of the certificate issued for your website.

## Step 4: Set Up Periodical Renewal of Your SSL/TLS Certificate

By default, a Let's Encrypt certificate is valid for 90 days, so it is recommended to renew it at least every 60 days or more frequently.

1. Test the renewal command in dry-run mode:

    ```bash
    sudo certbot renew --dry-run
    ```

2. Set up a cron job to run the renewal command twice a day. Edit the crontab file:

    ```bash
    crontab -e
    ```

3. Add the following line to the crontab file:

    ```bash
    * */12 * * * root /usr/bin/certbot renew > /dev/null 2>&1
    ```

    - You can adjust the interval of this cron job if twice a day is too often by modifying the schedule expression.
