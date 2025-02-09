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
