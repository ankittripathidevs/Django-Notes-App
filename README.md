# Django Notes App — SQLite & MySQL

A full-stack Notes application built with **React.js** and **Django REST Framework**.

This project supports two database configurations:

- **Django + SQLite** — simple deployment with plain Docker
- **Django + MySQL** — multi-container deployment with Docker Compose
- **Nginx** — optional reverse proxy

## Tech Stack

- React.js
- Django 4.1.5
- Django REST Framework
- SQLite3
- MySQL 8
- Docker
- Docker Compose
- Nginx


# Django + SQLite with Docker

This is the simple single-container deployment.

## 1. Build the Docker Image

```bash
docker build -t notes-app:latest .
```

## 2. Run the Container

```bash
docker run -d   --name notes-app   -p 8000:8000   notes-app:latest
```

## 3. Check the Container

```bash
docker ps
```

## 4. View Logs

```bash
docker logs notes-app
```

## 5. Test the Application

```bash
curl http://localhost:8000
```

Open in a browser:

```text
http://EC2-PUBLIC-IP:8000
```

---

# Django + MySQL with Docker Compose

For the MySQL setup, Django and MySQL run in separate containers.

```text
                Docker Network
                     │
          ┌──────────┴──────────┐
          │                     │
          ▼                     ▼
     Django App              MySQL
     notes-app               db_cont
       :8000                  :3306
```

## 1. Environment Variables

Create a `.env` file:

```env
USE_MYSQL=true

DB_NAME=notes_db
DB_USER=notes_user
DB_PASSWORD=your_password
DB_HOST=db_cont
DB_PORT=3306
```

Do not commit real credentials to Git.

Add `.env` to `.gitignore`:

```gitignore
.env
```

## 2. Start the Application

```bash
docker compose up -d --build
```

## 3. Check Containers

```bash
docker compose ps
```

You should have both the Django and MySQL services running.


## 4. View Logs

```bash
docker compose logs django_app
```

```bash
docker compose logs db_cont
```

## 5. Access the Application

```text
http://EC2-PUBLIC-IP:8000
```

---

# Nginx Reverse Proxy

Nginx can be placed in front of Django to expose the application through port `80`.

```text
Internet
    │
    ▼
EC2 :80
    │
    ▼
 Nginx
    │
    ▼
Django :8000
    │
    ▼
SQLite / MySQL
```

## Install Nginx

```bash
sudo apt-get update
sudo apt install nginx -y
```

Check Nginx:

```bash
sudo systemctl status nginx
```

```bash
sudo systemctl restart nginx
```

Now access:

```text
http://EC2-PUBLIC-IP
```

If Nginx is the public entry point, allow **HTTP port 80** in the EC2 Security Group.

---


# Docker Commands
### List running containers

```bash
docker ps
```

### List all containers

```bash
docker ps -a
```

### View logs

```bash
docker logs notes-app
```

### Stop container

```bash
docker stop notes-app
```

### Remove container

```bash
docker rm notes-app
```

### Remove image

```bash
docker rmi notes-app:latest
```

### Rebuild image

```bash
docker build --no-cache -t notes-app:latest .
```

---

# Docker Compose Commands

### Start

```bash
docker compose up -d
```

### Build and start

```bash
docker compose up -d --build
```

### Stop and remove containers

```bash
docker compose down
```

### Check services

```bash
docker compose ps
```

### View logs

```bash
docker compose logs
```

### Follow logs

```bash
docker compose logs -f
```

---

# Deployment Architecture

## Plain Docker + SQLite

```text
EC2
 │
 │ :8000
 ▼
Django Container
 │
 ▼
SQLite
db.sqlite3
```

## Docker Compose + MySQL

```text
EC2
 │
 │ :8000
 ▼
Django Container
 │
 │ Docker Network
 ▼
MySQL Container
 │
 ▼
Persistent MySQL Data
```

## Nginx + Django

```text
Internet
    │
    │ :80
    ▼
  Nginx
    │
    │ :8000
    ▼
 Django
    │
    ▼
SQLite / MySQL
```

## Deployment Options

| Setup | Database | Containers | Public Port |
|---|---|---:|---:|
| Plain Docker | SQLite | 1 | 8000 |
| Docker Compose | MySQL | 2 | 8000 |
| Docker + Nginx | SQLite/MySQL | 1+ | 80 |
| Docker Compose + Nginx | MySQL | 3 | 80 |

---

