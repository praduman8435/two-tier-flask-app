# 2-Tier Application Deployment

## [Launch an EC2 instance on AWS](#wrap-up)

### name the EC2 instance:

  ![name.png](images/name.png)

### select an OS image ubuntu free tier eligible:

  ![os.png](images/os.png)

### create a new key pair:

  ![key.png](images/key.png)

### rest remain as default and click on launch instance:

  ![launch.png](images/launch.png)

### make sure instance is running:

  ![instance.png](images/instance.png)


## [_Connect EC2 instance with the local terminal_](#wrap-up)

### connect the created EC2 instance to the local terminal using ssh client:

  ![connect.png](images/connect.png)

### open the local terminal and run the commands:

  ```bash
  chmod 400 2-tier-app.pem
  ```

### check your instance public IP address:

  ![ip.png](images/ip.png)

### run command on local terminal:

  ```bash
  ssh -i "path-of-your-key-pair" ubuntu@13.232.235.216
  ```

### EC2 instance connected:

  ![cnection.png](images/cnection.png)

### update and upgrade the machine :

  ```bash
  sudo apt update
  sudo apt upgrade
  ```

## [_Setting up Docker on ubuntu_](#wrap-up)

 ```bash
 sudo apt install docker.io
 ```

### change owner to the current user for /var/run/docker.sock :

  ```bash
  sudo chown $USER /var/run/docker.sock
  ```

### to check images that are running :

  ```bash
  docker ps
  ```

### clone the project repo from github :

  ```bash
  git clone https://github.com/praduman8435/two-tier-flask-app.git
  ```


## [_Create a Docker container using the image_](#wrap-up)

 ```bash
 docker run -d -p 5000:5000 flaskapp:latest
 ```

### change the inbound rules to access the port 5000 from anywhere:

  ![inbnd.png](images/inbnd.png)

### create a mysql container using the image:

  ```bash
  docker run -d -p 3306:3306 --name mysql -e MYSQL_ROOT_PASSWORD="admin" mysql:5.7
  ```

### check the containers that are running:

  ```bash
  docker ps
  ```

  ![dbh.png](images/dbh.png)

### _here the both containers are running but we can’t see website on the port 5000 because both the conatiners are not interconnected_


## [_Create a Dockerfile for flask application_](#wrap-up)

### change directory to two-tier-flask-app:

  ![cd.png](images/cd.png)

  ![ls.png](images/ls.png)

### the remove the Dockerfile first to write from beginning:

  ```bash
  rm -rf Dockerfile
  ```

### create a Dockerfile:

  ```bash
  vim Dockerfile
  ```

  ```docker
  #base image (gives an operating system in which python installed)
  FROM python:3.9-slim

  #we need to run the application in a folder (so create one)
  WORKDIR /app

      #update the OS
  RUN apt-get update -y \
      #upgrade the packages
      && apt-get upgrade -y \
      #install mysql libraries\
      && apt-get install -y gcc default-libmysqlclient-dev pkg-config \
      #remove the temporary files that creates during installation
      && rm -rf /var/lib/lists/*

  #copy requirements.txt file
  COPY requirements.txt .

  #python is internally accesing mysql so we require to install a mysql client of python
  RUN pip install mysqlclient

  #install the packages in requiremets.txt
  RUN pip install -r requirements.txt

  COPY . .

  CMD ["python","app.py"]
  ```

### build image using Dockerfile:

  ```bash
  docker build . -t flaskapp
  ```

### to check the images:

  ```bash
  docker images
  ```

  ![im.png](images/im.png)


## [_Create a docker network to connect the flaskapp and mysql container_](#wrap-up)

 ```bash
 docker network create "network-name"
 ```

### now we required to attach this network with both the container so we need to kill the previous containers that are running and make a new one:

  ```bash
  docker kill "container_id_mysql"  "container_id_flask"
  ```

  ![jid.png](images/jid.png)

### now create again containers for mysql and flask using docker run command in which docker network are attached:

  ```bash
  docker run -d --name mysql --network=twotier -e MYSQL_DATABASE=myDb -e MYSQL_USER=admin -e 
  MYSQL_PASSWORD=admin -e MYSQL_ROOT_PASSWORD=admin -p 3306:3306 mysql:5.7
  ```

  ```bash
  docker run -d --name flaskapp --network=twotier -e MYSQL_HOST=mysql -e MYSQL_USER=admin -e 
  MYSQL_PASSWORD=admin -e MYSQL_DB=myDb -p 5000:5000 flaskapp:latest
  ```

### to check the network that are present:

  ```bash
  docker network ls
  ```

  ![ner.png](images/ner.png)

### to check the containers that are in the network:

  ```bash
  docker network inspect "network-name"
  ```

  ![inu.png](images/inu.png)


## [Create the `messages` table in MySQL database](#wrap-up)

### firstly, we require to enter in the mysql container:

  ```bash
  docker exec -it "mysql-container-id" bash
  ```

  ```bash
  mysql -u root -p
  #enter the password you save for mysql
  ```

  ![oio.png](images/oio.png)

  ```bash
  show databases;
  ```

  ![pp.png](images/pp.png)

  ```bash
  use myDb;
  ```

  ```bash
  CREATE TABLE messages (
      id INT AUTO_INCREMENT PRIMARY KEY,
      message TEXT
  );
  ```

  ![ui.png](images/ui.png)


## [now you can see the website on public IPv4 address on port 5000:](#wrap-up)

###  ![tn.png](images/tn.png)

###  ![tr.png](images/tr.png)
