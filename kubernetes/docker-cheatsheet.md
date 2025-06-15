# Docker Cheatsheet

Docker is a platform for developing, shipping, and running applications using containerization technology.

## Installation

### macOS

```bash
# Install via Homebrew
brew install --cask docker

# Or download Docker Desktop from https://docker.com/products/docker-desktop
```

### Linux (Ubuntu/Debian)

```bash
# Install Docker Engine
curl -fsSL https://get.docker.com -o get-docker.sh
sh get-docker.sh

# Add user to docker group
sudo usermod -aG docker $USER

# Start Docker service
sudo systemctl start docker
sudo systemctl enable docker
```

### Windows

```powershell
# Download Docker Desktop from https://docker.com/products/docker-desktop
# Or use winget
winget install Docker.DockerDesktop
```

## Basic Commands

### Docker System

```bash
# Check Docker version
docker --version
docker version

# Show system information
docker system info
docker info

# Show system usage
docker system df

# Clean up unused data
docker system prune
docker system prune -a  # Remove all unused data
```

### Images

#### Pull and Push Images

```bash
# Pull an image from registry
docker pull ubuntu:20.04
docker pull nginx:latest

# Push image to registry
docker push myusername/myimage:tag

# Login to registry
docker login
docker login registry.example.com

# Logout from registry
docker logout
```

#### List and Manage Images

```bash
# List all images
docker images
docker image ls

# List images with filters
docker images --filter "dangling=true"
docker images --filter "reference=ubuntu*"

# Remove images
docker rmi image_id
docker rmi ubuntu:20.04
docker image rm image_id

# Remove all unused images
docker image prune
docker image prune -a
```

#### Build Images

```bash
# Build image from Dockerfile
docker build -t myapp:latest .
docker build -f Dockerfile.prod -t myapp:prod .

# Build with build arguments
docker build --build-arg VERSION=1.0 -t myapp:1.0 .

# Build without cache
docker build --no-cache -t myapp:latest .

# Show image history
docker history image_name

# Inspect image details
docker inspect image_name
```

### Containers

#### Run Containers

```bash
# Run container
docker run ubuntu:20.04
docker run -it ubuntu:20.04 /bin/bash  # Interactive with TTY

# Run container in background (detached)
docker run -d nginx:latest

# Run with port mapping
docker run -p 8080:80 nginx:latest
docker run -p 127.0.0.1:8080:80 nginx:latest  # Bind to specific IP

# Run with volume mounts
docker run -v /host/path:/container/path ubuntu:20.04
docker run -v myvolume:/data ubuntu:20.04

# Run with environment variables
docker run -e ENV_VAR=value -e ANOTHER_VAR=value2 ubuntu:20.04

# Run with custom name
docker run --name mycontainer nginx:latest

# Run with resource limits
docker run --memory=512m --cpus=1.0 ubuntu:20.04

# Run with restart policy
docker run --restart=always nginx:latest
docker run --restart=unless-stopped nginx:latest
```

#### Container Management

```bash
# List running containers
docker ps
docker container ls

# List all containers (including stopped)
docker ps -a
docker container ls -a

# Stop containers
docker stop container_id
docker stop container_name
docker stop $(docker ps -q)  # Stop all running containers

# Start containers
docker start container_id
docker restart container_id

# Remove containers
docker rm container_id
docker rm container_name
docker rm $(docker ps -aq)  # Remove all containers

# Remove running container (force)
docker rm -f container_id

# Remove all stopped containers
docker container prune
```

#### Container Interaction

```bash
# Execute command in running container
docker exec -it container_name /bin/bash
docker exec container_name ls -la

# Attach to running container
docker attach container_name

# Copy files between container and host
docker cp file.txt container_name:/path/to/destination
docker cp container_name:/path/to/file.txt ./local_file.txt

# Show container logs
docker logs container_name
docker logs -f container_name  # Follow logs
docker logs --tail 100 container_name  # Last 100 lines

# Show container processes
docker top container_name

# Show container resource usage
docker stats
docker stats container_name

# Inspect container details
docker inspect container_name
```

## Dockerfile

### Basic Dockerfile Instructions

```dockerfile
# Base image
FROM ubuntu:20.04

# Set maintainer
LABEL maintainer="your-email@example.com"

# Set working directory
WORKDIR /app

# Copy files
COPY . /app
ADD archive.tar.gz /app  # Also extracts archives

# Run commands during build
RUN apt-get update && apt-get install -y python3
RUN pip install -r requirements.txt

# Set environment variables
ENV NODE_ENV=production
ENV PORT=3000

# Expose ports
EXPOSE 3000
EXPOSE 8080/tcp

# Create volumes
VOLUME ["/data"]

# Set user
USER 1000
USER appuser

# Define entry point
ENTRYPOINT ["python3", "app.py"]

# Default command
CMD ["--help"]

# Health check
HEALTHCHECK --interval=30s --timeout=3s \
  CMD curl -f http://localhost:3000/ || exit 1
```

### Multi-stage Dockerfile Example

```dockerfile
# Build stage
FROM node:16 AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production

# Production stage
FROM node:16-alpine
WORKDIR /app
COPY --from=builder /app/node_modules ./node_modules
COPY . .
EXPOSE 3000
CMD ["node", "server.js"]
```

## Docker Compose

### Basic docker-compose.yml

```yaml
version: '3.8'

services:
  web:
    build: .
    ports:
      - "3000:3000"
    environment:
      - NODE_ENV=production
    volumes:
      - .:/app
      - /app/node_modules
    depends_on:
      - db

  db:
    image: postgres:13
    environment:
      POSTGRES_DB: myapp
      POSTGRES_USER: user
      POSTGRES_PASSWORD: password
    volumes:
      - postgres_data:/var/lib/postgresql/data
    ports:
      - "5432:5432"

  redis:
    image: redis:6-alpine
    ports:
      - "6379:6379"

volumes:
  postgres_data:

networks:
  default:
    driver: bridge
```

### Docker Compose Commands

```bash
# Start services
docker-compose up
docker-compose up -d  # Detached mode
docker-compose up --build  # Build images before starting

# Stop services
docker-compose down
docker-compose down -v  # Remove volumes
docker-compose stop

# View running services
docker-compose ps

# View logs
docker-compose logs
docker-compose logs web  # Logs for specific service
docker-compose logs -f  # Follow logs

# Execute commands
docker-compose exec web /bin/bash
docker-compose run web python manage.py migrate

# Scale services
docker-compose up --scale web=3

# Build services
docker-compose build
docker-compose build web  # Build specific service

# Pull images
docker-compose pull

# Validate compose file
docker-compose config
```

## Volumes

### Volume Management

```bash
# Create volume
docker volume create myvolume

# List volumes
docker volume ls

# Inspect volume
docker volume inspect myvolume

# Remove volume
docker volume rm myvolume

# Remove all unused volumes
docker volume prune

# Use volume in container
docker run -v myvolume:/data ubuntu:20.04
```

### Bind Mounts vs Volumes

```bash
# Bind mount (host directory)
docker run -v /host/path:/container/path ubuntu:20.04
docker run --mount type=bind,source=/host/path,target=/container/path ubuntu:20.04

# Named volume
docker run -v myvolume:/container/path ubuntu:20.04
docker run --mount source=myvolume,target=/container/path ubuntu:20.04

# Anonymous volume
docker run -v /container/path ubuntu:20.04
```

## Networks

### Network Management

```bash
# List networks
docker network ls

# Create network
docker network create mynetwork
docker network create --driver bridge mynetwork

# Inspect network
docker network inspect mynetwork

# Connect container to network
docker network connect mynetwork container_name

# Disconnect container from network
docker network disconnect mynetwork container_name

# Remove network
docker network rm mynetwork

# Remove all unused networks
docker network prune
```

### Run Container with Custom Network

```bash
# Run with custom network
docker run --network=mynetwork ubuntu:20.04

# Run with host network
docker run --network=host ubuntu:20.04

# Run without network
docker run --network=none ubuntu:20.04
```

## Registry Operations

### Docker Hub

```bash
# Login to Docker Hub
docker login

# Tag image for pushing
docker tag myapp:latest username/myapp:latest

# Push to Docker Hub
docker push username/myapp:latest

# Pull from Docker Hub
docker pull username/myapp:latest
```

### Private Registry

```bash
# Run local registry
docker run -d -p 5000:5000 --name registry registry:2

# Tag for private registry
docker tag myapp:latest localhost:5000/myapp:latest

# Push to private registry
docker push localhost:5000/myapp:latest

# Pull from private registry
docker pull localhost:5000/myapp:latest
```

## Docker Swarm (Orchestration)

### Swarm Management

```bash
# Initialize swarm
docker swarm init
docker swarm init --advertise-addr 192.168.1.100

# Join swarm as worker
docker swarm join --token SWMTKN-... 192.168.1.100:2377

# Join swarm as manager
docker swarm join-token manager

# Leave swarm
docker swarm leave
docker swarm leave --force  # Force leave as manager

# List nodes
docker node ls

# Update node
docker node update --availability drain node_id
```

### Services

```bash
# Create service
docker service create --name web --publish 8080:80 nginx:latest

# List services
docker service ls

# Scale service
docker service scale web=5

# Update service
docker service update --image nginx:1.20 web

# Remove service
docker service rm web

# Service logs
docker service logs web
```

## Security

### Security Best Practices

```bash
# Run as non-root user
docker run --user 1000:1000 ubuntu:20.04

# Read-only root filesystem
docker run --read-only ubuntu:20.04

# Drop capabilities
docker run --cap-drop ALL --cap-add NET_BIND_SERVICE nginx:latest

# Security options
docker run --security-opt no-new-privileges ubuntu:20.04

# Limit resources
docker run --memory=512m --cpus=1.0 ubuntu:20.04

# Use secrets (in swarm mode)
docker secret create my_secret secret.txt
docker service create --secret my_secret nginx:latest
```

### Image Security

```bash
# Scan image for vulnerabilities (Docker Scout)
docker scout cves ubuntu:20.04

# Sign images (Docker Content Trust)
export DOCKER_CONTENT_TRUST=1
docker push username/myapp:latest
```

## Monitoring and Debugging

### Container Monitoring

```bash
# Real-time container stats
docker stats
docker stats --no-stream  # Single snapshot

# Container processes
docker top container_name

# Container changes
docker diff container_name

# Export container as tar
docker export container_name > container.tar

# Import tar as image
docker import container.tar myimage:latest
```

### Debugging

```bash
# Debug container startup
docker run --rm -it ubuntu:20.04 /bin/bash

# Override entrypoint
docker run --rm -it --entrypoint /bin/bash ubuntu:20.04

# Debug failed container
docker logs container_name
docker exec -it container_name /bin/bash

# Inspect container configuration
docker inspect container_name

# Save container as image
docker commit container_name myimage:debug
```

## Cleanup Commands

### System Cleanup

```bash
# Remove all stopped containers
docker container prune

# Remove all unused images
docker image prune
docker image prune -a  # Include unused images

# Remove all unused volumes
docker volume prune

# Remove all unused networks
docker network prune

# Remove everything unused
docker system prune
docker system prune -a --volumes  # Include images and volumes

# Remove all containers (force)
docker rm -f $(docker ps -aq)

# Remove all images
docker rmi $(docker images -q)
```

## Useful Docker Commands

| Category | Command | Description |
|----------|---------|-------------|
| **Images** | `docker images` | List all images |
| | `docker pull <image>` | Pull image from registry |
| | `docker build -t <name> .` | Build image from Dockerfile |
| | `docker rmi <image>` | Remove image |
| **Containers** | `docker ps` | List running containers |
| | `docker ps -a` | List all containers |
| | `docker run <image>` | Run container |
| | `docker stop <container>` | Stop container |
| | `docker rm <container>` | Remove container |
| **Execution** | `docker exec -it <container> bash` | Execute command in container |
| | `docker logs <container>` | View container logs |
| | `docker cp <src> <dest>` | Copy files to/from container |
| **Compose** | `docker-compose up` | Start services |
| | `docker-compose down` | Stop services |
| | `docker-compose ps` | List services |
| **System** | `docker system info` | Show system information |
| | `docker system prune` | Clean up unused resources |

## Environment Variables and Secrets

### Environment Variables

```bash
# Set environment variables
docker run -e VAR1=value1 -e VAR2=value2 ubuntu:20.04

# Use environment file
docker run --env-file .env ubuntu:20.04

# In docker-compose.yml
services:
  web:
    environment:
      - NODE_ENV=production
      - API_KEY=secret
    env_file:
      - .env
```

### Docker Secrets (Swarm Mode)

```bash
# Create secret
echo "my_secret_password" | docker secret create db_password -
docker secret create db_password ./password.txt

# List secrets
docker secret ls

# Use secret in service
docker service create \
  --name myapp \
  --secret db_password \
  myimage:latest
```

## Health Checks

### Health Check Examples

```dockerfile
# In Dockerfile
HEALTHCHECK --interval=30s --timeout=10s --start-period=5s --retries=3 \
  CMD curl -f http://localhost:8080/health || exit 1

HEALTHCHECK --interval=30s --timeout=3s \
  CMD pg_isready -U postgres || exit 1
```

```bash
# Add health check when running
docker run -d \
  --health-cmd="curl -f http://localhost:8080/health || exit 1" \
  --health-interval=30s \
  --health-timeout=10s \
  --health-retries=3 \
  myapp:latest

# Check container health
docker ps  # Shows health status
docker inspect container_name | grep Health
```

## Performance Tips

### Dockerfile Optimization

```dockerfile
# Use specific tags, not 'latest'
FROM node:16-alpine

# Minimize layers by combining RUN commands
RUN apt-get update && \
    apt-get install -y package1 package2 && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

# Use .dockerignore to exclude unnecessary files
# Copy package files first for better caching
COPY package*.json ./
RUN npm ci --only=production
COPY . .

# Use multi-stage builds
FROM node:16 AS builder
# ... build steps ...
FROM node:16-alpine
COPY --from=builder /app/dist ./dist
```

### Runtime Optimization

```bash
# Set resource limits
docker run --memory=512m --cpus=1.5 myapp:latest

# Use init system for proper signal handling
docker run --init myapp:latest

# Enable logging drivers
docker run --log-driver=json-file --log-opt max-size=10m myapp:latest
```

---

*For more detailed information, visit the [official Docker documentation](https://docs.docker.com/)*
