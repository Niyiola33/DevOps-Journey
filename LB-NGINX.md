# Load Balancer Solution With Apache-102

## DevOps/Cloud Engineering Load Balancer Solution With Apache-102

### Status: Complete

---

## Prerequisites

Make sure that you have the following servers installed and configured within the previous project:

- Two RHEL8 Web Servers
- One MySQL DB Server (based on Ubuntu 20.04)
- One RHEL8 NFS Server

---

## Configure Apache As A Load Balancer

1. Create an Ubuntu Server 20.04 EC2 instance and name it `Project-8-apache-lb`, so your EC2 list will look like this:

2. Open TCP port 80 on `Project-8-apache-lb` by creating an Inbound Rule in Security Group.

3. Install Apache Load Balancer on `Project-8-apache-lb` server and configure it to point traffic coming to LB to both Web Servers:

    ```bash
    # Install apache2
    sudo apt update
    sudo apt install apache2 -y
    sudo apt-get install libxml2-dev

    # Enable following modules:
    sudo a2enmod rewrite
    sudo a2enmod proxy
    sudo a2enmod proxy_balancer
    sudo a2enmod proxy_http
    sudo a2enmod headers
    sudo a2enmod lbmethod_bytraffic

    # Restart apache2 service
    sudo systemctl restart apache2
    ```

4. Make sure apache2 is up and running:

    ```bash
    sudo systemctl status apache2
    ```

5. Configure load balancing:

    ```bash
    sudo vi /etc/apache2/sites-available/000-default.conf
    ```

    Add this configuration into the `<VirtualHost *:80> </VirtualHost>` section:

    ```apache
    <Proxy "balancer://mycluster">
        BalancerMember http://<WebServer1-Private-IP-Address>:80 loadfactor=5 timeout=1
        BalancerMember http://<WebServer2-Private-IP-Address>:80 loadfactor=5 timeout=1
        ProxySet lbmethod=bytraffic
        # ProxySet lbmethod=byrequests
    </Proxy>

    ProxyPreserveHost On
    ProxyPass / balancer://mycluster/
    ProxyPassReverse / balancer://mycluster/
    ```

6. Restart Apache server:

    ```bash
    sudo systemctl restart apache2
    ```

    The `bytraffic` balancing method will distribute incoming load between your Web Servers according to current traffic load. We can control in which proportion the traffic must be distributed by the `loadfactor` parameter.

    You can also study and try other methods, like: `bybusyness`, `byrequests`, `heartbeat`.

7. Verify that your configuration works by trying to access your LB's public IP address or Public DNS name from your browser:

    ```http
    http://<Load-Balancer-Public-IP-Address-or-Public-DNS-Name>/index.php
    ```

    **Note:** If in the previous project, you mounted `/var/log/httpd/` from your Web Servers to the NFS server - unmount them and make sure that each Web Server has its own log directory.

8. Open two SSH/Putty consoles for both Web Servers and run the following command:

    ```bash
    sudo tail -f /var/log/httpd/access_log
    ```

9. Try to refresh your browser page `http://<Load-Balancer-Public-IP-Address-or-Public-DNS-Name>/index.php` several times and make sure that both servers receive HTTP GET requests from your LB. New records must appear in each server's log file. The number of requests to each server will be approximately the same since we set `loadfactor` to the same value for both servers - it means that traffic will be distributed evenly between them.

    If you have configured everything correctly - your users will not even notice that their requests are served by more than one server.
## Optional Step - Configure Local DNS Names Resolution

Sometimes it is tedious to remember and switch between IP addresses, especially if you have a lot of servers under your management. We can configure local domain name resolution to make things easier. The simplest way is to use the `/etc/hosts` file. Although this approach is not very scalable, it is easy to configure and illustrates the concept well. Let's configure IP address to domain name mapping for our Load Balancer (LB).

1. Open the `/etc/hosts` file on your LB server:

    ```bash
    sudo vi /etc/hosts
    ```

2. Add 2 records into this file with the Local IP address and an arbitrary name for both of your Web Servers:

    ```plaintext
    <WebServer1-Private-IP-Address> Web1
    <WebServer2-Private-IP-Address> Web2
    ```

3. Update your LB config file with those names instead of IP addresses:

    ```apache
    BalancerMember http://Web1:80 loadfactor=5 timeout=1
    BalancerMember http://Web2:80 loadfactor=5 timeout=1
    ```

4. You can try to curl your Web Servers from the LB locally:

    ```bash
    curl http://Web1
    curl http://Web2
    ```

    It should work as expected.
