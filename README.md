📦 Overview
This project demonstrates a complete CI/CD pipeline for deploying a full-stack application (frontend + backend) using Docker, Docker Compose, GitHub Actions, and Nginx as a reverse proxy. The application is containerized and deployed on an Ubuntu virtual machine (VM) in the cloud.

🔧 Prerequisites
Before setting up the project, ensure you have the following:

- A GitHub account to host the repository.
- A Docker Hub account to store Docker images.
- Access to a cloud platform (e.g., AWS, Azure) to set up an Ubuntu VM.
- Basic knowledge of Docker, Docker Compose, Nginx, and CI/CD pipelines.

🛠 Setup Instructions
### 1. Repository Setup
Clone this repository to your local machine:
```bash
git clone https://github.com/dineshvkumarv/DevOps-project.git
cd DevOps-project
```

Push any changes to the main branch to trigger the CI/CD pipeline.

---

### 2. Containerization & Deployment
#### Dockerfiles
- **Frontend Dockerfile**: Builds the frontend application using Node.js.
```dockerfile
FROM node:18-alpine
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
EXPOSE 3000
CMD ["npm", "start", "--", "--host", "0.0.0.0"]
```

- **Backend Dockerfile**: Builds the backend application using Node.js.
```dockerfile
FROM node:18-alpine
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
EXPOSE 8080
CMD ["npm", "start"]
```

#### Docker Compose
The `docker-compose.yml` file defines the services for the frontend, backend, and database:
```yaml
version: '3.8'

services:
  mongo:
    image: mongo
    restart: always
    volumes:
      - mongo-data:/data/db
    networks:
      - app-network

  backend:
    build: ./backend
    ports:
      - "5000:5000"
    env_file:
      - ./backend/.env
    depends_on:
      - mongo
    networks:
      - app-network

  frontend:
    build: ./frontend
    ports:
      - "3000:3000"
    command: ["npm", "start", "--", "--host", "0.0.0.0"]
    networks:
      - app-network

  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
    volumes:
      - ./nginx/nginx.conf:/etc/nginx/nginx.conf:ro
    depends_on:
      - backend
      - frontend
    networks:
      - app-network

volumes:
  mongo-data:

networks:
  app-network:
    driver: bridge
```

---

### 3. Database Setup
Choose one of the following options:

**Option 1**: Install MongoDB directly on the Ubuntu VM.

---

### 4. CI/CD Pipeline Configuration
The GitHub Actions workflow (`deploy.yml`) automates the following steps:
- Builds Docker images for the frontend and backend.
- Pushes the images to Docker Hub.
- Deploys the application on the Ubuntu VM by pulling the latest images and restarting containers.

---

### 5. Nginx Reverse Proxy
Set up Nginx to route traffic to the frontend and backend services:

- **Install Nginx on the Ubuntu VM**:
  ```bash
  sudo apt update
  sudo apt install -y nginx
  ```

- **Configure Nginx** (`/etc/nginx/sites-available/default`):
  ```nginx
  events {}

  http {
      server {
          listen 80;

          location / {
              proxy_pass http://frontend:3000;
              proxy_set_header Host $host;
              proxy_set_header X-Real-IP $remote_addr;
              proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
          }

          location /api/ {
              proxy_pass http://backend:8080;
              proxy_set_header Host $host;
              proxy_set_header X-Real-IP $remote_addr;
              proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
          }
      }
  }
  ```

---

### 🔑 Additional Commands

#### Docker Commands
- **Build Docker Images**:
  ```bash
  docker build -t <username>/dine-backend:latest ./dine/backend
  docker build -t <username>/dine-frontend:latest ./dine/frontend
  ```

- **Run Docker Containers**:
  ```bash
  docker run -d -p 3000:3000 <username>/dine-frontend:latest
  docker run -d -p 5000:5000 <username>/dine-backend:latest
  ```

- **Stop All Containers**:
  ```bash
  docker stop $(docker ps -q)
  ```

- **Remove Unused Images**:
  ```bash
  docker image prune -f
  ```

#### Docker Compose Commands
- **Start All Services in Detached Mode**:
  ```bash
  docker-compose up -d
  ```

- **Stop and Remove All Services**:
  ```bash
  docker-compose down
  ```

- **View Logs for All Services**:
  ```bash
  docker-compose logs -f
  ```

#### Debugging Commands
- **List Running Containers**:
  ```bash
  docker ps
  ```

- **View Logs of a Container**:
  ```bash
  docker logs <container_id>
  ```

- **Access a Running Container**:
  ```bash
  docker exec -it <container_id> /bin/sh
  ```

---

### 6. Deployment Verification
- Access the application in your web browser using the public IP or domain of your server.
- Verify that the frontend and backend are properly routed via Nginx.

---

This README now includes additional commands to help you better manage and debug your project. Let me know if you need further assistance!
