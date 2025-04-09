DevOps Project: CI/CD Pipeline
Overview
This project demonstrates a complete CI/CD pipeline for deploying a full-stack application (frontend + backend) using Docker, Docker Compose, GitHub Actions, and Nginx as a reverse proxy. The application is containerized and deployed on an Ubuntu virtual machine (VM) in the cloud.

Prerequisites
Before you begin setting up this project, ensure you have the following:

A GitHub account to host the repository.

A Docker Hub account to store Docker images.

Access to a cloud platform (e.g., AWS, Azure) to set up an Ubuntu VM.

Basic knowledge of Docker, Docker Compose, Nginx, and CI/CD pipelines.

Setup Instructions
1. Repository Setup
Clone this repository to your local machine:

bash
Copy
Edit
git clone https://github.com/dineshvkumarv/DevOps-project.git
cd DevOps-project
Push any changes to the main branch to trigger the CI/CD pipeline.

2. Containerization & Deployment
Dockerfiles
Frontend Dockerfile: Builds the frontend application using Node.js.

Dockerfile
Copy
Edit
FROM node:18-alpine
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
EXPOSE 3000
CMD ["npm", "start", "--", "--host", "0.0.0.0"]
Backend Dockerfile: Builds the backend application using Node.js.

Dockerfile
Copy
Edit
FROM node:18-alpine
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
EXPOSE 8080
CMD ["npm", "start"]
Docker Compose
The docker-compose.yml file defines the services for the frontend, backend, and database:

yaml
Copy
Edit
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
3. Database Setup
Choose one of the following options:

Option 1: Install MongoDB directly on the Ubuntu VM.

4. CI/CD Pipeline Configuration
The GitHub Actions workflow (deploy.yml) automates the following steps:

Build Docker images for the frontend and backend.

Push the images to Docker Hub.

Deploy the application on the Ubuntu VM by pulling the latest images and restarting containers.

5. Nginx Reverse Proxy Setup
Set up Nginx to route traffic to the frontend and backend services:

Install Nginx on the Ubuntu VM:
bash
Copy
Edit
sudo apt update
sudo apt install -y nginx
Configure Nginx (/etc/nginx/sites-available/default):
nginx
Copy
Edit
events {}

http {
    upstream frontend {
        server frontend:3000;
    }

    upstream backend {
        server backend:5000;
    }

    server {
        listen 80;

        location /api/ {
            proxy_pass http://backend;
            proxy_http_version 1.1;
            proxy_set_header Upgrade $http_upgrade;
            proxy_set_header Connection 'upgrade';
            proxy_set_header Host $host;
            proxy_cache_bypass $http_upgrade;
        }

        location / {
            proxy_pass http://frontend;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_http_version 1.1;
            proxy_set_header Upgrade $http_upgrade;
            proxy_set_header Connection 'upgrade';
            proxy_cache_bypass $http_upgrade;
            try_files $uri $uri/ /index.html;
        }
    }
}
Screenshots
View the screenshots of the project setup here:
Project Screenshots

