# Azure VM 3-Tier Application Deployment Guide

**Made by - Aditya Bahuguna**

---

## Table of Contents

1. [Prerequisites](#prerequisites)
2. [Architecture Overview](#architecture-overview)
3. [Step-by-Step Deployment](#step-by-step-deployment)
4. [Database Configuration](#database-configuration)
5. [DBeaver Connection Setup](#dbeaver-connection-setup)
6. [Testing & Verification](#testing--verification)
7. [Troubleshooting](#troubleshooting)

---

## Prerequisites

Before starting the deployment, ensure you have:

### Hardware Requirements
- Azure Virtual Machine (Ubuntu 24.04 LTS recommended)
- Minimum 2GB RAM
- Minimum 2 vCPU
- Minimum 20GB Storage

### Software Requirements
- Ubuntu 24.04 LTS or similar Linux distribution
- SSH access to Azure VM
- Terminal/Command line knowledge
- Docker & Docker Compose installed on VM
- Basic understanding of MySQL and Docker

### Access Requirements
- SSH key or password for Azure VM
- Admin/sudo privileges on VM
- Port access: 3306 (MySQL), 8081 (Backend API), 3000 (Frontend)

### Application Files
- Backend application source code with Dockerfile
- Frontend application source code with Dockerfile
- Database migration scripts (if any)

---

## Architecture Overview - (IP take as examples in this file )

```
┌─────────────────────────────────────────────────────┐
│              Azure Virtual Machine                   │
│  IP: 10.0.0.6 (Internal) | 135.235.194.44 (Public) │
├─────────────────────────────────────────────────────┤
│                                                       │
│  ┌──────────────┐    ┌──────────────┐              │
│  │   Frontend   │    │   Backend    │              │
│  │  Container   │    │  Container   │              │
│  │   (Port 3000)│    │  (Port 8081) │              │
│  └──────────────┘    └──────────────┘              │
│         ↓                    ↓                       │
│         └────────────┬───────┘                      │
│                      ↓                               │
│         ┌────────────────────────┐                 │
│         │  MySQL Database        │                 │
│         │  (Port 3306)           │                 │
│         │  Host Machine          │                 │
│         └────────────────────────┘                 │
│                                                       │
└─────────────────────────────────────────────────────┘
```

---

## Step-by-Step Deployment

### Step 1: Connect to Azure VM

```bash
# SSH into your Azure VM
ssh azureuser@135.235.194.44

# Or with key
ssh -i </path/to/key.pem> azureuser@135.235.194.44
```

---

### Step 2: Update System and Install Prerequisites

```bash
# Update package lists
sudo apt update && sudo apt upgrade -y

# Install required packages
sudo apt install -y curl wget git nano

# Install MySQL Server
sudo apt install -y mysql-server

# Verify MySQL installation
mysql --version
```

---

### Step 3: Install Docker and Docker Compose

```bash
# Install Docker using official script
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh
```

```bash
# Add current user to docker group
sudo usermod -aG docker $USER
newgrp docker
```

```bash
# Verify Docker installation
docker --version
docker-compose --version
```

```bash
# If Docker Compose is not installed
sudo curl -L "https://github.com/docker/compose/releases/latest/download/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
```

## OR you can install Docker and Docker Compose from the Docker installation doc: 
[[DOCKER INSTALLATION DOCUMANTATION] (https://docs.docker.com/engine/install/debian/)]

---

### Step 4: Configure MySQL for Remote Access

#### 4.1 Start and Enable MySQL

```bash
# Start MySQL service
sudo systemctl start mysql

# Enable MySQL to start on boot
sudo systemctl enable mysql

# Check MySQL status
sudo systemctl status mysql

# MySQL secure setup
sudo mysql_secure_installation
```

#### 4.2 Set Up MySQL Root User Authentication

```bash
# Access MySQL with sudo (no password required initially)
sudo mysql

# In MySQL prompt, run these commands:
```

```sql
-- Enable password-based authentication for root
ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'Yourpass@123';
FLUSH PRIVILEGES;
EXIT;
```

#### 4.3 Create Database and Application User

```bash
# Login to MySQL with password
mysql -u root -pYourpass@123

```

# In MySQL prompt, run these commands after enter:
```sql
-- Create database
CREATE DATABASE IF NOT EXISTS your_db;

-- Create application user (can connect from any host)
CREATE USER IF NOT EXISTS 'yourdb_user'@'%' IDENTIFIED WITH mysql_native_password BY 'ourpass@123';

-- Grant privileges to application user
GRANT ALL PRIVILEGES ON your_db.* TO 'yourdb_user'@'%';

-- Flush privileges to apply changes
FLUSH PRIVILEGES;

-- Verify user creation
SELECT user, host FROM mysql.user WHERE user='yourdb_user';

-- Exit MySQL
EXIT;
```

#### 4.4 Configure MySQL for Remote Connections

```bash
# Edit MySQL configuration file
sudo nano /etc/mysql/mysql.conf.d/mysqld.cnf

# Find this line:
# bind-address = 127.0.0.1

# Change it to:
bind-address = 0.0.0.0

# Save and exit (Ctrl+X, then Y, then Enter)

# Restart MySQL to apply changes
sudo systemctl restart mysql

# Verify configuration
sudo systemctl status mysql
```

#### 4.5 Configure Firewall inside the VM or Open port from the VM's Network settings port-3306 

```bash
# Allow MySQL port through firewall
sudo ufw allow 3306/tcp

# Verify firewall rules
sudo ufw status

# If needed, enable firewall
sudo ufw enable
```

---

### Step 5: Prepare Application Directories

```bash
# Create project directory structure
mkdir -p ~/prod/backend
mkdir -p ~/prod/frontend

# Copy your backend application to ~/prod/backend
# Copy your frontend application to ~/prod/frontend

# Verify structure
ls -la ~/prod/
```

---

### Step 6: Configure Backend Application

#### 6.1 Create/Update Backend app.env File

```bash
# Navigate to backend directory
cd ~/prod/backend

# Create or edit app.env file
nano app.env
```
# or
```bash
# Navigate to backend directory
cd ~/prod/backend

vi app.env 
``` 

# Add/Update the following content:
```

```env
# Database Configuration - HOST MACHINE (172.17.0.1 is Docker bridge IP to host)
DATABASE_HOST_STRING=172.17.0.1:3306
DATABASE_NAME=your_db
DB_USER_NAME=yourdb_user
DB_PASSWORD=Yourpass@123
```

#### 6.2 Create Backend docker-compose.yml

```bash
# Create docker-compose.yml in backend directory
nano docker-compose.yml

# Add the following content:
```

```yaml
services:
  your-backend-service:
    container_name: your-backend-service
    image: your-backend-service
    build:
      context: .
      dockerfile: Dockerfile
    ports:
      - "8081:8081"
    env_file:
      - ./app.env
    networks:
      - your-network-name
    restart: always

networks:
  your-network-name:
    driver: bridge
```

#### 6.3 Build and Deploy Backend

```bash
# Navigate to backend directory
cd ~/prod/backend

# Build Docker image and start it 
docker compose up --build -d  

# Check container status
docker ps | grep your-backend-service

# View logs
docker-compose logs -f your-backend-service
```

---

### Step 7: Configure Frontend Application

#### 7.1 Create/Update Frontend .env File

```bash
# Navigate to frontend directory
cd ~/prod/frontend

# Create or edit .env file
nano .env

# Add the following content:
```

```env
VITE_API_BASE_URL=http://135.235.194.44:8081/backend/api/v1/
```

**Note:** Replace `135.235.194.44` with your actual public IP address.

#### 7.2 Create Frontend docker-compose.yml

```bash
# Create docker-compose.yml in frontend directory
nano docker-compose.yml

# Add the following content:
```

```yaml
services:
  your-frontend-service:
    build:
      context: .
    container_name: your-frontend-service
    ports:
      - "3000:3000"
    networks:
      - your-frontend-service-network
    restart: always

networks:
  your-frontend-service-network:
    driver: bridge
```

#### 7.3 Build and Deploy Frontend

```bash
# Navigate to frontend directory
cd ~/prod/frontend

# Build Docker image and start container
docker-compose up --build  -d

# Check container status
docker ps | grep your-frontend-service

# View logs
docker-compose logs -f your-frontend-service
```

---

### Step 8: Verify Deployment

```bash
# Check all running containers
docker ps

# Check Docker networks
docker network ls

# Verify backend is running
curl -i http://135.235.194.44:8081/backend/api/v1/health

# Check MySQL connection from backend container
docker exec your-backend-service mysql -h 172.17.0.1 -u yourdb_user -p your_db -e "SELECT 1;"
# Password: Yourpass@123
```

---

## Database Configuration

### Database Details

| Property | Value |
|----------|-------|
| **Host** | 172.17.0.1 (for Docker containers) |
| **Host** | 10.0.0.6 (for internal network) |
| **Host** | 135.235.194.44 (for external access) |
| **Port** | 3306 |
| **Database** | your_db |
| **Username** | yourdb_user |
| **Password** | Yourpass@123 |

### Test Database Connection

```bash
# From VM command line
mysql -u yourdb_user -p -h 172.17.0.1 your_db

# Run a test query
SELECT 1;
EXIT;
```

---

## DBeaver Connection Setup

### Step 1: Open DBeaver

1. Launch DBeaver on your local machine
2. Go to **Database → New Database Connection**
3. Select **MySQL** and click **Next**

### Step 2: Connection Settings

Fill in the following details:

```
Server Host:        135.235.194.44
Port:              3306
Database:          your_db
Username:          yourdb_user
Password:          Yourpass@123
Save password:     ✓ (Check if you want to save)
```

### Step 3: Connection String Format

```
jdbc:mysql://135.235.194.44:3306/your_db?useSSL=false&serverTimezone=UTC
```

### Step 4: DBeaver Configuration Details

**Connection Properties Tab:**

```
Server Host:        135.235.194.44
Port:              3306
Database:          your_db
Username:          yourdb_user
Password:          Yourpass@123
SSL Mode:          DISABLED (for initial setup)
Connection Name:   Your DB Connection
```

**Advanced Settings (Optional):**

| Property | Value |
|----------|-------|
| serverTimezone | UTC |
| useSSL | false |
| allowPublicKeyRetrieval | true |
| characterEncoding | UTF-8 |

### Step 5: Test Connection

1. Click **Test Connection** button
2. If prompted, install missing drivers (select **Download**)
3. You should see: **"Connection successful"**

### Step 6: Save Connection

1. Click **Finish**
2. Connection will appear under **Connections** in the left panel
3. Expand to view databases and tables

### Complete DBeaver Connection String for Copy-Paste

```
Server Host: 135.235.194.44
Port: 3306
Database: your_db
Username: yourdb_user
Password: Yourpass@123
Connection URL: jdbc:mysql://135.235.194.44:3306/your_db?useSSL=false&serverTimezone=UTC&allowPublicKeyRetrieval=true&characterEncoding=UTF-8
```

---

## Testing & Verification

### 1. Backend API Testing

```bash
# Test backend health endpoint
curl -i http://135.235.194.44:8081/cynide/api/v1/health

# Or open in browser:
# http://135.235.194.44:8081/cynide/api/v1/health

# Expected response: 200 OK with health status
```

### 2. Frontend Testing

```bash
# Open in browser:
# http://135.235.194.44:3000

# Check browser console (F12) for any errors
```

### 3. Database Connection Testing

```bash
# From VM
mysql -u yourdb_user -p -h localhost your_db
# Password: Yourpass@123

# Or from Docker container
docker exec your-backend-service mysql -h 172.17.0.1 -u yourdb_user -p your_db -e "SHOW TABLES;"
# Password: Yourpass@123
```

### 4. Container Logs

```bash
# Backend logs
docker logs your-backend-service

# Frontend logs
docker logs your-frontend-service

# Real-time logs
docker-compose logs -f

# Specific service logs
cd ~/prod/backend
docker-compose logs -f your-backend-service
```

---

## Troubleshooting

### Issue 1: Backend Cannot Connect to MySQL

**Symptoms:** Backend container logs show "Connection refused" or "Access denied"

**Solutions:**

```bash
# Check MySQL is running
sudo systemctl status mysql

# Restart MySQL
sudo systemctl restart mysql

# Test MySQL from container
docker exec your-backend-service mysql -h 172.17.0.1 -u yourdb_user -p your_db -e "SELECT 1;"

# Verify credentials in app.env
cat ~/prod/backend/app.env | grep DB_

# Check firewall
sudo ufw status
sudo ufw allow 3306/tcp
```

### Issue 2: Frontend Cannot Reach Backend

**Symptoms:** Browser console shows CORS errors or connection refused

**Solutions:**

```bash
# Verify backend is running
curl -i http://135.235.194.44:8081/cynide/api/v1/health

# Check frontend .env has correct backend URL
cat ~/prod/frontend/.env

# Check firewall rules
sudo ufw status
sudo ufw allow 8081/tcp
sudo ufw allow 3000/tcp

# Verify containers are on the same network
docker network inspect cynide-network
docker network inspect cynoresource-new-network
```

### Issue 3: Docker Container Port Already in Use

**Symptoms:** Error "Address already in use"

**Solutions:**

```bash
# Check what's using the port
sudo lsof -i :3000
sudo lsof -i :8081
sudo lsof -i :3306

# Kill process using the port
sudo kill -9 <PID>

# Or stop existing containers
docker stop your-backend-service
docker stop your-frontend-service

# Restart containers
cd ~/prod/backend && docker-compose up -d
cd ~/prod/frontend && docker-compose up -d
```

### Issue 4: MySQL Connection Password Issues

**Symptoms:** "Access denied for user 'root'@'localhost'"

**Solutions:**

```bash
# Access MySQL with sudo
sudo mysql

# Reset root user password
ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'Yourpass@123';
FLUSH PRIVILEGES;
EXIT;

# Test with new password
mysql -u root -pYourpass@123

# Or without password via sudo
sudo mysql -u root
```

### Issue 5: DBeaver Cannot Connect

**Symptoms:** DBeaver shows "Connection refused" or timeout

**Solutions:**

```bash
# Verify MySQL is accepting remote connections
sudo nano /etc/mysql/mysql.conf.d/mysqld.cnf
# Ensure: bind-address = 0.0.0.0

# Restart MySQL
sudo systemctl restart mysql

# Test connection from VM
mysql -h 135.235.194.44 -u yourdb_user -p your_db

# Check Azure firewall rules allow port 3306 and 8081
```

---

## Useful Commands

### Container Management

```bash
# View all containers
docker ps -a

# Stop a container
docker stop <container_name>

# Start a container
docker start <container_name>

# Remove a container
docker rm <container_name>

# View container logs
docker logs <container_name>

# Execute command in container
docker exec -it <container_name> bash
```

### Docker Compose Commands

```bash
# Build images
docker compose build

# Start services
docker compose up -d

# Stop services
docker compose down

# View logs
docker compose logs -f

# Rebuild and restart
docker compose down
docker compose up -d --build
```

### MySQL Commands

```bash
# Login to MySQL
mysql -u yourdb_user -p -h 172.17.0.1 your_db

# Common queries
SHOW DATABASES;
USE your_db;
SHOW TABLES;
DESC <table_name>;
SELECT * FROM <table_name> LIMIT 10;

# Create new user
CREATE USER 'username'@'%' IDENTIFIED BY 'password';
GRANT ALL PRIVILEGES ON your_db.* TO 'username'@'%';
FLUSH PRIVILEGES;

# Backup database
mysqldump -u yourdb_user -p your_db > backup.sql

# Restore database
mysql -u yourdb_user -p your_db < backup.sql
```

### VM Commands

```bash
# Check system resources
free -h
df -h
top
```
```bash
# Check IP addresses
hostname -I
ifconfig
```

```bash
# Check running services
systemctl status mysql
systemctl status docker
```

```bash
# View firewall rules
sudo ufw status
sudo ufw allow <port>/tcp
```

---

## Important IPs and Ports Reference

| Component | IP Address | Port | Purpose |
|-----------|-----------|------|---------|
| **Frontend** | 135.235.194.44 | 3000 | User Interface |
| **Backend API** | 135.235.194.44 | 8081 | API Endpoints |
| **MySQL** | 172.17.0.1 | 3306 | Database (Docker) |
| **MySQL** | 10.0.0.6 | 3306 | Database (Internal) |
| **MySQL** | 135.235.194.44 | 3306 | Database (External) |
| **VM Internal** | 10.0.0.6 | - | Internal Network |
| **Docker Bridge** | 172.17.0.1 | - | Container to Host |

---

## Security Recommendations

1. **Change Default Passwords:** Replace `Yourpass@123` with strong passwords
2. **Use SSL/TLS:** Enable SSL for database and API connections
3. **Firewall Rules:** Restrict access to only necessary ports
4. **Regular Backups:** Schedule daily database backups
5. **Monitor Logs:** Regularly check application and MySQL logs
6. **Update System:** Keep OS and packages updated
7. **SSH Keys:** Use SSH keys instead of passwords for VM access
8. **Environment Variables:** Keep sensitive data in .env files, not in code

---

## Maintenance and Monitoring

### Daily Tasks

```bash
# Check container status
docker ps
```

```bash
# Check logs for errors
docker logs your-backend-service | grep -i error
docker logs your-frontend-service | grep -i error
```

```bash
# Check disk space
df -h
```

```bash
# Check memory usage
free -h
```

### Weekly Tasks

```bash
# Database backup
mysqldump -u yourdb_user -p your_db > ~/backups/your_db_$(date +%Y%m%d).sql
```

```bash
# Check system updates
sudo apt update
sudo apt list --upgradable
```

```bash
# Verify all services are running
sudo systemctl status mysql
docker ps
```

### Monthly Tasks

```bash
# Update system packages
sudo apt update && sudo apt upgrade -y
```

```bash
# Clean Docker images and containers
docker image prune
docker container prune
```

```bash
# Review and optimize database
mysql -u yourdb_user -p your_db -e "ANALYZE TABLE <table_name>;"
```

---

## Contact and Support

**Deployment Guide Created by:** Aditya Bahuguna

For issues or questions regarding this deployment:
1. Check the Troubleshooting section
2. Review logs using docker logs command
3. Verify all configuration files have correct values
4. Ensure all prerequisites are installed and running

---

## Version History

| Version | Date | Changes |
|---------|------|---------|
| 1.0 | Oct 09, 2025 | Initial deployment guide |

---

**Last Updated:** October 09, 2025

**Made by - Aditya Bahuguna**




Azure VM deployment documentation for Any Resource Management System. Deploy a 3-tier application (Frontend, Backend API, MySQL) using Docker containers with complete configuration guides and troubleshooting steps.
