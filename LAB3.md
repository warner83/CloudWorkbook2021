# Upload the file employees.yaml on https://editor.swagger.io/
# Download the generated file Generate Server -> Python Flask
# Upload the file on the VM
# Generate the container
    docker build -t rest-server .

# Run the container
    docker run -p 8080:8080 -it rest-server

# Test it

    curl -X POST "http://172.16.0.253:8080/v2/employees" -H  "accept: application/json" -H  "Content-Type: application/json" -d "{  \"id\": 0,  \"name\": \"Jim\",  \"photoUrls\": \"http://mywebsite.org/mypic.jpg\"}"

# Deploy a database
    docker pull mysql/mysql-server
    docker run --name=mysql -d mysql/mysql-server
    
# Recover MySQL password
    docker logs mysql 2>&1 | grep GENERATED

# Configure the database
    docker exec -it mysql bash
    mysql -uroot –p
    ALTER USER 'root'@'localhost' IDENTIFIED BY 'password';
    use mysql;
    update user set host='%' where host='localhost' AND user='root';
    FLUSH PRIVILEGES;
    
# Create a new database
    CREATE DATABASE company;
    CREATE TABLE IF NOT EXISTS `company`.`employees` ( 
    `id` INT  AUTO_INCREMENT ,
    `name` VARCHAR(150) NOT NULL ,
    `picUrl` VARCHAR(150) NOT NULL ,
    PRIMARY KEY (`id`) )
    ENGINE = InnoDB;
    
# Reconfigure the frontend
# Add in the Dockerfile (before RUN pip3...)
    RUN apk add --update --no-cache mariadb-connector-c-dev \
        && apk add --no-cache --virtual .build-deps \
            mariadb-dev \
            gcc \
            musl-dev \
        && pip install mysqlclient==1.4.6 \
        && apk del .build-deps
        
# Add in the requirements.txt
    mysqlclient

    
    
