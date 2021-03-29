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
