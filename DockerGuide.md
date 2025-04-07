## 🐳 Docker Commands & Node.js Project Containerization

### 🔧 Basic Docker Commands

```bash
docker -v
docker info
docker --help
```

### 🐳 Docker Hub

1. Create an account on [Docker Hub](https://hub.docker.com)
2. Use this account to push/pull Docker images

### 📦 Docker Images & Containers

```bash
docker pull nginx:latest
docker pull <imageName>:<Tag>
docker search <imageName>
docker images
docker ps
docker ps -a
docker images -aq
docker inspect <containerId>
docker rmi <imageName>
docker rmi -f <imageName>
docker rm <containerId>
docker rm $(docker ps -aq)
docker rmi $(docker images -aq)
```

### 🚀 Running & Managing Containers

```bash
docker run -dit -p <localPort>:<containerPort> <imageId>
docker run -dit --name <containerName> -p 3000:8080 <imageId>
docker exec -it <containerId> bash
docker attach <containerId>
docker start <containerId>
docker stop <containerId>
docker stop $(docker ps -aq)
docker logs <containerId>
```

### 🐧 Working with CentOS (Example)

```bash
docker pull centos:7
docker run -it centos:7
```

### 📂 Dockerfile Example (Node.js)

Create a `Dockerfile` in your project folder:

```Dockerfile
FROM node
WORKDIR /myapp
COPY . /myapp
RUN npm install
CMD ["npm", "start"]
```

### 🛠️ Build & Run Custom Image

```bash
docker build -t nodejs:v1 .
docker run -dit nodejs:v1
docker run -dit -p 3000:80 nodejs:v1
```

### 🏷️ Tag and Push Image to Docker Hub

```bash
docker tag <imageName> <yourDockerHubUsername>/<repo_name>
docker push <yourDockerHubUsername>/<repo_name>
```

### 🧹 System Clean-up

```bash
docker system prune
```

### 📦 Containerize a Node.js Project

1. Create your app folder structure:

```
myapp/
├── Dockerfile
├── package.json
├── package-lock.json
└── app.js
```

2. Add Dockerfile (see above)

3. Ensure package.json has:
```json
"scripts": {
  "start": "node app.js"
}
```

4. Build the image:
```bash
docker build -t my-node-app:v1 .
```

5. Run the container:
```bash
docker run -dit -p 3000:3000 my-node-app:v1
```

6. Access app: Visit `http://localhost:3000`

---
