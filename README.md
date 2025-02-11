# DevOps-Docker
```bash
mkdir /tools;cd /tools/;mkdir docker; cd docker
vi install-docker.sh
chmod +x install-docker.sh
./install-docker.sh
docker pull ubuntu:24.04
docker images
root@c75f6482d55c:~# apt update
root@c75f6482d55c:~# apt install netstat
root@c75f6482d55c:~# apt install net-tools -y
root@c75f6482d55c:~# exit
docker ps -a
docker start ubuntu
docker exec -it ubuntu bash
exit
docker run --name nginx -dp 9999:80 nginx
docker ps -a
docker stop ubuntu
docker rmi ubuntu
```
install-docker.sh content:
```bash
#!/bin/bash

sudo apt update
sudo apt install -y apt-transport-https ca-certificates curl software-properties-common
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
echo "deb [signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt update
sudo apt install -y docker-ce
sudo systemctl start docker
sudo systemctl enable docker
sudo curl -L "https://github.com/docker/compose/releases/latest/download/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
docker --version
docker-compose --version
```
Create Dockerfile for shoeshop project
```bash
echo "
##BUILD STAGE##
FROM maven:3.5.3-jdk-8-alpine AS builder
WORKDIR /app
COPY . .
RUN mvn install -DskipTest=true
##RUN STAGE##
FROM eclipse-temurin:8-jre-alpine
WORKDIR /run
COPY --from=builder /app/target/shoe-ShoppingCart-0.0.1-SNAPSHOT.jar /run/
EXPOSE 8080
ENTRYPOINT java -jar /run/shoe-ShoppingCart-0.0.1-SNAPSHOT.jar
"> /home/giang/datas/shoeshop/Dockerfile
docker build -t shoeshop:v1 .
docker run --name shoeshop -dp 8888:8080 shoeshop:v1
```
Show log docker app:
```bash
docker logs -f shoeshop
```
# Docker build and run personal project with specific user todolist
Dockerfile content
```bash
##BUILD STAGE##
FROM node:16.3.0-alpine AS builder
WORKDIR /app
COPY . .
RUN npm install
RUN npm run build
##RUN STAGE##
FROM nginx:alpine
WORKDIR /run
RUN addgroup -S todolist && adduser -S todolist -G todolist
COPY --from=builder /app/dist/ /usr/share/nginx/html
RUN chown -R todolist:todolist /var/cache/nginx /var/run /var/log/nginx /usr/share/nginx/html /etc/nginx
RUN sed -i 's|/var/run/nginx.pid|/tmp/nginx.pid|g' /etc/nginx/nginx.conf
USER todolist
EXPOSE 80
CMD ["nginx", "-g","daemon off;"]
```
# Docker Registry
```bash
mkdir registry; cd registry
apt install openssl
mkdir certs
openssl req -newkey rsa:4096 -nodes -sha256 -keyout certs/domain.key -subj "/CN=192.168.184.132" -addext "subjectAltName = DNS:192.168.184.132,IP:192.168.184.132" -x509 -days 365 -out certs/domain.crt
echo "
version: '3'

services:
        registry:
                image: registry:2
                restart: always
                container_name: registry-server
                ports:
                        - "5000:5000"
                volumes:
                        - ./data:/var/lib/registry
                        - ./certs:/certs
                environment:
                        REGISTRY_HTTP_TLS_CERTIFICATE: /certs/domain.crt
                        REGISTRY_HTTP_TLS_KEY: /certs/domain.key
" > docker-compose.yml
docker-compose up -d
docker login 192.168.184.132:5000
mkdir -p /etc/docker/certs.d/192.168.184.132:5000/
cp certs/domain.crt /etc/docker/certs.d/192.168.184.132:5000/ca.crt
systemctl restart docker
docker login 192.168.184.132:5000
```
# Copy cert to lab-server then login to registry and push docker images
```bash
root@dev-server:/tools/docker# mkdir -p /etc/docker/certs.d/192.168.184.132:5000/
root@registry-server:/tools/registry# scp certs/domain.crt giang@192.168.184.130:/home/giang
root@dev-server:/tools/docker# cp /home/giang/domain.crt /etc/docker/certs.d/192.168.184.132:5000/
root@dev-server:/tools/docker# systemctl restart docker
root@dev-server:/tools/docker# docker login 192.168.184.132:5000
root@dev-server:/tools/docker# docker tag todolist:v1 192.168.184.132:5000/elroydevops/todolist:v1
root@dev-server:/tools/docker# docker push 192.168.184.132:5000/elroydevops/todolist:v1
firefox https://192.168.184.132:5000/v2/_catalog
