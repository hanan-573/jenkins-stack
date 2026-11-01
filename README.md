# Jenkins CI/CD Deployment with Docker & PostgreSQL

This repository contains the configuration to deploy Jenkins via Docker with persistent storage, alongside a PostgreSQL database. It includes a complete setup for a custom domain with Nginx and SSL.

## 🌐 Live Setup Information
* **Domain:** [https://jen.mughalkhan1.online](https://jen.mughalkhan1.online)
* **VPS IP:** `185.193.17.177`
* **Jenkins Port (Internal):** `8080`
* **PostgreSQL Port (Internal):** `5432`

---

## 🏗️ Architecture & Services
1. **Jenkins (LTS):** Main CI/CD server.
2. **PostgreSQL 15:** Database server.
3. **Nginx:** Acts as a reverse proxy to route traffic from the domain to the Jenkins container.
4. **Let's Encrypt (Certbot):** Provides SSL/HTTPS security.

---

## 📋 Prerequisites
Before deploying, ensure you have:
* A VPS (Ubuntu/Debian recommended).
* Docker and Docker Compose installed on the VPS.
* A domain (`jen.mughalkhan1.online`) with an **A Record** pointing to your VPS IP.
* Ports `80` and `443` open in the firewall.

---

## 🚀 Deployment Steps

### 1. Install Docker
```bash
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh
sudo apt install docker-compose-plugin -y




2. Setup Docker Compose
Create a directory for the project:

bash
mkdir -p ~/jenkins-stack
cd ~/jenkins-stack
Create a docker-compose.yml file:

yaml
version: '3.8'

services:
  jenkins:
    image: jenkins/jenkins:lts-jdk17
    container_name: jenkins
    restart: unless-stopped
    ports:
      - "127.0.0.1:8080:8080"   # Exposed locally, Nginx handles the public traffic
      - "50000:50000"           # Jenkins agent communication
    volumes:
      - jenkins_home:/var/jenkins_home
    environment:
      - JAVA_OPTS=-Djenkins.install.runSetupWizard=true
    networks:
      - jenkins-net

  postgres:
    image: postgres:15-alpine
    container_name: jenkins-postgres
    restart: unless-stopped
    environment:
      POSTGRES_DB: jenkinsdb
      POSTGRES_USER: jenkinsuser
      POSTGRES_PASSWORD: StrongPasswordChangeMe123!  # Change this in production!
    volumes:
      - postgres_data:/var/lib/postgresql/data
    networks:
      - jenkins-net

volumes:
  jenkins_home:
    driver: local
  postgres_data:
    driver: local

networks:
  jenkins-net:
    driver: bridge
3. Configure Nginx
Create a new Nginx configuration file:

bash
sudo nano /etc/nginx/sites-available/jenkins.mughalkhan1.online
Add the following HTTP configuration:

nginx
server {
    listen 80;
    server_name jen.mughalkhan1.online;

    location / {
        proxy_pass http://127.0.0.1:8080;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_http_version 1.1;
        proxy_request_buffering off;
        proxy_read_timeout 90s;
    }
}
Enable the site and reload Nginx:

bash
sudo ln -s /etc/nginx/sites-available/jenkins.mughalkhan1.online /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl reload nginx
4. Setup SSL (HTTPS)
Install Certbot and generate the certificate:

bash
sudo apt install certbot python3-certbot-nginx -y
sudo certbot --nginx -d jen.mughalkhan1.online
Select "Redirect HTTP to HTTPS" when prompted.

5. Start the Containers
Navigate to the project folder and start the services:

bash
cd ~/jenkins-stack
docker compose up -d
6. Initial Jenkins Setup
Retrieve the initial admin password from the Jenkins container:

bash
docker exec jenkins cat /var/jenkins_home/secrets/initialAdminPassword
Open https://jen.mughalkhan1.online in your browser.

Paste the password.

Install the suggested plugins.

Create your first Admin user.

🗄️ Connecting Jenkins to PostgreSQL
Go to Manage Jenkins -> Plugins.

Install the PostgreSQL Database plugin.

Restart Jenkins.

Go to Manage Jenkins -> System.

Scroll down to the Global Database section.

Configure the connection:

Database Type: PostgreSQL

Hostname: postgres
 
Port: 5432

Database Name: jenkinsdb

Username: jenkinsuser

Password: StrongPasswordChangeMe123! (or your custom password)

Click Test Connection to verify.

🛠️ Management Commands
Run these commands inside the ~/jenkins-stack directory:

Action	                                     Command
Start containers	                     docker compose up -d
Stop containers	                             docker compose down
Restart containers	                     docker compose restart
View status	                             docker compose ps
View Jenkins logs	                     docker compose logs -f jenkins
View Postgres logs	                     docker compose logs -f postgres
Delete everything (including data)	     docker compose down -v
Created for the Jenkins + PostgreSQL deployment on mughalkhan1.online
