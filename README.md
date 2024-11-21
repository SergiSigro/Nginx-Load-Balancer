# Nginx Reverse Proxy with HTTPS using Docker and Docker Compose

This repository demonstrates how to create a simple web application with a Node.js backend, dockerize it to run multiple instances, and set up Nginx as a reverse proxy with HTTPS support.

## Table of Contents

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
- [Contributing](#contributing)
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

   This command will build the Node.js application image, create three instances, and set up the Nginx reverse proxy.

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
  const app = express();
  const port = 3000;

  app.get('/', (req, res) => {
    res.send('Hello from Node.js app!');
  });

  app.listen(port, () => {
    console.log(`App listening at http://localhost:${port}`);
  });
  ```

- **app/package.json**

  ```json
  {
    "name": "node-app",
    "version": "1.0.0",
    "description": "Simple Node.js application",
    "main": "server.js",
    "scripts": {
      "start": "node server.js"
    },
    "dependencies": {
      "express": "^4.17.1"
    }
  }
  ```

- **app/Dockerfile**

  ```dockerfile
  FROM node:14
  WORKDIR /usr/src/app
  COPY package*.json ./
  RUN npm install
  COPY . .
  EXPOSE 3000
  CMD [ "npm", "start" ]
  ```

### Docker Setup

- **docker-compose.yml**

  ```yaml
  version: '3'

  services:
    app:
      build: ./app
      environment:
        - NODE_ENV=production
      deploy:
        replicas: 3
      expose:
        - "3000"
      networks:
        - webnet

    nginx:
      image: nginx
      ports:
        - "80:80"
        - "443:443"
      volumes:
        - ./nginx/conf.d:/etc/nginx/conf.d
        - ./nginx/certs:/etc/nginx/certs
      depends_on:
        - app
      networks:
        - webnet

  networks:
    webnet:
  ```

### Nginx Configuration

- **nginx/conf.d/default.conf**

  ```nginx
  upstream app_servers {
      server app:3000;
  }

  server {
      listen 80;
      server_name localhost;

      location / {
          return 301 https://$host$request_uri;
      }
  }

  server {
      listen 443 ssl;
      server_name localhost;

      ssl_certificate     /etc/nginx/certs/server.crt;
      ssl_certificate_key /etc/nginx/certs/server.key;

      location / {
          proxy_pass http://app_servers;
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

## Contributing

Contributions are welcome! Feel free to open an issue or submit a pull request.

## License

This project is licensed under the [MIT License](LICENSE).
