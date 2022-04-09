# LOAD-BALANCER-SOLUTION-WITH-NGINX-AND-SSL-TLS
PROJECT 10: LOAD BALANCER SOLUTION WITH NGINX AND SSL/TLS

## Provision the environment
 1. Create an Ubuntu Server 20.04  
[Creating Ubuntu instance in AWS (**EC2**)](https://github.com/hectorproko/RepeatableSteps_tutorials/blob/main/AWS_Ubuntu_Instnace.md)

2. Open TCP port **80** and **443** creating an **Inbound Rule** in Security Group.  
[Open Ports in AWS (**EC2**)](https://github.com/hectorproko/RepeatableSteps_tutorials/blob/main/OpenPortAWS.md)


## Configure Nginx as a Load Balancer
``` bash
#Install Nginx
sudo apt update
sudo apt install nginx
```
Will update **/etc/hosts** file for local DNS with Web Servers’ names **Web1**, **Web2** and their **local IP addresses**. This is my hosts file

``` bash
ubuntu@ip-172-31-91-30:~$ sudo vi /etc/hosts
ubuntu@ip-172-31-91-30:~$ cat /etc/hosts
    127.0.0.1 localhost
    172.31.87.158 Web1 #<< Added these two lines
    172.31.81.144 Web2 #<<
    ...
```


We configure **LB** by updating config file **nginx.conf** 
```bash
sudo vi /etc/nginx/nginx.conf
```
Make sure to comment-out line **include /etc/nginx/sites-enabled/*;**
``` bash
#include /etc/nginx/sites-enabled/*;
```

Sections **upstream** and **server** should look like this
``` bash
upstream myproject {
    server Web1 weight=5; #using Local DNS names Web1,2 instead of IPs
    server Web2 weight=5;
  }
server {
    listen 80;
    server_name www.domain.com;
    location / {
      proxy_pass http://myproject;
    }
  }

}
```

## Register a new domain name and configure secured connection using SSL/TLS certificates
Let us make necessary configurations to make connections to our Tooling Web Solution secured!
In order to get a valid SSL certificate – you need to register a new domain name. I registered hectorsdomainforproject.de

Assign an Elastic IP to your Nginx LB server and associate your domain name with this Elastic IP. To avoid new public ip after restart
 ## elasticip.png  
 ## elasticip2.png 


Update A record in your registrar to point to Nginx LB using Elastic IP address
## image: mydomains.png



Configure Nginx to recognize your new domain name
Update your nginx.conf with server_name www.<your-domain-name.com> instead of server_name www.domain.com
```bash
server {
    server_name www.hectorsdomainforproject.de;
    location / {
      proxy_pass http://myproject;
    }
```
Make sure [snapd](https://snapcraft.io/snapd) service is active and running.
```
sudo systemctl status snapd
```

Install **certbot** and request for an **SSL/TLS** certificate

``` perl
ubuntu@ip-172-31-91-30:~$ sudo snap install --classic certbot
certbot 1.20.0 from Certbot Project (certbot-eff✓) installed
ubuntu@ip-172-31-91-30:~$
```

Request your certificate (just follow the certbot instructions – you will need to choose which domain you want your certificate to be issued for, domain name will be looked up from nginx.conf file so make sure you have updated it on step 4).
https://certbot.eff.org/lets-encrypt/ubuntufocal-nginx

```
sudo ln -s /snap/bin/certbot /usr/bin/certbot  (created symbolic link)
sudo certbot --nginx
```

Test secured access to your Web Solution by trying to reach https://<your-domain-name.com>
You shall be able to access your website by using HTTPS protocol (that uses TCP port 443) and see a padlock pictogram in your browser’s search string.
Click on the padlock icon and you can see the details of the certificate issued for your website.