# Projet Docker - application : student-list

La consigne est accessible sur ce lien [ici](https://github.com/diranetafen/student-list.git "ici")

------------

Prénom : John

Nom de famille : VIEIRA

Pour le 20ème Bootcamp DevOps d'Eazytraining

Période : juillet-aout-septembre

Dimanche 11 aout 2024

LinkedIn : https://www.linkedin.com/in/johnvieira45/


------------

## Contexte

Dans le cadre du projet docker, une application (*student_list*) doit être mise en oeuvre pour la compagnie POZOS.

Cette application est composé de deux modules :

- Une API REST : Via une authentification simple, elle renvoit la liste souhaitée des étudiants basée sur un fichier JSON.
- Un site web : Il permet d'avoir un rendu graphique plus conviviable pour l'utilisateur des résultats de l'API.


## Objectifs

Les objectifs se comptent aux nombres de trois, à savoir :

- Construire le conteneur pour le module API et effectuer un test de validation.
- Faire du Infrastructure-as-Code (IaC) en utilisant docker-compose (on inclut le site web)
- Mise en place d'un registre privé

## Les fichiers

Concernant les fichiers de ce dépôt, il faut noter que ce sont les résultats des actions réalisées plus bas.

Composition du dépôt :

- docker-compose.yml          : fichier de déploiement pour l'API et l'application web.

- docker-compose_registry.yml : fichier de déploiement pour le registre privé en local.

- Dossier simple_api :

    - Dockerfile              : fichier d'instructions pour la construction de l'image de l'API.
    - requirements.txt        : liste des paquets Python nécessaire pour l'API.
    - student_age.json        : liste des étudiants au format JSON.
    - student_age.py          : code source au format Python de l'API.

- Dossier website :
    - index.php               : page PHP offrant une interface graphique pour l'utilisateur.

## Déroulement

La suite de ce document est l'ensemble des actions réalisées pour obtenir le dépôt actuel.

### Construction du conteneur API et test

Il faut cloner le dépôt initiale fournit dans la consigne

```bash
git clone https://github.com/diranetafen/student-list.git ./POZOS/
```

#### API REST

1) Changez de répertoire et alimentation du fichier *Dockerfile*

```bash
cd POZOS/simple_api/
vi Dockerfile
```







2) Construction de l'image du conteneur API

```bash
docker build . -t pozos_student_list_api_img
docker images
```
![docker5](https://github.com/user-attachments/assets/4cede5be-660b-4c91-a1b7-fcf707be1895)

3) Création d'un réseau de type pont pour permettre par la suite aux deux conteneurs de communiquer via le DNS

```bash
docker network create pozos_student_list_network --driver=bridge
docker network ls
```

![docker6](https://github.com/user-attachments/assets/7ef416ac-f7fa-4984-9f90-d09467295c03)


4) Lancez le conteneur en utilisant notre image


```bash
cd ..
docker run -d --name=pozos_student_list_api --network=pozos_student_list_network pozos_student_list_api_img
```

![docker1](https://github.com/user-attachments/assets/5d8beca6-09e7-488b-9080-1f5fb2d84ddb)


Le conteneur "pozos_student_list_api" est bien démarré et il écoute sur le port 5000.
En suivant les consignes, un volume persistant est créé afin d'y stocker le fichier *student_age.json*


![docker4](https://github.com/user-attachments/assets/0d2cdeea-e6a0-48b5-9a4b-1015a3e9e87a)



5) Tester l'API

Via une boucle for, on récupère l'adresse IP du conteneur API dans la log afin de pouvoir effectuer une commande CURL.

```bash
for i in $(docker logs pozos_student_list_api |& grep -oE "\b([0-9]{1,3}\.){3}[0-9]{1,3}:5000\b" |& uniq); do curl -u toto:python -X GET http://$i/pozos/api/v1.0/get_student_ages; done
```

![docker3](https://github.com/user-attachments/assets/5363153b-f3e7-42f1-887c-a1d17c30babc)


L'utilisateur et le mot de passe, à savoir toto/python sont fournis dans la consigne. Mais on peut les retrouver dans le code source de l'API (*student_age.py*)


![docker2](https://github.com/user-attachments/assets/d5282497-2ee6-4dcf-b691-7399197bc459)



A noter qu'il est possible de tester l'API également en créant un conteneur web ce qui permettrait de vérifier que le frontend arrive à afficher la liste des étudiants. Mais ceci n'étant pas dans la consigne, nous n'aborderons pas cette partie.


6) Suppression du conteneur API et nettoyage

Afin de préparer la suite, il est nécessaire de supprimer l'API et le réseau tout en conservant l'image créée.


```bash
docker rm -f pozos_student_list_api
docker network rm pozos_student_list_network
docker ps
docker network ls
```




### IaC via docker-compose

1) Alimenter le fichier docker-compose

```bash
vi docker-compose.yml
```

On retrouve en environnement l'utilisateur toto et son mot de passe.


2) Adapter le fichier de l'application web

Etant donné que nous avons défini un nom à notre conteneur API, nous allons l'utiliser pour notre site web.

```bash
for i in $(grep container_name ${PWD}/docker-compose.yml | grep api | cut -d: -f2); do sed -i "s/<api_ip_or_name:port>/$i:5000/g" ${PWD}/website/index.php ; done
```

3) Construction de l'infrastructure


```bash
docker-compose up -d
```
![docker7](https://github.com/user-attachments/assets/08921e4b-00ff-4f06-8b6a-f9152338444d)

Les conteneurs API et web sont bien créés.


![docker8](https://github.com/user-attachments/assets/d71d60fd-7e86-4604-9b16-e57f1daa15a3)




En utilisant un navigateur web et en pointant sur l'IP et le port 80, nous arrivons à obtenir le visuel.


![docker9](https://github.com/user-attachments/assets/8c741e86-f01f-4b43-b032-dd4bc620074d)


### Registre privé


1) Créer le fichier docker-compose dédié au registre privé

Pour une question de simplicité, le choix c'est porté sur l'image `registry:2` auquel il a été ajouté une interface web via l'image `joxit/docker-registry-ui`

```bash
vi docker-compose_registry.yml
```

Pour le conteneur du registre, on a définit un utilisateur avec mot de passe pour sécurisé au minimum l'accès.



2) Lancer le déploiement de notre registre privé

```bash
docker-compose -f /root/POZOS/docker-compose_registry.yml up -d
```
![docker10](https://github.com/user-attachments/assets/e5341009-43df-444f-8463-80943ad795cc)

Via un navigateur web, on vérifie que nous obtenons bien un rendu sur le port 8090

![docker11](https://github.com/user-attachments/assets/592893bb-03b4-45ce-8abf-93f049a1df9c)


3) Pousser l'image dans notre registre privé

Avant de pousser l'image, il faut renommer l'image

```bash
docker login localhost:5000
docker image tag pozos_student_list_api_img:latest localhost:5000/pozos_student_list_api_img:latest
docker image ls
docker image push localhost:5000/pozos_student_list_api_img:latest
```

![docker12](https://github.com/user-attachments/assets/fd31c04e-f94d-4178-a199-d52f1f71c1eb)


Enfin, dans le navigateur web on regarde sur le port 80 pour l'API si nous avons toujours un rendu.


![docker9](https://github.com/user-attachments/assets/fbd8e3d3-ea3d-4065-aa10-4fd1d4d4c6a1)


puis sur le port 8090 pour le registre et constater l'apparition de notre image dans le registre.

![docker13](https://github.com/user-attachments/assets/895cd479-6e8d-4da0-baee-bacb59caab93)

# Conclusion du projet Docker

Les différents points vu en cours ont put être mis en oeuvre de manière cohérentes.
