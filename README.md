
<div align="center">

![Docker Build](https://img.shields.io/badge/docker-build-green)       ![Docker Build](https://img.shields.io/badge/dockercompose-up-green)

![Static Badge](https://img.shields.io/badge/Docker-2CA5E0?style=for-the-badge&logo=docker&logoColor=white)        ![Flask](https://img.shields.io/badge/Flask-000000?style=for-the-badge&logo=flask&logoColor=white)        ![Nginx](https://img.shields.io/badge/nginx-%23009639.svg?style=for-the-badge&logo=nginx&logoColor=white)         ![Python](https://img.shields.io/badge/Python-14354C?style=for-the-badge&logo=python&logoColor=white)         ![PHP](https://img.shields.io/badge/PHP-777BB4?style=for-the-badge&logo=php&logoColor=white)             

</div>
<div align="center">
Conteneurisation d'une webApp composée d'un serveur web (PHP) et d'une API (Flask)

![project](https://user-images.githubusercontent.com/18481009/84582395-ba230b00-adeb-11ea-9453-22ed1be7e268.jpg)
</div>

# Application

L'application permet d'afficher sur une page web, une liste d'étudiants avec leur nom et age.
Les données sont stockées dans un fichier JSON.
Une API permet de renvoyer les données JSON au frontend.

<strong>Elle est composée de 2 modules :</strong>

1. API REST => renvoi une liste d'étudiants à partir d'un fichier JSON (authentification requise)
2. Web APP => une page PHP permettant d'afficher la liste des étudiants

Chaque module est conteneurisé puis déployé en local avec le docker-compose.yml

L'API est accessible via cette URL :
http://<api_ip_or_name:port>/pozos/api/v1.0/get_student_ages


# Fichiers

- docker-compose.yml: déploiement de l'application (API et frontend)
- Dockerfile: Conteneurisation de l'API Rest
- student_age.json: contient la liste des étudiants avec leur nom et age au format JSON
- student_age.py: code source python de l'API
- index.php: Page affichant le frontend et envoie la requête au backend


# Builder l'image

Exécutez le fichier Dockerfile /simple_api/Dockerfile pour conteneuriser l'API

Image Base => python:2.7-buster <br/>
Copie du code source => COPY . /<br/>
Création d'un volume afin de monter les données JSON => VOLUME /data <br/>
Exposition du conteneur => port 5000

<strong>- Builder</strong>
```
Docker build -t api-service .
```

![screen](https://github.com/Tony-Dja/Docker-API-webapp/blob/e12bb5924d71eef708ffdec59bfae04766c943ea/screenshots/build-api-service.png)

<strong>- Vérifier les images</strong>
```
docker images
```

![screen](https://github.com/Tony-Dja/Docker-API-webapp/blob/e12bb5924d71eef708ffdec59bfae04766c943ea/screenshots/docker-images.png)


# Lancer le conteneur

Les données JSON sont chargées au lancement en spécifiant un volume BindMount -v pointant sur le fichier "student_age.json"

<strong>- Lancement</strong>

=> docker run -d --name container-api-service -p 5000:5000 -v /home/tonydja/formation/ci-cd/Docker-API-webapp/simple_api/student_age.json:/data/student_age.json api-service

![screen](https://github.com/Tony-Dja/Docker-API-webapp/blob/e12bb5924d71eef708ffdec59bfae04766c943ea/screenshots/docker-run.png)

<strong>- Tester le conteneur</strong>

=> curl -u toto:python -X GET http://<api_ip_or_name:port>/pozos/api/v1.0/get_student_ages

exemple : curl -u toto:python -X GET http://localhost:5000/pozos/api/v1.0/get_student_ages

![screen](https://github.com/Tony-Dja/Docker-API-webapp/blob/e12bb5924d71eef708ffdec59bfae04766c943ea/screenshots/curl-api.png)

<strong>- Supprimer le conteneur</strong>

Ensuite supprimer le conteneur pour pouvoir le déployer avec le frontend en utilisant le docker-compose.yml

=> docker container stop container-api-service <br/>
=> docker container rm container-api-service


## Déploiement

Pour déployer le frontend et l'API en backend nous allons exécuter le fichier docker-compose.yml

<strong>Il contient 2 services :</strong>

=> frontend<br/>
=> api

Le service frontend dépend du service api. Il ne peut pas démarrer si l'API n'est pas encore active.<br/>
Il est exposé sur le port 80.

=> depends_on:<br/>
      - api

<strong>- Lancement de l'application</strong>

=> docker compose up

![screen](https://github.com/Tony-Dja/Docker-API-webapp/blob/ec950f89772dab7f02253184a910a7dd3127a693/screenshots/docker-compose-up.png)


Vérifiez que l'API est bien accessible via cette URL :<br/>
http://localhost:5000/pozos/api/v1.0/get_student_ages

Pour accéder à l'application (frontend) :<br/>
http://localhost


![screen](https://github.com/Tony-Dja/Docker-API-webapp/blob/929ee6c621a724e0a673456cd359702c65d411f7/screenshots/frontend.png)




- Base image

To build API image you must use "python:2.7-buster"

- Maintainer

Please don't forget to specify the maintainer information

- Add the source code

You need to copy the source code of the API in the container at the root "/" path

- Prerequisite

The API is using FLASK engine,  here is a list of the package you need to install
```
apt update -y && apt install python-dev python3-dev libsasl2-dev python-dev libldap2-dev libssl-dev -y
pip install flask==1.1.2 flask_httpauth==4.1.0 flask_simpleldap python-dotenv==0.14.0
```
- Persistent data (volume)

Create data folder at the root "/" where data will be stored and declare it as a volume

You will use this folder to mount student list

- API Port

To interact with this API expose 5000 port

- CMD

When container start, it must run the student_age.py (copied at step 4), so it should be something like

`CMD [ "python", "./student_age.py" ]`

Build your image and try to run it (don't forget to mount *student_age.json* file at */data/student_age.json* in the container), check logs and verify that the container is listening and is  ready to answer

Run this command to make sure that the API correctly responding (take a screenshot for delivery purpose)

`curl -u toto:python -X GET http://<host IP>:<API exposed port>/pozos/api/v1.0/get_student_ages`

**Congratulation! Now you are ready for the next step (docker-compose.yml)**

## Infrastructure As Code (5 points)

After testing your API image, you need to put all together and deploy it, using docker-compose.

The ***docker-compose.yml*** file will deploy two services :

- website: the end-user interface with the following characteristics
   - image: php:apache
   - environment: you will provide the USERNAME and PASSWORD to enable the web app to access the API through authentication
   - volumes: to avoid php:apache image run with the default website, we will bind the website given by POZOS to use. You must have something like
`./website:/var/www/html`
   - depend on: you need to make sure that the API will start first before the website
   - port: do not forget to expose the port
- API: the image builded before should be used with the following specification
   - image: the name of the image builded previously
   - volumes: You will mount student_age.json file in /data/student_age.json
   - port: don't forget to expose the port

Delete your previous created container

Run your docker-compose.yml

Finally, reach your website and click on the bouton "List Student"

**If the list of the student appears, you are successfully dockerizing the POZOS application! Congratulation (make a screenshot)**

## Docker Registry (4 points)

POZOS need you to deploy a private registry and store the built images

So you need to deploy :

- a docker [registry](https://docs.docker.com/registry/ "registry")
- a web [interface](https://hub.docker.com/r/joxit/docker-registry-ui/ "interface") to see the pushed image as a container

Or you can use [Portus](http://port.us.org/ "Portus") to run both

Don't forget to push your image on your private registry and show them in your delivery.

## Delivery (4 points)

Your delivery must be zip named firstname.zip (replace firstname by your own) that contain:

- A doc or PDF file with your screenshots and explanations.
- Configuration files used to realize the graded exercise (docker-compose.yml and Dockerfile).

Your delivery will be evaluated on:

- Explanations quality
- Screenshots quality (relevance, visibility)
- Presentation quality

Send your delivery at ***eazytrainingfr@gmail.com*** and we will provide you the link of the solution.

![good luck](https://user-images.githubusercontent.com/18481009/84582398-cad38100-adeb-11ea-95e3-2a9d4c0d5437.gif)
