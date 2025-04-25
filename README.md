## 📘 Dockerize WordPress with phpMyAdmin, NGROK, SSL, and GCP Deployment
Welcome to this beginner-friendly guide! Here you’ll learn how to dockerize a WordPress website, add phpMyAdmin, optionally expose your site securely using NGROK, enable local SSL, and deploy everything to Google Cloud Platform (GCP).

# Prerequisites

Docker and Docker Compose installed. (sudo apt update && sudo apt install docker.io) or [Install Docker Guide](https://docs.docker.com/get-docker/)
GCP account (https://cloud.google.com)
NGROK account (optional for secure local exposure) at [free tier account](https://dashboard.ngrok.com/signup)
A domain name (recommended for SSL setup)

Note: docker compose comes bundled with docker.

# 🗂️ Step 1: Project Setup
mkdir wordpress-docker-setup && cd wordpress-docker-setup

Create the following files:

1.  docker-compose.yml
  
2. .env

3. (Optional) ngrok.yml

then initialize git to track your project
git init
add all files
git add . (the. adds all files in that folder)
git commit -m "initial commit"

go to github and create an empty repository, then copy the repo url and connect it to your local project.
git remote add origin https://github.com/Chrimau/wordpress-docker/
git push -u origin main

# 🐳 Step 2: Docker Compose Configuration
open the docker-compose.yml using vs code or your nano editor (code docker-compose.yml or nano docker-compose.yml) and write:

version: '3.8'

services:
  wordpress:
    image: wordpress:latest
    ports:
      - "8000:80"
    environment:
      WORDPRESS_DB_HOST: db
      WORDPRESS_DB_USER: ${MYSQL_USER}
      WORDPRESS_DB_PASSWORD: ${MYSQL_PASSWORD}
      WORDPRESS_DB_NAME: ${MYSQL_DATABASE}
    volumes:
      - wordpress_data:/var/www/html
    depends_on:
      - db

  db:
    image: mysql:5.7
    restart: always
    environment:
      MYSQL_DATABASE: ${MYSQL_DATABASE}
      MYSQL_USER: ${MYSQL_USER}
      MYSQL_PASSWORD: ${MYSQL_PASSWORD}
      MYSQL_ROOT_PASSWORD: ${MYSQL_ROOT_PASSWORD}
    volumes:
      - db_data:/var/lib/mysql

  phpmyadmin:
    image: phpmyadmin/phpmyadmin
    restart: always
    ports:
      - "8080:80"
    environment:
      PMA_HOST: db
      MYSQL_ROOT_PASSWORD: ${MYSQL_ROOT_PASSWORD}

volumes:
  wordpress_data:
  db_data:

## 🔐 Step 3: Configure Environment Variables
<pre> ```env
MYSQL_DATABASE=wordpress
MYSQL_USER=wp_user
MYSQL_PASSWORD=secretpassword
MYSQL_ROOT_PASSWORD=rootpassword``` </pre>

# 🌐 Step 4: (Optional) Expose with NGROK
ngrok.yml

authtoken: YOUR_NGROK_AUTHTOKEN
tunnels:
  wordpress:
    proto: http
    addr: 8000
  phpmyadmin:
    proto: http
    addr: 8080

    Start NGROK:
    ngrok start --all --config=ngrok.yml

  # 🔒 Step 5: Enable Local SSL with mkcert (Optional)

Install mkcert and create certificates:
brew install mkcert
mkcert -install
mkcert localhost

Update docker-compose.yml to include a reverse proxy like Nginx with mounted certs or use Caddy/Traefik for easier SSL setup.

☁️ Step 6: Deploy to Google Cloud Platform (GCP)

1. Install gcloud CLI

Follow: https://cloud.google.com/sdk/docs/install

2. Authenticate and Create a Project

gcloud auth login
gcloud projects create wordpress-docker --set-as-default

3. Enable Required APIs

gcloud services enable compute.googleapis.com container.googleapis.com

4. Set Up GCP Deployment (use option A or B)

Option A: Compute Engine (VM)

scp -r . your-user@your-vm-ip:~/wordpress-docker-setup
ssh your-user@your-vm-ip
cd wordpress-docker-setup && docker-compose up -d

Option B: Google Kubernetes Engine (GKE)

1. Create a GKE cluster
   gcloud container clusters create wordpress-cluster --num-nodes=1 --zone=us-central1-a
   
2. Get cluster credentials:
   gcloud container clusters get-credentials wordpress-cluster --zone=us-central1-a

3. Push Docker image to Google Container Registry:
   docker tag wordpress gcr.io/your-project-id/wordpress
   docker push gcr.io/your-project-id/wordpress

4. Create Kubernetes deployment and service for WordPress:
  wordpress-deployment.yml

apiVersion: apps/v1
kind: Deployment
metadata:
  name: wordpress
spec:
  replicas: 1
  selector:
    matchLabels:
      app: wordpress
  template:
    metadata:
      labels:
        app: wordpress
    spec:
      containers:
      - name: wordpress
        image: gcr.io/YOUR_PROJECT_ID/wordpress
        ports:
        - containerPort: 80
        
 wordpress-service.yml

apiVersion: v1
kind: Service
metadata:
  name: wordpress
spec:
  type: LoadBalancer
  selector:
    app: wordpress
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80

5. Apply Kubernetes configurations:
   kubectl apply -f wordpress-deployment.yml
kubectl apply -f wordpress-service.yml

You’ll receive an external IP for accessing your site once the service is ready.

# ✅ Summary

You have just:

Dockerized WordPress and phpMyAdmin

Optionally exposed them with NGROK

Secured local access with mkcert SSL

Deployed to GCP via VM or GKE with Kubernetes YAML manifests

🎉 Great job! If you need help scaling, optimizing, or adding CI/CD, feel free to reach out.
