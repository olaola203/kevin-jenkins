# Jenkins Docker Setup với Docker-in-Docker

Hướng dẫn cài đặt và sử dụng Jenkins với Docker-in-Docker trên Windows.

## 📋 Yêu cầu hệ thống

- Windows 10/11 với WSL2
- Docker Desktop for Windows
- Git (tùy chọn)

## 🚀 Cài đặt nhanh

### Phương pháp 1: Sử dụng Image từ Docker Hub (Khuyến nghị)

```bash
# Pull image từ Docker Hub
docker pull olaola203/kevin-jenkins:latest

# Kiểm tra image đã được tải
docker images olaola203/kevin-jenkins
```

### Phương pháp 2: Build Image từ source

```bash
# Build image kevin-jenkins từ Dockerfile
docker build -t kevin-jenkins .

# Kiểm tra image đã được tạo
docker images kevin-jenkins
```

### Cập nhật Docker Compose

Cập nhật file `docker-compose.yml` để sử dụng image:

```yaml
services:
  jenkins:
    container_name: jenkins
    image: olaola203/kevin-jenkins:latest  # Sử dụng image từ Docker Hub
    # hoặc
    # image: kevin-jenkins  # Sử dụng image local
    user: root
    privileged: true
    ports:
      - "8080:8080"
      - "50000:50000"
    volumes:
      - ./jenkins_home:/var/jenkins_home
      - //var/run/docker.sock:/var/run/docker.sock
    restart: unless-stopped
    environment:
      - DOCKER_HOST=unix:///var/run/docker.sock
```

### Bước 3: Chạy Jenkins

```bash
# Khởi động Jenkins
docker-compose up -d

# Kiểm tra container đang chạy
docker ps

# Xem logs
docker-compose logs -f jenkins
```

## 🔧 Cấu hình Jenkins

### Truy cập Jenkins

1. Mở trình duyệt và truy cập: `http://localhost:8080`
2. Lấy mật khẩu admin ban đầu:

```bash
# Lấy mật khẩu admin
docker exec jenkins cat /var/jenkins_home/secrets/initialAdminPassword
```

3. Làm theo hướng dẫn setup Jenkins

### Cài đặt Docker Plugin

1. Vào **Manage Jenkins** → **Manage Plugins**
2. Tìm và cài đặt plugin **Docker**
3. Restart Jenkins sau khi cài đặt

## 🐳 Sử dụng Docker trong Jenkins

### Pipeline Example

Tạo một Jenkinsfile để test Docker:

```groovy
pipeline {
    agent any
    
    stages {
        stage('Test Docker') {
            steps {
                script {
                    // Test Docker command
                    sh 'docker --version'
                    sh 'docker ps'
                }
            }
        }
        
        stage('Build Docker Image') {
            steps {
                script {
                    // Build một image đơn giản
                    sh '''
                        echo "FROM alpine:latest" > Dockerfile.test
                        echo "RUN echo 'Hello from Docker!'" >> Dockerfile.test
                        docker build -f Dockerfile.test -t test-image .
                        docker run test-image
                    '''
                }
            }
        }
    }
    
    post {
        always {
            // Cleanup
            sh 'docker rmi test-image || true'
            sh 'rm -f Dockerfile.test'
        }
    }
}
```

## 📁 Cấu trúc thư mục

```
jenkins/
├── docker-compose.yml    # Docker Compose configuration
├── Dockerfile           # Custom Jenkins image với Docker CLI
├── jenkins_home/        # Jenkins data (tự động tạo)
└── README.md           # File hướng dẫn này
```

## 🔄 Quản lý Container

### Các lệnh hữu ích

```bash
# Khởi động Jenkins
docker-compose up -d

# Dừng Jenkins
docker-compose down

# Restart Jenkins
docker-compose restart

# Xem logs
docker-compose logs -f jenkins

# Truy cập shell trong container
docker exec -it jenkins bash

# Kiểm tra Docker trong container
docker exec jenkins docker --version
```

### Backup và Restore

```bash
# Backup Jenkins data
docker exec jenkins tar -czf /tmp/jenkins-backup.tar.gz -C /var/jenkins_home .
docker cp jenkins:/tmp/jenkins-backup.tar.gz ./jenkins-backup.tar.gz

# Restore Jenkins data
docker cp ./jenkins-backup.tar.gz jenkins:/tmp/
docker exec jenkins tar -xzf /tmp/jenkins-backup.tar.gz -C /var/jenkins_home
```

## 🛠️ Troubleshooting

### Lỗi thường gặp

1. **Docker command not found**
   ```bash
   # Kiểm tra Docker trong container
   docker exec jenkins which docker
   docker exec jenkins docker --version
   ```

2. **Permission denied**
   ```bash
   # Kiểm tra quyền container
   docker exec jenkins ls -la /var/run/docker.sock
   ```

3. **Container không khởi động**
   ```bash
   # Xem logs chi tiết
   docker-compose logs jenkins
   ```

### Reset hoàn toàn

```bash
# Dừng và xóa tất cả
docker-compose down -v
docker rmi kevin-jenkins

# Build lại và chạy
docker build -t kevin-jenkins .
docker-compose up -d
```

## 📝 Ghi chú

- Jenkins data được lưu trong thư mục `./jenkins_home`
- Container chạy với quyền root để có thể sử dụng Docker
- Port 8080: Jenkins web interface
- Port 50000: Jenkins agent connections

## 🆘 Hỗ trợ

Nếu gặp vấn đề, hãy kiểm tra:
1. Docker Desktop đang chạy
2. WSL2 được bật
3. Container có quyền privileged
4. Docker socket được mount đúng

---

**Tác giả**: Kevin Project  
**Ngày tạo**: $(date)  
**Phiên bản**: 1.0
