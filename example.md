example 1
-----

```
# File: .github/workflows/build-and-deploy.yml

name: Build and Deploy to Server

on:
  push:
    branches:
      - main
  pull_request:
    branches:
      - main

jobs:
  build-and-deploy:
    name: Build and Deploy to Server
    runs-on: self-hosted

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Set up QEMU
        uses: docker/setup-qemu-action@v3

      - name: Log in to Docker Hub
        uses: docker/login-action@v2
        with:
          username: ${{ secrets.DOCKER_USERNAME }}
          password: ${{ secrets.DOCKER_PASSWORD }}

      - name: Tags Docker image
        id: meta
        uses: docker/metadata-action@v5
        with:
          images: toandnseta/demo-cicd

      - name: Build and push to Docker Hub
        uses: docker/build-push-action@v6
        with:
          context: .
          push: true
          tags: ${{ steps.meta.outputs.tags }}

      - name: Deploy to Ubuntu Server
        run: |
          docker login -u ${{ secrets.DOCKER_USERNAME }} -p ${{ secrets.DOCKER_PASSWORD }}
          docker pull toandnseta/demo-cicd:latest
          docker ps -q | xargs -r docker stop || true
          docker ps -aq | xargs -r docker rm || true
          docker run -d --name demo-cicd -p 192.168.81.156:8080:80 toandnseta/demo-cicd:latest
```




----------
example 2
--------------


```
# Update 1


# File: .github/workflows/deploy-ci.yml

name: CI for Docker HTML Project

on:
  push:
    branches:
      - main

jobs:
  build-and-push-docker-image:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v3

      - name: Log in to Docker Hub
        uses: docker/login-action@v2
        with:
          username: ${{ secrets.DOCKER_USERNAME }}
          password: ${{ secrets.DOCKER_PASSWORD }}

      - name: Build Docker image
        run: |
          docker build -t ${{ secrets.DOCKER_USERNAME }}/html-project:latest .

      - name: Push Docker image to Docker Hub
        run: |
          docker push ${{ secrets.DOCKER_USERNAME }}/html-project:latest

# File: .github/workflows/deploy-server.yml

name: CD to Ubuntu Server

on:
  push:
    branches:
      - main

jobs:
  deploy-to-server:
    runs-on: ubuntu-latest

    steps:
      - name: SSH into server and pull Docker image
        uses: appleboy/ssh-action@v0.1.7
        with:
          host: 192.168.81.156
          username: ${{ secrets.SERVER_USERNAME }}
          key: ${{ secrets.SERVER_SSH_KEY }}
          script: |
            docker login -u ${{ secrets.DOCKER_USERNAME }} -p ${{ secrets.DOCKER_PASSWORD }}
            docker pull ${{ secrets.DOCKER_USERNAME }}/html-project:latest
            docker stop html-container || true
            docker rm html-container || true
            docker run -d --name html-container -p 80:80 ${{ secrets.DOCKER_USERNAME }}/html-project:latest
```

