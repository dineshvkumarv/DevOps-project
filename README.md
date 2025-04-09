# DevOps Project: Full-Stack CI/CD Pipeline Deployment 🚀

This project demonstrates a complete CI/CD pipeline for deploying a full-stack application (frontend + backend) using Docker, Docker Compose, GitHub Actions, and Nginx as a reverse proxy. The application is containerized and deployed on an Ubuntu virtual machine (VM) in the cloud.

## 🔧 Prerequisites

Before setting up the project, ensure you have the following:

- A GitHub account to host the repository.
- A Docker Hub account to store Docker images.
- Access to a cloud platform (e.g., AWS, Azure) to set up an Ubuntu VM.
- Basic knowledge of Docker, Docker Compose, Nginx, and CI/CD pipelines.

---

## 🛠 Setup Instructions

### 1. Repository Setup

Clone this repository to your local machine and navigate to the project directory:

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

#### Docker Compose Configuration

The `docker-compose.yml` file defines the services for the frontend, backend, database, and Nginx:

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

- **Option 1**: Use the MongoDB container defined in the `docker-compose.yml`.
- **Option 2**: Install MongoDB directly on the Ubuntu VM:
  ```bash
  sudo apt update
  sudo apt install -y mongodb
  ```

---

### 4. CI/CD Pipeline Configuration

The GitHub Actions workflow (`deploy.yml`) automates the following steps:

1. Build Docker images for the frontend and backend.
2. Push the images to Docker Hub.
3. Deploy the application on the Ubuntu VM by pulling the latest images and restarting containers.

---

### 5. Nginx Reverse Proxy

#### Install Nginx on the Ubuntu VM:

```bash
sudo apt update
sudo apt install -y nginx
```

#### Configure Nginx:

Update the Nginx configuration file (`/etc/nginx/nginx.conf`) to route traffic to the frontend and backend services.

---

### 6. Deployment Commands

Use the following Docker Compose commands to manage the deployment:

- Stop and remove containers:
  ```bash
  docker compose down
  ```

- Build and start containers in detached mode:
  ```bash
  docker compose up --build -d
  ```

---

## 📸 Screenshots

View screenshots of the deployment process and application [here](https://drive.google.com/drive/folders/101KAsRzAalgH7KdRuTinzoJ6CcF4hpdP?usp=sharing).

---

## 🌟 Features

- **Frontend**: Built using Node.js, served on port 3000.
- **Backend**: Built using Node.js, served on port 8080.
- **Database**: MongoDB containerized for data persistence.
- **Reverse Proxy**: Nginx routes traffic to the frontend and backend services.
- **CI/CD Pipeline**: Automated with GitHub Actions.

---

## 📬 Contact

For inquiries or suggestions, feel free to reach out:

- **Author**: Dinesh V Kumar
- **GitHub**: [dineshvkumarv](https://github.com/dineshvkumarv)
