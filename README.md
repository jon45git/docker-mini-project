# Docker project - application: student-list

The instruction is accessible on this link [here](https://github.com/diranetafen/student-list.git "here")

------------

First name : John

Surname : VIEIRA

For the 20th Bootcamp DevOps from Eazytraining

Period : July-August-September

Sunday 11 August 2024

LinkedIn : https://www.linkedin.com/in/johnvieira45/


------------

## Background

Under the docker project, a study-list application must be implemented for the company PO-OS.

This application consists of two modules:

    - A REST API: Through simple authentication, it returns the desired list of students based on a JSON file.
    - A website: It allows a graphical rendering that is more convenient for the user of the results of the API results.




## Objectives

The objectives are three, namely:

    - Build the container for the API module and perform a validation test.
    - Make Infrastructure-as-Code (IaC) using docker-compose (we include the website)
    - Establishment of a private register


## Files

With regard to the files of this repository, it should be noted that these are the results of the actions carried out below.

Composition of the deposition:

    - docker-compose.yml: deployment file for the API and web application.

    - docker-compose-registry.yml: deployment file for the local private registry.

    - Simple-api file:
        - Dockerfile: file of instructions for the construction of the image of the API.
        - requirements.txt: list of Python packets required for the API.
        - student.json: list of students in JSON format.
        - student-age.py: source code in Python format of the API.

    - Website:
        - index.php: PHP page providing a graphical interface for the user.


## Progress

The remainder of this document is the set of actions taken to obtain the current deposit.

### Construction of the API container and test

The initial deposit provided in the instruction should be cloned.

```bash
git clone https://github.com/diranetafen/student-list.git ./POZOS/
```

#### API REST

1) Change the directory and power supply of the *Dockerfile* file

```bash
cd POZOS/simple_api/
vi Dockerfile
```



2) Construction of the Image of the API Container

```bash
docker build . -t pozos_student_list_api_img
docker images
```
![docker5](https://github.com/user-attachments/assets/4cede5be-660b-4c91-a1b7-fcf707be1895)

3) Creation of a bridge-type network to subsequently allow both containers to communicate via the DNS

```bash
docker network create pozos_student_list_network --driver=bridge
docker network ls
```

![docker6](https://github.com/user-attachments/assets/7ef416ac-f7fa-4984-9f90-d09467295c03)


4) Launch the container using our image


```bash
cd ..
docker run -d --name=pozos_student_list_api --network=pozos_student_list_network pozos_student_list_api_img
```

![docker1](https://github.com/user-attachments/assets/5d8beca6-09e7-488b-9080-1f5fb2d84ddb)

The container "pozos-student_list_api" is well started and it is listening on the port 5000. By following the instructions, a persistent volume is created in order to store therein the *student-age.json* file.

![docker4](https://github.com/user-attachments/assets/0d2cdeea-e6a0-48b5-9a4b-1015a3e9e87a)



5) Testing the API

Via a loop for, the IP address of the API container is recovered in the log in order to be able to perform a CURL command.

```bash
for i in $(docker logs pozos_student_list_api |& grep -oE "\b([0-9]{1,3}\.){3}[0-9]{1,3}:5000\b" |& uniq); do curl -u toto:python -X GET http://$i/pozos/api/v1.0/get_student_ages; done
```

![docker3](https://github.com/user-attachments/assets/5363153b-f3e7-42f1-887c-a1d17c30babc)

The user and the password, i.e. toto/python are provided in the deposit. But we can find them in the source code of the API (*student_age.py*)


![docker2](https://github.com/user-attachments/assets/d5282497-2ee6-4dcf-b691-7399197bc459)



It should be noted that it is possible to test the API also by creating a web container, which would make it possible to verify that the frontend manages to display the list of students. But this is not in the instruction, we will not address this part.

6) API container removal and cleaning

In order to prepare the rest, it is necessary to delete the API and the network while retaining the image created.


```bash
docker rm -f pozos_student_list_api
docker network rm pozos_student_list_network
docker ps
docker network ls
```




### IaC via docker-compose

1) Fill the docker-compose file

```bash
vi docker-compose.yml
```

The user toto and his password are found in the environment.


2) Adapt the web application file

Since we have defined a name to our API container, we will use it for our website.

```bash
for i in $(grep container_name ${PWD}/docker-compose.yml | grep api | cut -d: -f2); do sed -i "s/<api_ip_or_name:port>/$i:5000/g" ${PWD}/website/index.php ; done
```

3) Create of infrastructure


```bash
docker-compose up -d
```
![docker7](https://github.com/user-attachments/assets/08921e4b-00ff-4f06-8b6a-f9152338444d)

API and web containers are well created.


![docker8](https://github.com/user-attachments/assets/d71d60fd-7e86-4604-9b16-e57f1daa15a3)




En utilisant un navigateur web et en pointant sur l'IP et le port 80, nous arrivons à obtenir le visuel.


![docker9](https://github.com/user-attachments/assets/8c741e86-f01f-4b43-b032-dd4bc620074d)


### Private register


1) Create the docker-compose file dedicated to the private register

For a question of simplicity, the choice is focused on the image `registry:2`to which a web interface has been added via the image `joxit/docker-registry-ui`

```bash
vi docker-compose_registry.yml
```

For the registry container, a user with a password has been defined for at least access.



2) Launching the deployment of our private registry

```bash
docker-compose -f /root/POZOS/docker-compose_registry.yml up -d
```
![docker10](https://github.com/user-attachments/assets/e5341009-43df-444f-8463-80943ad795cc)

Via a web browser, we check that we get a rendering on port 8090

![docker11](https://github.com/user-attachments/assets/592893bb-03b4-45ce-8abf-93f049a1df9c)


3) Pushing the image into our private register

Before pushing the image, the image must be renamed

```bash
docker login localhost:5000
docker image tag pozos_student_list_api_img:latest localhost:5000/pozos_student_list_api_img:latest
docker image ls
docker image push localhost:5000/pozos_student_list_api_img:latest
```

![docker12](https://github.com/user-attachments/assets/fd31c04e-f94d-4178-a199-d52f1f71c1eb)


Finally, in the web browser we look at port 80 for the API if we still have a rendering.


![docker9](https://github.com/user-attachments/assets/fbd8e3d3-ea3d-4065-aa10-4fd1d4d4c6a1)


then on port 8090 for the register and see the appearance of our image in the register.

![docker13](https://github.com/user-attachments/assets/895cd479-6e8d-4da0-baee-bacb59caab93)

# Conclusion of the Docker project

The various points seen in the course were able to be implemented in a coherent manner.
