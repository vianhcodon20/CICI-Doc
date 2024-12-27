# CICI-Doc

## Step 1:

Tao repo moi tren github























## Step

Tao Dockerfile cho web static

`Dockerfile`
```
# Sử dụng base image Nginx để phục vụ file tĩnh
FROM nginx:latest

# Copy các file tĩnh của bạn vào thư mục mặc định của Nginx
COPY . /usr/share/nginx/html

# Expose port 80 để phục vụ nội dung
EXPOSE 80

# Chạy Nginx ở foreground
CMD ["nginx", "-g", "daemon off;"]
```

## Step

Build Docker images tu Dokerfile
```
cd demo-cicd/
docker build -t demo-cicd .
```

Run the Docker container on port 8080
```
docker run -d -p 8080:80 demo-cicd
```

Kiem tra web:

```
http://localhost:8080/
```

## Step 

Day image vua duoc build len Docker Hub de luu tru

```
docker login
```

![image](https://github.com/user-attachments/assets/8b5902ab-95a4-4316-afbf-81d2df704f3c)


Build image voi username docker hub
```
sudo docker build --platform=linux/amd64,linux/arm64 -t toandnseta/demo-cicd .
```

------
Neu gap loi nay
------

```
sudo docker buildx build --platform linux/amd64,linux/arm64 -t toandnseta/demo-cicd .
[sudo] password for ubuntu: 
[+] Building 0.0s (0/0)                                                                                                                 docker:default
ERROR: Multi-platform build is not supported for the docker driver.
Switch to a different driver, or turn on the containerd image store, and try again.
Learn more at https://docs.docker.com/go/build-multi-platform/
```

Fix

Enable BuildKit

```
export DOCKER_BUILDKIT=1
```

Create and Use a New Builder with `docker container` Driver

```
docker buildx create --use
docker buildx build --platform linux/amd64,linux/arm64 -t toandnseta/demo-cicd .
```

Retry the Build Command

```
docker buildx build --platform linux/amd64,linux/arm64 -t toandnseta/demo-cicd --push .
```
------

## Step:
Build server

Install docker

```
sudo apt update && sudo apt upgrade -y
```

```
sudo apt remove docker docker-engine docker.io containerd runc
```

```
sudo apt install -y apt-transport-https ca-certificates curl software-properties-common
````

```
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
```

```
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

```
sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io
```

```
sudo docker --version
```

```
sudo systemctl enable docker
sudo systemctl start docker
```

Run docker khong can quyen root (sudo)

```
sudo usermod -aG docker $USER
```

Pull project

```
docker pull toandnseta/demo-cicd
```

Kiem tra image da duoc pull ve chua

```
sudo docker image ls
```

Chay container vua duoc pull ve

```
sudo docker run -d -p 8080:80 toandnseta/demo-cicd
```

Kiem tra container da duoc chay chua

```
http://192.168.81.156:8080/
```


## Step 
Thiết lập quy trình CI/CD bằng Github Actions

Tại project `demo-cicd`. Tao thu muc sau:
- .github
  - workflows
    - deploy.yml 

```
name: "Build and deploy to server"

on:
  push:
    branches: ["main"]
  pull_request:
    branches: ["main"]

jobs:
  deploy:
    name: Deploy to server
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Set up QEMU
        uses: docker/setup-qemu-action@v3

      - name: Login to docker hub
        uses: docker/login-action@v2
        with:
          username: ${{ secrets.DOCKER_USERNAME }}
          password: ${{ secrets.DOCKER_PASSWORD }}

      - name: Tags docker image
        id: meta
        uses: docker/metadata-action@v5
        with:
          images: ${{ secrets.DOCKER_USERNAME }}/demo-cicd # Change this to your docker image name

      - name: Build and push to docker hub
        uses: docker/build-push-action@v6
        with:
          context: .
          push: true
          tags: ${{ steps.meta.outputs.tags }}
```



























