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







































