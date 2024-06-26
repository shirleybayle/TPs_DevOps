# Shirley Bayle - DevOps TPs

## TP1 - Docker part
### Database
Build the image created:
`docker build -t shirleybayle/my-postgres`
Run the myPostgres container with the right network and volume:
`docker run -p 8888:5432 --name myPostgres --network app-network -v C:\Users\shirl\Desktop\Cours\EPF\4A\DevOps\Postgres\Data:/var/lib/postgresql/data shirleybayle/my-postgres`
Create a network:
`docker network create app-network`
Run the adminer container:
`docker run -p "8080:8080" --net=app-network --name=adminer -d adminer`
Content of the Dockerfile:
```
FROM postgres:14.1-alpine

ENV POSTGRES_DB=db \
   POSTGRES_USER=usr \
   POSTGRES_PASSWORD=pwd 

COPY 01-CreateScheme.sql /docker-entrypoint-initdb.d/
COPY 02-InsertData.sql /docker-entrypoint-initdb.d/
```

### Backend API
```
# Build
FROM maven:3.8.6-amazoncorretto-17 AS myapp-build
ENV MYAPP_HOME /opt/myapp
WORKDIR $MYAPP_HOME
COPY pom.xml .
COPY src ./src
RUN mvn package -DskipTests

# Run
FROM amazoncorretto:17
ENV MYAPP_HOME /opt/myapp
WORKDIR $MYAPP_HOME
COPY --from=myapp-build $MYAPP_HOME/target/*.jar $MYAPP_HOME/myapp.jar

ENTRYPOINT java -jar myapp.jar
```

This docker file first build the simple api: it uses the maven image, then creates an environment variable called **MYAPP_HOME** containing the path */opt/myapp*, it changes the workdir with this new path, copy the files pom.xml and src to the container and finally run mvn package to create the .jar of the application. The second part runs the application from the build we just did, copying the .jar it finds in the build container to our new run container, end then specifizes the entry point of the container.
Multistage builds in Docker help make smaller and more secure images by separating the steps needed to build the application from those needed to run it.

### Docker-compose
The docker-compose file contains three different paragraphs that corresponds to the three different containers that will be created for this docker-compose. The first one is refering to the backend, the student api: we specify the path to the docker file of this container, we specify the network that should be the same for the three containers, and we specify that the backend depends on the database; it shouldn't be created before the database is created. Then, we have about the same commands for the database, and for the httpd server we also have a command to specify the port on which the server will be running. Finally we have a section for the network, specifying the name of the network.

The command to build the containers of the docker-compose:
`docker-compose build`

The command to run the three containers of the docker-compose:
`docker-compose up` 

### Publish
To publish our image with a tag, we need first to add a tag to our image:
`docker tag imagename shirleybayle/imagename:tag`
And then, to publish the image:
`docker push shirleybayle/imagename:tag`

## TP2 - Github actions part
Testcontainers help you create reliable and isolated environments for testing applications that depend on external systems, using Docker to manage these environments easily.

In this part, we did a workflows folder that allows github to run tests for us. In this folder, we created a main.yml file. It was important not to forget to specify the `java-version: '17'`and the `distribution: 'adopt'`, that we forgot at first. Except for this, we use actions to check our github code, and build our app with maven.

### SonarCloud
We created an account on sonarcloud, an organization and a project. Then, we linked the project to our code to test the quality gate.
The quality gate failed, it does not have a good grade. We can see that about 57 lines aren't covered by test, so about 46.4% of the code, it is not a good practice.

When we did the bonus, we splitted the worklow in **two different workflows**. The first workflow executes itself when we commit and **push** the code on the main branch on github, whereas the second is executing itself only when the first is **completed with success**.

## TP3 - Ansible part
To do this part, I used a ubuntu shell, imitating Linux for my system. 
The base command used for this part are:
`sudo ansible all -i /mnt/c/Users/shirl/Desktop/Cours/EPF/4A/DevOps/ansible/inventories/setup.yml -m setup -a "filter=ansible_distribution*"` this command request my server to get my OS distribution, thanks to the setup module.
`sudo ansible all -i /mnt/c/Users/shirl/Desktop/Cours/EPF/4A/DevOps/ansible/inventories/setup.yml -m yum -a "name=httpd state=absent" --become` this command removes the Apache server we installed earlier on the server.

# Playbook Documentation
This Ansible playbook automates the installation and configuration of Docker on target hosts. It consists of a series of tasks designed to install the required dependencies, add the Docker repository, install Docker, install Python 3 and the Docker Python library, and ensure that the Docker service is running.

Hosts: The playbook targets all hosts specified in the inventory file.
Gather Facts: Facts gathering is disabled (gather_facts: false) to improve playbook performance.
Privilege Escalation: Privilege escalation is enabled (become: true) to execute tasks with elevated privileges.
*Tasks:*
__Install device-mapper-persistent-data:__
Installs the device-mapper-persistent-data package using the yum module.
__Install lvm2:__
Installs the lvm2 package using the yum module.
__Add Docker Repository:__
Adds the Docker repository using the command module to execute the yum-config-manager command.
__Install Docker:__
Installs Docker CE (Community Edition) using the yum module.
__Install Python 3:__
Installs Python 3 using the yum module.
__Install Docker Python Library:__
Installs the Docker Python library (docker) using the pip module with Python 3.
__Ensure Docker Service is Running:__
Ensures that the Docker service is running using the service module with the state set to started.
*Tags:*
docker: Tasks related to Docker installation and configuration are tagged with docker. This allows for selective execution of tasks using tags.
*Usage:*
To execute this playbook, run the following command:
`ansible-playbook -i /path/to/inventory playbook.yml`
Replace /path/to/inventory with the path to your inventory file.
