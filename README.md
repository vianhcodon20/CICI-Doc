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
cd my-web-app/
docker build -t my-static-web .
```

Run the Docker container on port 8080
```
docker run -d -p 8080:80 my-static-web
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





sudo docker buildx build --platform linux/amd64,linux/arm64 -t toandnseta/demo-cicd .
[sudo] password for ubuntu: 
[+] Building 0.0s (0/0)                                                                                                                 docker:default
ERROR: Multi-platform build is not supported for the docker driver.
Switch to a different driver, or turn on the containerd image store, and try again.
Learn more at https://docs.docker.com/go/build-multi-platform/






























