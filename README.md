# DeepSeek Deployment Guide

This guide provides step-by-step instructions to host **DeepSeek** using **Nginx** and **Docker**.

---

## Table of Contents

- [About DeepSeek](#about-deepseek)
- [Tech Stack](#tech-stack)
- [Prerequisites](#prerequisites)
- [Commands](#commands)
- [Docker compose yaml file](#docker-compose-yaml-file)
- [Nginx configuration](#nginx-configuration)
- [Contact](#contact)

---

## About DeepSeek

DeepSeek is the name of a free AI-powered chatbot, which looks, feels and works very much like ChatGPT.
That means it's used for many of the same tasks, though exactly how well it works compared to its rivals is up for debate.
It is reportedly as powerful as OpenAI's o1 model - released at the end of last year - in tasks including mathematics and coding.
Like o1, R1 is a "reasoning" model. These models produce responses incrementally, simulating how humans reason through problems or ideas.

---

## Tech Stack

- **Web Server:** Nginx
- **Containerization:** Docker and Docker Compose
- **LLM's running Framework:** Ollama 
- **Frontend:** Open Webui

---

## Prerequisites

Before you begin, ensure you have the following installed:

- [Docker](https://docs.docker.com/get-docker/)
- [Docker Compose](https://docs.docker.com/compose/install/)
- [Nginx](https://nginx.org/en/docs/install.html)
- [Ollama](https://ollama.com/download)

---

## Commands
- ### update system
  ```bash
  sudo apt-get update
  ```
- ### Install ollama
  ```bash
  curl -fsSL https://ollama.com/install.sh | sh
  ```
- ### Install nginx
  ```bash
  sudo apt-get install nginx
  ```
- ### Install docker-compose
  ```bash
  sudo apt-get install docker-compose
  ```
- ### Stop ollama service
  ```bash
  sudo systemctl stop ollama
  ```
- ### clone openui/webui
  ```bash
  git clone https://github.com/open-webui/open-webui.git
  ```
- ### run docker compose ( check below for docker-compose.yaml file modification)
  ```bash
  cd open-webui/ && docker compose up -d
  ```
- ### issue with buildX commands
  ```bash
    BUILDX_VERSION="v0.20.1"
    mkdir -p ~/.docker/cli-plugins
    curl -fsSL "https://github.com/docker/buildx/releases/download/${BUILDX_VERSION}/buildx-${BUILDX_VERSION}.linux-amd64" -o ~/.docker/cli-plugins/docker-buildx
    chmod +x ~/.docker/cli-plugins/docker-buildx
  ```
- ### issue with package.json (replace this in package.json file for build)
  ```bash
  "build": "export NODE_OPTIONS=--max_old_space_size=4096 && npm run pyodide:fetch && vite build",
  ```

---
## Docker compose yaml file
  ```yaml
  services:
    ollama:
      volumes:
        - ollama:/root/.ollama
      container_name: ollama
      pull_policy: always
      tty: true
      restart: unless-stopped
      image: ollama/ollama:${OLLAMA_DOCKER_TAG-latest}
  
    open-webui:
      build:
        context: .
        args:
          OLLAMA_BASE_URL: 'http://ollama:11434'
        dockerfile: Dockerfile
      image: ghcr.io/open-webui/open-webui:${WEBUI_DOCKER_TAG-main}
      container_name: open-webui
      volumes:
        - open-webui:/app/backend/data
      depends_on:
        - ollama
      ports:
        - "0.0.0.0:8082:8080" #${OPEN_WEBUI_PORT-3000}:8080
      environment:
        OLLAMA_BASE_URL: http://ollama:11434
        WEBUI_NAME: Aikopoko
      extra_hosts:
        - host.docker.internal:host-gateway
      restart: unless-stopped
  
  volumes:
    ollama: {}
    open-webui: {}
  ```
---
## Nginx configuration
```nginx
upstream openwebui {
server 0.0.0.0:8082;
}

server {
listen 80;
listen [::]:80;
root /var/www/html;
# Add index.php to the list if you are using PHP

index index.html index.htm index.nginx-debian.html;

server_name yoursite.com;

# To allow special characters in headers
ignore_invalid_headers off;
# Allow any size file to be uploaded.
# Set to a value such as 1000m; to restrict file size to a specific value
client_max_body_size 0;
# To disable buffering
proxy_buffering off;
location / {
    proxy_set_header Host $http_host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;
    proxy_connect_timeout 300;
    # Default is HTTP/1, keepalive is only enabled in HTTP/1.1
    proxy_http_version 1.1;
    chunked_transfer_encoding off;
    proxy_pass http://openwebui;

     # Add WebSocket support
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";

    set $upstream_keepalive false;
}

}
```
---
## Contact

AikoPoko - [aikopoko.np@gmail.com](mailto:aikopoko.np@gmail.com)
