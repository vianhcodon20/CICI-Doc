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

## Step:

Vi ly do bao mat nen ta khong the public thong tin cua user pass cua Docker Hub nen ta sẽ sử dụng tính năng secrets của Github để bảo vệ và quản lý các thông tin này.

Truy cập lại vào repository trên Github, chọn `Repo` -> `demo` -> `Setting` -> `Secrets and variables` -> `Actions` -> `New repository secret`

![image](https://github.com/user-attachments/assets/54810f3a-8d65-482d-abce-9ca226914c89)

Chay lai actions

![image](https://github.com/user-attachments/assets/5330bb92-5456-4a93-8746-75871ac081ae)



![image](https://github.com/user-attachments/assets/3b453277-e95a-4a8d-8b3e-45f8cc770f1d)


## Step

Khoi chay container tren Server

Tao 1 wfl moi `deploy-server.yml`

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

      - name: Deploy to server
        uses: appleboy/ssh-action@v1.0.3
        with:
          host: ${{ secrets.HOST }} # Địa chỉ của server
          username: ${{ secrets.USERNAME }} # Username để login vào server
          key: ${{ secrets.SSH_KEY }} # Private key để login vào server

          script: |
#           Pull image về lại server
            docker pull ${{ steps.meta.outputs.tags }}
#           Xóa container cũ nếu có
            docker rm -f demo-cicd &>/dev/null
#           Chạy container mới
            docker run -it -d --name demo-cicd -p 8080:80 ${{ steps.meta.outputs.tags }}
```

Them secret HOST tren github
![image](https://github.com/user-attachments/assets/e7a28cbc-7cb8-4b47-951a-fad0dab079cc)


















