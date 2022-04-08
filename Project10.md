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
In order to get a valid SSL certificate – you need to register a new domain name, you can do it using any Domain name registrar – a company that manages reservation of domain names. The most popular ones are: Godaddy.com, Domain.com, Bluehost.com.
	1. Register a new domain name with any registrar of your choice in any domain zone (e.g. .com, .net, .org, .edu, .info, .xyz or any other)
	2. Assign an Elastic IP to your Nginx LB server and associate your domain name with this Elastic IP
	
You might have noticed, that every time you restart or stop/start your EC2 instance – you get a new public IP address. When you want to associate your domain name – it is better to have a static IP address that does not change after reboot. Elastic IP is the solution for this problem, learn how to allocate an Elastic IP and associate it with an EC2 server on this page.
	3. Update A record in your registrar to point to Nginx LB using Elastic IP address
Learn how associate your domain name to your Elastic IP on this page.
