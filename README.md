# Nginx Reverse Proxy with HTTPS using Docker and Docker Compose

This repository demonstrates how to create a simple web application with a Node.js backend, dockerize it to run multiple instances, and set up Nginx as a reverse proxy with HTTPS support.

## Table of Contents

- [Nginx Reverse Proxy with HTTPS using Docker and Docker Compose](#nginx-reverse-proxy-with-https-using-docker-and-docker-compose)
  - [Table of Contents](#table-of-contents)
  - [Introduction](#introduction)
  - [Architecture](#architecture)
  - [Prerequisites](#prerequisites)
  - [Installation](#installation)
  - [Usage](#usage)
  - [Configuration Details](#configuration-details)
    - [Node.js Application](#nodejs-application)
    - [Docker Setup](#docker-setup)
    - [Nginx Configuration](#nginx-configuration)
  - [Customization](#customization)
  - [License](#license)

## Introduction

The goal of this project is to demonstrate how to:

1. Create a simple web application with a Node.js backend.
2. Dockerize the Node.js application and run multiple instances using Docker Compose.
3. Install Nginx and configure it as a reverse proxy to distribute traffic among the Node.js instances.
4. Configure HTTPS for secure communication.

## Architecture

The project architecture consists of:

- **app**: A simple Node.js web application.
- **nginx**: An Nginx server configured as a reverse proxy with HTTPS support.
- **Docker Compose**: Orchestrates multiple instances of the Node.js application and the Nginx server.

```
Client ---> Nginx Reverse Proxy (HTTPS) ---> [ app1 | app2 | app3 ]
```

## Prerequisites

- **Docker** installed on your machine. [Install Docker](https://docs.docker.com/get-docker/)
- **Docker Compose** for container orchestration. [Install Docker Compose](https://docs.docker.com/compose/install/)
- **Nginx** installed on your machine. [Install Nginx](https://nginx.org/en/)
- **NodeJS** installed on your machine for backend and web server. [Install NodeJS](https://nodejs.org/en/)

## Installation

1. **Clone the repository:**

   ```bash
   git clone https://github.com/SergiSigro/Nginx-Load-Balancer.git
   cd Nginx-Load-Balancer
   ```

2. **Build and run the containers:**

   ```bash
   docker-compose up -d
   ```

   This command will build the Node.js application image, create three instances.

## Usage

- **Access the web application:**

  Open your web browser and navigate to `https://localhost`. You should see the web application served securely over HTTPS.

- **Stop the containers:**

  ```bash
  docker-compose down
  ```

## Configuration Details

### Node.js Application

The simple Node.js application is located in the `app` directory.

- **app/server.js**

  ```javascript
  const express = require('express');
  const path = require('path');
  const app = express();
  const port = 3000;

  const replicaApp = process.env.APP_NAME

  app.use('/images', express.static(path.join(__dirname, 'images')));

  app.use('/',(req, res) => {
      res.sendFile(path.join(__dirname, 'index.html'));
      console.log(`Request served by ${replicaApp}`);
  }); 

  app.listen(port, () => {
      console.log(`${replicaApp} is listening on port ${port}`);
  });
  ```

- **app/package.json**

  ```json
  {
      "name": "nginx-crash-course",
      "version": "1.0.0",
      "description": "A Node.js application serving a static HTML file, used for load balancing with NGINX.",
      "main": "server.js",
      "scripts": {
        "start": "node server.js"
      },
      "author": "Sergi Sigro Barruz",
      "license": "MIT",
      "dependencies": {
        "express": "^4.17.1",
        "path": "^0.12.7"
      }
    }  
  ```

- **app/Dockerfile**

  ```dockerfile
  FROM node:14

  WORKDIR /app

  COPY server.js .
  COPY index.html .
  COPY images ./images
  COPY package.json .

  RUN npm install

  EXPOSE 3000

  CMD [ "node","server.js" ]
  ```

### Docker Setup

- **docker-compose.yml**

  ```yaml
  version: '3'
  services:
    app1:
      build: .
      environment:
        - APP_NAME=App1
      ports:
        - "3001:3000"
    app2:
      build: .
      environment:
        - APP_NAME=App2
      ports:
        - "3002:3000"
    app3:
      build: .
      environment:
        - APP_NAME=App3
      ports:
        - "3003:3000"
  ```

### Nginx Configuration

- **nginx/conf.d/default.conf**

  ```nginx
    # Main context (this is the global configuration)
  worker_processes 1;

  events {
      worker_connections 1024;
  }

  http {
      include mime.types;

      # Upstream block to define the Node.js backend servers
      upstream nodejs_cluster {
          server 127.0.0.1:3001;
          server 127.0.0.1:3002;
          server 127.0.0.1:3003;
      }

      server {
          # Listen on port 443 for HTTPS
          listen 443 ssl;  
          server_name localhost;

          # SSL certificate settings
          ssl_certificate /Users/sergi/nginx-certs/nginx-selfsigned.crt;
          ssl_certificate_key /Users/sergi/nginx-certs/nginx-selfsigned.key;

          # Proxying requests to Node.js cluster
          location / {
              proxy_pass http://nodejs_cluster;
              proxy_set_header Host $host;
              proxy_set_header X-Real-IP $remote_addr;
          }
      }

      # Optional server block for HTTP to HTTPS redirection
      server {
          listen 8080;
          server_name localhost;

          # Redirect all HTTP traffic to HTTPS
          location / {
              return 301 https://$host$request_uri;
          }
      }

  }
  ```

- **SSL Certificates**

  Place your `server.crt` and `server.key` files in the `nginx/certs` directory. For testing purposes, you can generate self-signed certificates:

  ```bash
  openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
    -keyout nginx/certs/server.key -out nginx/certs/server.crt \
    -subj "/CN=localhost"
  ```

## Customization

- **Scale the number of Node.js instances:**

  Modify the `replicas` value in `docker-compose.yml`:

  ```yaml
  deploy:
    replicas: 5
  ```

- **Change the load balancing algorithm:**

  Edit the `upstream` block in `nginx/conf.d/default.conf` to use different load balancing methods like `least_conn`, `ip_hash`, etc.

  ```nginx
  upstream app_servers {
      least_conn;
      server app:3000;
  }
  ```

- **Use a custom domain:**

  Update the `server_name` directive in the Nginx configuration and configure DNS accordingly.

## License

This project is licensed under the [MIT License](LICENSE).
