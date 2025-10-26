# Jenkins Docker Setup

Simple Jenkins setup with Docker support.

## Requirements

- Docker Desktop
- Windows with WSL2

## Quick Start

### Option 1: Use Docker Hub Image

```bash
docker pull olaola203/kevin-jenkins:latest
```

### Option 2: Build Locally

```bash
docker build -t kevin-jenkins .
```

### Run Jenkins

```bash
docker compose up -d
```

### Access Jenkins

- URL: `http://localhost:8080`
- Get password: `docker exec jenkins cat /var/jenkins_home/secrets/initialAdminPassword`

## Docker Compose

```yaml
services:
  jenkins:
    container_name: jenkins
    image: olaola203/kevin-jenkins:latest  # Docker Hub
    # image: kevin-jenkins  # Local build
    user: root
    privileged: true
    ports:
      - "8080:8080"
      - "50000:50000"
    volumes:
      - ./jenkins_home:/var/jenkins_home
      - //var/run/docker.sock:/var/run/docker.sock
    restart: unless-stopped
```

## Useful Commands

```bash
# Start/Stop
docker compose up -d
docker compose down

# Logs
docker compose logs -f jenkins

# Shell access
docker exec -it jenkins bash

# Test Docker
docker exec jenkins docker --version
```

## Pipeline Example

```groovy
pipeline {
    agent any
    stages {
        stage('Test Docker') {
            steps {
                sh 'docker --version'
                sh 'docker ps'
            }
        }
    }
}
```

## Troubleshooting

- **Docker not found**: Check container permissions
- **Can't start**: Check Docker Desktop is running
- **Reset**: `docker compose down -v && docker compose up -d`

---

**Author**: Kevin
