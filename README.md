# TravelMemory-Deployment-Guide

# Introduction
This guide provides step-by-step instructions to deploy the TravelMemory application (MERN stack) on AWS EC2 instances with a load balancer and Cloudflare integration. The backend runs on Node.js with MongoDB, and the frontend is built using React.

# Prerequisites
AWS account with permissions to manage EC2, S3, and Load Balancers.
Domain name registered with a DNS provider (Cloudflare is recommended).
MongoDB Atlas or a self-hosted MongoDB instance.
Basic knowledge of SSH, Linux, and the MERN stack.

# Deployment Steps

# 1. Setting up Backend Servers

# 1.1 Launch EC2 Instances
Launch two Ubuntu 24.04 EC2 instances in the same region.
Configure security groups to allow:
Port 22 (SSH)
Port 3001 (Backend server)
Port 3000 (Frontend server)
Port 80 (HTTP for Nginx)

# 1.2 SSH into the Instances

bash

ssh -i "YourKey.pem" ubuntu@<InstancePublicIP>

# 1.3 Update and Install Dependencies

bash

sudo apt update && sudo apt upgrade -y
sudo apt install -y nodejs npm nginx git

# 1.4 Clone and Set Up Backend

bash

git clone https://github.com/UnpredictablePrashant/TravelMemory.git

cd TravelMemory/backend

npm install

# 1.5 Configure MongoDB

Add the connection string to .env:

bash

nano .env
Example .env file:

bash

MONGO_URI=mongodb+srv://<username>:<password>@cluster.mongodb.net/TravelMemory?retryWrites=true&w=majority

PORT=3001

Start the server:

bash

node index.js

# 1.6 Configure Nginx Reverse Proxy

Open the Nginx default configuration:

bash

sudo nano /etc/nginx/sites-available/default

Add the following:


server {
    listen 80;
    server_name <BackendPublicIP>;

    location / {
        proxy_pass http://localhost:3001;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
}

Restart Nginx:
bash

sudo nginx -t

sudo systemctl restart nginx

# 2. Setting up Frontend Servers

# 2.1 Launch Two More EC2 Instances

Follow steps in 1.1 for the frontend servers.

# 2.2 SSH into Instances and Install Dependencies

bash

ssh -i "YourKey.pem" ubuntu@<InstancePublicIP>

sudo apt update && sudo apt upgrade -y

sudo apt install -y nodejs npm nginx git

# 2.3 Clone and Build Frontend

bash

git clone https://github.com/UnpredictablePrashant/TravelMemory.git

cd TravelMemory/frontend

npm install



# 2.4 Update the urls.js File: Modify src/urls.js to point to the backend load balancer endpoint:


bash

export const baseUrl = process.env.REACT_APP_BACKEND_URL || "http://backend-load-balancer-dns:3000";

# 3. Configure Load Balancer

# 3.1 Create an Application Load Balancer

Navigate to the EC2 dashboard in AWS.
Go to Load Balancers → Create Load Balancer → Application Load Balancer.
Configure:
Add both backend instances to the backend target group.
Add both frontend instances to the frontend target group.

# 4. Cloudflare DNS Configuration

# 4.1 Add CNAME and A Records

Login to Cloudflare and navigate to the DNS settings.
Add:
CNAME: backend → <Backend Load Balancer DNS>
A Record: frontend → <Frontend Load Balancer DNS>

# 4.2 Enable Proxying

Toggle the proxy option in Cloudflare for the DNS records.

# 5. Verify and Test

Access the frontend via the custom domain.
Test the application's features and validate backend connectivity.
Check MongoDB for new entries.
