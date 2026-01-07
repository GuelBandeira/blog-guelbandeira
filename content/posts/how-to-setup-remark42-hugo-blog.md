---
title: "How to setup Remark42 for Hugo blog?"
date: 2025-12-09
draft: false
series: "Definitive Guide to Hosting a Blog"
tags: ["Hugo", "Remark42", "Oracle"]
---
# How to Install and Configure Remark42 on Oracle Cloud Using Docker + Nginx

If you have a Hugo blog and want to add comments, Remark42 is one of the best choices, because is free, different from Disqus that you need to pay, but the thing that makes Remark42 not so simple, is that it is self-hosted, so I created this guide to help you, telling how I did it, so don't have to go to the same trouble I had.

In this guide, I’ll show exactly how I installed Remark42 on an Oracle Cloud Always Free VPS using Docker, Nginx, and a domain with proper DNS configuration. I also included my errors that happened during the setup, and how I fixed them, if you encounter the same problem.

---

## What we will set up

- Oracle Cloud VPS (Ubuntu)
- DNS configuration (A record)
- SSH access using a generated key pair  
- Docker
- Remark42 container (HTTP mode)  
- Nginx reverse proxy  
- Oracle VCN/Firewall rules  
- Fixing common issues

---

## Setup Oracle VPS


### 1. Creating Your Free Instance 

In this guide I will use the Oracle UI on the web to configure our server that is going to host the Remark42, the first you need is to create an account on Oracle Cloud, and after that you will [create an instance](https://docs.oracle.com/en-us/iaas/Content/Compute/Tasks/launchinginstance.htm)

When selecting the image, I'm going to choose Ubuntu 24.04, but you can choose whatever you want, but note that this guide only cover Ubuntu.

![Selecting Ubuntu](/images/how-to-setup-remark42-hugo-blog/ubuntu-image.png)

![Selecting Ubuntu Version](/images/how-to-setup-remark42-hugo-blog/ubuntu-image2.png)

**IMPORTANT**: When selecting the shape, you can choose the 'Always Free' option, but this option isn't always available depending on which region you are, I am on AD1-SAOPAULO and in this region I get an **“out of host capacity”** error when I try to create an Always Free Compute instance, so I recommend you try creating the instance with VM.Standard.A1.Flex, and if you got the error above, use the **VM.Standard.A2.Flex**, and don't worry about not being free, it is going to remain free because we will not use the full capacity of this instance, we are going to use it with 4 CPUs and 24GB of RAM, that is the same resources the 'Always Free' instance has.

![Selecting Shape](/images/how-to-setup-remark42-hugo-blog/select-shape.png)

In the security section I have the **Shielded Instance** disabled.

![Security page](/images/how-to-setup-remark42-hugo-blog/security-page.png)


And in the Networking you can put whatever name you want in the VNIC, and be sure that this option is enabled.
![IPV4 option](/images/how-to-setup-remark42-hugo-blog/ipv4-assignment.png)


After that, download your SSH-Keys and save it in your computer, we are going to use this in the next step.

![SSH Keys download](/images/how-to-setup-remark42-hugo-blog/ssh-keys.png)


On the **Storage** section on can leave the way it is

![Storage Section](/images/how-to-setup-remark42-hugo-blog/storage-section.png)


Now your instance is created it! Now let's do one more step to ensure we stay on the free version, and we will configure your public IP, alogn with your access through SSH, as well as allowing the ports Remark42 is going to use on your VNIC.


### 2. Ensuring the instance stays on Free

Go to your instance, and stop the instance

![Stopping the Instance](/images/how-to-setup-remark42-hugo-blog/stopping-instance.png)

Now wait until the instance is completely stopped, and go on Actions > More actions > Edit

![Edit the Instance](/images/how-to-setup-remark42-hugo-blog/edit-instance.png)

Now edit the number of CPU's and the amount of memory, the Always Free has **4 CPUs and 24GB of RAM**, so you can enter the number until this limit, if you surpass that, you are gonna be charged based on the paid plan, so if you stay below this limit you are safe, you can even set the number of CPU to 1 and the amount of memory to 2GB like I did, and remark42 will work fine. 

![Configuring the amout of RAM and Storage of the Instance](/images/how-to-setup-remark42-hugo-blog/config-instance.png)

### 3. Create Public IP

This step will only be needed if you don't have a public IP assigned, like mine below: 

![Instance](/images/how-to-setup-remark42-hugo-blog/select-instance.png)


If that's your case, enter the instance, go to the networking.

![Instance Networking](/images/how-to-setup-remark42-hugo-blog/instance-networking.png)

And enter your primary VNIC

![Instance Networking](/images/how-to-setup-remark42-hugo-blog/select-vnic.png)

After that, go to **IP Administration**, click the three dots button on the right side of the page, and go on **Edit**.

You should see this section, Select the **Create new Reserved IP Address** and define the Public IP Name, after that click on the white update button.


![Instance](/images/how-to-setup-remark42-hugo-blog/assign-public-ipv4.png)

Now you have the IP that we are going to use to connect through SSH.

### 3. Connect with SSH

Grab the SSH keys you have downloaded in the first step, and follow this tutorial so you can see how you will generate your access

**IMPORTANT**: The username will **ubuntu**

{{< youtube mZiyH-ibyNM >}}

### 4. Allowing Remark42 ports on VNIC security list

Before we install Docker + Nginx, we need to ensure the ports that remark42 is going to use are defined in your VNIC security list, so let's go

Go to Networking > Enter your Primary VNIC

Install Docker:

```bash
curl -fsSL https://get.docker.com | sudo bash
```

Install Docker Compose plugin:

```bash
sudo apt install docker-compose-plugin -y
```

Check:

```bash
docker --version
docker compose version
```

---

## 4. Creating the Docker Compose for Remark42

Create a folder:

```bash
mkdir remark42
cd remark42
```

Create the `docker-compose.yml`:

```yaml
version: "3"

services:
  remark42:
    image: umputun/remark42:latest
    container_name: remark42
    restart: always
    ports:
      - "8080:8080"
    environment:
      - SITE=remark42.blogprogrammer.com
      - REMARK_URL=https://remark42.blogprogrammer.com
      - SECRET=your-secret-here
```

Start Remark42:

```bash
sudo docker compose up -d
```

Check logs:

```bash
sudo docker compose logs -f
```

---

## 5. Create DNS A Record

On your domain provider, add:

```
Type: A
Host: remark42
Value: YOUR_PUBLIC_IP
TTL: Auto
```

Check propagation:

```bash
dig +short remark42.blogprogrammer.com
```

It must return your VPS IP.

---

## 6. Fixing Oracle Cloud Firewall (VCN Rules)

This is the part that caused the main problems.

Initially:

- `curl http://SERVER_IP` failed  
- `nc -vz IP 80` timed out  
- Certbot failed  
- Nginx couldn't proxy  
- The server was unreachable externally

The reason: **Oracle blocks all ingress ports by default**.

Fix:

1. Go to **Oracle Console → Networking → VCN → Subnet → Security Lists**
2. Edit the security list applied to your instance
3. Add ingress rules:

```
Protocol: TCP
Port: 80
Source: 0.0.0.0/0
```

```
Protocol: TCP
Port: 443
Source: 0.0.0.0/0
```

Save and wait 1 minute.

Now test again:

```bash
curl -v http://YOUR_PUBLIC_IP
```

It should respond.

---

## 7. Installing and Configuring Nginx

Install:

```bash
sudo apt install nginx -y
```

Create the config file:

```bash
sudo nano /etc/nginx/sites-available/remark42.conf
```

Paste:

```nginx
server {
    listen 80;
    server_name remark42.blogprogrammer.com;

    location / {
        proxy_pass http://127.0.0.1:8080;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

Enable the config:

```bash
sudo ln -s /etc/nginx/sites-available/remark42.conf /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl restart nginx
```

Test:

```bash
curl -I http://127.0.0.1
curl -I http://YOUR_PUBLIC_IP
```

---

## 8. Generating SSL with Certbot

Install Certbot:

```bash
sudo apt install certbot python3-certbot-nginx -y
```

Run:

```bash
sudo certbot --nginx -d remark42.blogprogrammer.com
```

If you originally ran it before fixing Oracle Cloud firewall, you saw this:

```
Challenge failed for domain
Some challenges have failed
```

When the firewall is fixed, it works.

---

## 9. Final Verification

Open:

```
https://remark42.blogprogrammer.com/web
```

You should see the admin UI.

Check logs:

```bash
docker compose logs -f
```

---

## 10. Integration With Your Website

Add this script on your pages:

```html
<script>
  var remark_config = {
    host: "https://remark42.blogprogrammer.com",
    site_id: "remark42.blogprogrammer.com",
  }
</script>

<script src="https://remark42.blogprogrammer.com/web/embed.js"></script>
<link rel="stylesheet" href="https://remark42.blogprogrammer.com/web/remark.css">
```

Insert the container:

```html
<div id="remark42"></div>
```

---

## Conclusion

You now have:

- Remark42 running in Docker  
- Nginx reverse proxy  
- SSL via Certbot  
- Oracle Cloud firewall properly configured  
- A fully self-hosted comment system  

This setup is lightweight, fast, and completely under your control.
