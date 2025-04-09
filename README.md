DevOps Project: CI/CD Pipeline
Overview
This project demonstrates how to set up a complete CI/CD pipeline to deploy a full-stack application (frontend + backend) using Docker, Docker Compose, GitHub Actions, and Nginx as a reverse proxy. The entire application is containerized and deployed on an Ubuntu VM in the cloud.

Prerequisites
Before getting started, make sure you have the following:

A GitHub account to host your repository.

A Docker Hub account to store your Docker images.

Access to a cloud platform (like AWS or Azure) to set up an Ubuntu VM.

Basic knowledge of Docker, Docker Compose, Nginx, and CI/CD pipelines.

Setup Instructions
1. Clone the Repository
Start by cloning this repository to your local machine:

bash
Copy
Edit
git clone https://github.com/dineshvkumarv/DevOps-project.git
cd DevOps-project
Once cloned, you can push any changes to the main branch to trigger the CI/CD pipeline.

2. Containerization & Deployment
Dockerfiles
Frontend Dockerfile (for building the frontend with Node.js):

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
Backend Dockerfile (for building the backend with Node.js):

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
Here’s the docker-compose.yml that defines the services for the frontend, backend, and database:

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
You have two options here for setting up the database:

Option 1: Install MongoDB directly on the Ubuntu VM.

4. CI/CD Pipeline Configuration
The CI/CD pipeline is handled by GitHub Actions. The workflow (defined in deploy.yml) automates these steps:

Builds Docker images for both the frontend and backend.

Pushes the images to Docker Hub.

Deploys the latest version of the application to the Ubuntu VM by pulling the updated images and restarting the containers.

5. Nginx Reverse Proxy Setup
Nginx will route traffic to the frontend and backend services. Follow these steps to set it up:

Install Nginx on your Ubuntu VM:
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
You can find the project screenshots here:
Project Screenshots
