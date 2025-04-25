## 📘 Dockerize WordPress with phpMyAdmin, NGROK, SSL, and GCP Deployment
Welcome to this beginner-friendly guide! Here you’ll learn how to dockerize a WordPress website, add phpMyAdmin, optionally expose your site securely using NGROK, enable local SSL, and deploy everything to Google Cloud Platform (GCP).

# Prerequisites

1. Docker and Docker Compose installed. (sudo apt update && sudo apt install docker.io) or [Install Docker Guide](https://docs.docker.com/get-docker/)
2. GCP account (https://cloud.google.com)
3. NGROK account (optional for secure local exposure) at [free tier account](https://dashboard.ngrok.com/signup)
4. A domain name (recommended for SSL setup)

Note: docker compose comes bundled with docker.

# 🗂️ Step 1: Project Setup
<pre> bash
  
  mkdir wordpress-docker-setup && cd wordpress-docker-setup </pre>

Create the following files:

1.  docker-compose.yml
  
2. .env

3. (Optional) ngrok.yml

I recommend initializing git to track your project
<pre> bash
  
git init
git add .    #the . adds all files in that folder
git commit -m "initial commit" </pre>

go to github and create an empty repository, then copy the repo url and connect it to your local project.
<pre> bash

git remote add origin https://github.com/Chrimau/wordpress-docker/
git push -u origin main </pre>

# 🐳 Step 2: Docker Compose Configuration
open the docker-compose.yml using vs code or your nano editor (code docker-compose.yml or nano docker-compose.yml), its not necessary to declare version if using docker 27.01 or later:

<pre> yaml
  
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
  db_data: </pre>

## 🔐 Step 3: Configure Environment Variables
<pre> env
  
MYSQL_DATABASE=wordpress
MYSQL_USER=wp_user
MYSQL_PASSWORD=secretpassword
MYSQL_ROOT_PASSWORD=rootpassword``` </pre>

# 🌐 Step 4: (Optional) Expose with NGROK
ngrok.yml

<pre> yaml
  
  authtoken: YOUR_NGROK_AUTHTOKEN
tunnels:
  wordpress:
    proto: http
    addr: 8000
  phpmyadmin:
    proto: http
    addr: 8080 </pre>

  Start NGROK:
    
   <pre> yaml
     
     ngrok start --all --config=ngrok.yml </pre>
     
  # 🔒 Step 5: Enable Local SSL with mkcert (Optional)

Install mkcert and create certificates
<pre> bash
  
brew install mkcert
mkcert -install
mkcert localhost </pre>

Update docker-compose.yml to include a reverse proxy like Nginx with mounted certs or use Caddy/Traefik for easier SSL setup.

# ☁️ Step 6: Deploy to Google Cloud Platform (GCP)
Make sure Docker is running, then build and start the services:
<pre> bash
  
  docker-compose up -d --build </pre>

  To stop everything later:
<pre> bash
  
docker-compose down </pre>

# ☁️ Step 7: Deploy to Google Cloud Platform (GCP)
1. Install gcloud CLI

Follow: https://cloud.google.com/sdk/docs/install

2. Authenticate and Create a Project

<pre> bash
  
  gcloud auth login
gcloud projects create wordpress-docker --set-as-default </pre>

3. Enable Required APIs

<pre> bash
  
  gcloud services enable compute.googleapis.com container.googleapis.com </pre>

4. Set Up GCP Deployment (use option A or B)

Option A: Compute Engine (VM)

<pre> bash
  
  scp -r . your-user@your-vm-ip:~/wordpress-docker-setup
ssh your-user@your-vm-ip
cd wordpress-docker-setup && docker-compose up -d </pre>

Option B: Google Kubernetes Engine (GKE)

1. Create a GKE cluster
  <pre< bash

 gcloud container clusters create wordpress-cluster --num-nodes=1 --zone=us-central1-a </pre>
   
3. Get cluster credentials:
   <pre> bash 
     
     gcloud container clusters get-credentials wordpress-cluster --zone=us-central1-a </pre>

4. Push Docker image to Google Container Registry:
  <pre> bash
    
    docker tag wordpress gcr.io/your-project-id/wordpress
   docker push gcr.io/your-project-id/wordpress </pre>

5. Create Kubernetes deployment and service for WordPress:
  wordpress-deployment.yml

<pre> yaml
  
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
        - containerPort: 80 </pre>
        
 wordpress-service.yml

<pre> yaml
  
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
      targetPort: 80 </pre>

5. Apply Kubernetes configurations:
   <pre> bash
   
  kubectl apply -f wordpress-deployment.yml
  kubectl apply -f wordpress-service.yml </pre>

You will receive an external IP for accessing your site once the service is ready.

# ✅ Summary

If you got here safely, then you have just:

Dockerized WordPress and phpMyAdmin

Optionally exposed them with NGROK

Secured local access with mkcert SSL

Deployed to GCP via VM or GKE with Kubernetes YAML manifests

🎉 Great job really! If you need help scaling, optimizing, or adding CI/CD, feel free to reach out.
