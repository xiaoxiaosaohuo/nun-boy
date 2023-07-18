---
title: 如何部署 Next.js 应用到 EC2

date: 2023-07-19T22:13:05.274Z
description: ""
image: "images/posts/aws.png"
categories: ["Tech"]
authors: ["siven"]
tags: ["Tech"]
draft: false
---

## Setup an EC2 instance

### 1. Connect to the instance

```
sudo chmod 400 keypair.pem (or keypair.cer)
ssh -i flywings.pem ec2-user@34.330.285.114


```

### 2. Install

```
sudo su
// install nvm
curl https://raw.githubusercontent.com/creationix/nvm/master/install.sh | bash
//Activate nvm
. ~/.nvm/nvm.sh
// install LTS (Gallium version)
nvm install v18

// install node
nvm install node

// install pm2 (Process Manager)
npm i -g pm2

npm install -g yarn

sudo yum update -y
sudo yum install git -y
git --version

```

### 3. Clone App

```
git clone xxxx.git
cd my-app
yarn
yarn run build
// copy .env file to my-app
//scp -i flywings.pem -r ./repos/order-app/.env ec2-user@34.330.285.114:/home/ec2-user/order-app

pm2 start yarn --name "order-app" -- start

```

### 4. Start Nginx server

```
server {
  listen 80;
  server_name nextjs.your-site.com;
  location / {
      proxy_pass http://127.0.0.1:3000;
      proxy_set_header Host $host;
      proxy_set_header X-Real-IP $remote_addr;
      proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
      proxy_set_header X-Forwarded-Proto $scheme;
    }
  }

  systemctl start nginx

  //restart
  systemctl restart nginx

```

### 5. Custom Domain

![hiking1](/nun-boy/images/posts/aws-route53.png)

### 6. Certificate
1. Request public Certificate
2. click Create Route53 records


### 7. Cloudfront

Create Cloudfront Distribution


1. Under ‘Origin Settings’, for ‘Origin Domain Name’, choose your EC2 instance or enter the Public IPv4 DNS
2. Under the ‘Origin Protocol Policy’, select ‘HTTP Only’. This ensures that CloudFront communicates with your EC2 instance using HTTP.
3. In the ‘Default Cache Behavior Settings’, set ‘Viewer Protocol Policy’ to ‘Redirect HTTP to HTTPS’. This forces users to connect to your CloudFront distribution with HTTPS for secure connections.
4. Select ‘GET, HEAD, OPTIONS, PUT, POST, PATCH, DELETE’. This allows CloudFront to cache responses to these HTTP methods.

5. In the same section (‘Default Cache Behavior Settings’), for ‘Cache Policy’ select ‘Managed-CachingDisabled’. For ‘Origin Request Policy’ select ‘Managed-AllViewer’. This effectively disables caching.

6. Under the ‘Distribution Settings’, for ‘SSL Certificate’, choose ‘Custom SSL Certificate (example.com)’. From the dropdown list, choose the certificate that matches your domain name (it should have been requested for the domain and created in AWS Certificate Manager in the N. Virginia region).
7. Still under the ‘Distribution Settings’, in the ‘Alternate Domain Names (CNAMEs)’ field, enter your domain name and its www variant, like this:
   domain.com www.domain.com

### 8. Update Route 53 Record Set

Once the distribution is created and deployed, copy the domain name of the distribution (it should look something like d111111abcdef8.cloudfront.net).

Go to Route 53 console, find the hosted zone for your domain, and create or update a record set to point your domain (‘domain.com’ and www.domain.com, to this CloudFront distribution.
Usually, I do create an ALIAS for the distribution

![hiking1](/nun-boy/images/posts/aws-route53-records.png)
