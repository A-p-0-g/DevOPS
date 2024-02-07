Grandhomme Jean-Baptiste

TP part 01 - Docker

Question 1-1 :
----------------
Création du fichier Dockerfile :
```dockerfile
FROM postgres:14.1-alpine

ENV 	POSTGRES_DB=db \
	POSTGRES_USER=usr \
	POSTGRES_PASSWORD=pwd

COPY * /docker-entrypoint-initdb.d
```
On crée un network "app-network" : 

docker network create app-network

Dans un dossier "/docker-entrypoint-initdb.d", on crée 2 fichiers sql permettant d'alimenter la bdd :

le fichier 01-CreateScheme.sql :
```sql
CREATE TABLE public.departments
(
 id      SERIAL      PRIMARY KEY,
 name    VARCHAR(20) NOT NULL
);

CREATE TABLE public.students
(
 id              SERIAL      PRIMARY KEY,
 department_id   INT         NOT NULL REFERENCES departments (id),
 first_name      VARCHAR(20) NOT NULL,
 last_name       VARCHAR(20) NOT NULL
);
```
et le fichier 02-InsertData.sql :
```sql
INSERT INTO departments (name) VALUES ('IRC');
INSERT INTO departments (name) VALUES ('ETI');
INSERT INTO departments (name) VALUES ('CGP');


INSERT INTO students (department_id, first_name, last_name) VALUES (1, 'Eli', 'Copter');
INSERT INTO students (department_id, first_name, last_name) VALUES (2, 'Emma', 'Carena');
INSERT INTO students (department_id, first_name, last_name) VALUES (2, 'Jack', 'Uzzi');
INSERT INTO students (department_id, first_name, last_name) VALUES (3, 'Aude', 'Javel');
```
Ensuite on ajoute ces fichiers au dockerfile avec la ligne : COPY * /docker-entrypoint-initdb.d

On build ensuite l'image avec la cmd : docker build -t jb/tpdocker .
On run ensuite le docker avec la cmd : docker run -d -v /my/own/datadir:/var/lib/postgresql/data --network app-network --name tpdocker jb/tpdocker
l'option "-v" permet d'avoir les données persistantes sur le disque de l'hôte
le network permet la connexion à la bdd / adminer

On run aussi adminer avec la cmd : docker run -p "8090:8080" --net=app-network --name=adminer -d adminer

On fait ensuite un "docker inspect tpdocker" afin de récupérer l'adresse IP permettant de se connecter à la bdd, dans notre cas c'est 172.18.0.2.

-----------------------
Question 1-2 :

```Dockerfile
# Build
FROM maven:3.8.6-amazoncorretto-17 AS myapp-build
ENV MYAPP_HOME /opt/myapp
WORKDIR $MYAPP_HOME
COPY pom.xml .
COPY src ./src
RUN mvn package -DskipTests
```

Dans cette partie, on importe maven avec un jdk, on crée une variable qui pointe vers /opt/myapp.
On se dirige ensuite vers ce dossier puis on copie pom.xml ainsi que src dans ce dossier puis on fait de maven un package.

```Dockerfile
# Run
FROM amazoncorretto:17
ENV MYAPP_HOME /opt/myapp
WORKDIR $MYAPP_HOME
COPY --from=myapp-build $MYAPP_HOME/target/*.jar $MYAPP_HOME/myapp.jar

ENTRYPOINT java -jar myapp.jar
```
Dans cette partie, on importe uniquement un jre puis on crée une variable pointant vers le chemin /opt/myapp.
Ensuite on se dirige vers ce dossier pour y copier le build créé précédemment.

Multistage build permet d'optimiser et d'alléger les images, comme cela on peut build avec un jdk puis lancer en run avec un jre pour optimiser au maximum.

----------------
Question 1.3 :
    docker-compose up : Lance les conteneurs selon la configuration spécifiée dans le fichier docker-compose.yml. Si aucun conteneur n'existe, il les crée et les lance. Si des conteneurs existent déjà, il les démarre.

    docker-compose down : Arrête et supprime les conteneurs, les réseaux, les volumes et autres ressources créées par docker-compose up.

    docker-compose build : Construit ou reconstruit les images Docker définies dans le fichier docker-compose.yml, en utilisant les instructions Dockerfile des services concernés.

    docker-compose start : Démarre les conteneurs déjà créés par docker-compose up, sans reconstruire les images ni recréer les conteneurs.

    docker-compose stop : Arrête les conteneurs déjà créés par docker-compose up, mais ne les supprime pas. Ils peuvent être redémarrés ultérieurement avec docker-compose start.

    docker-compose restart : Redémarre les conteneurs déjà créés par docker-compose up.

    docker-compose logs : Affiche les journaux (logs) de tous les services spécifiés dans le fichier docker-compose.yml. Pour limiter les journaux à un seul service, vous pouvez spécifier le nom du service après la commande, par exemple docker-compose logs backend.

    docker-compose ps : Affiche l'état des conteneurs définis dans le fichier docker-compose.yml, y compris les identifiants des conteneurs, les noms, les ports exposés, etc.

    docker-compose exec : Exécute une commande à l'intérieur d'un conteneur spécifique. Par exemple, pour exécuter une commande bash dans le conteneur backend, vous pouvez utiliser docker-compose exec backend bash.

    docker-compose down -v : En plus d'arrêter et de supprimer les conteneurs, cette commande supprime également les volumes associés. Cela peut être utile lorsque vous souhaitez supprimer complètement toutes les ressources créées par docker-compose up.
---------------------------
Question 1.4 :

```yml
version: '3.7'

services:
    backend:
        build: simple-api-student-main
        container_name: backend
        networks:
          - app-network
        depends_on:
          - database

    database:
        build: database
        container_name: database
        networks:
          - app-network
        volumes:
          - data:/var/lib/postgresql/data

    httpd:
        build: Frontend
        ports:
          - 80:80
        networks:
          - app-network
        depends_on:
          - backend

networks:
    app-network:
volumes:
    data:
```

-----------------------------
Question 1-5 : 

docker tag devops_backend:latest ap0g/devops_backend:1.0
docker tag devops_database:latest ap0g/devops_database:1.0
docker tag devops_httpd:latest ap0g/devops_httpd:1.0

docker push ap0g/devops_database:1.0
docker push ap0g/devops_backend:1.0
docker push ap0g/devops_httpd:1.0

-----------------------------------------

TP part 02 - GitHub

Question 2.1 :

Testcontainers est une bibliothèque Java qui fournit des instances légères et temporaires de bases de données courantes, de navigateurs web Selenium, ou de tout autre service pouvant s'exécuter dans un conteneur Docker pendant l'exécution de vos tests. Elle vous permet de configurer et de gérer facilement des conteneurs Docker dans le cadre de vos tests d'intégration.

Avec Testcontainers, vous pouvez définir et gérer des conteneurs Docker directement à partir de votre code Java, ce qui facilite le démarrage des instances de base de données, l'exécution de tests contre elles, et la suppression après coup, le tout dans le contexte de votre suite de tests.

En résumé, Testcontainers est un outil puissant pour simplifier la configuration et la suppression des environnements de test, en particulier lorsqu'il s'agit de dépendances telles que des bases de données ou d'autres services pouvant être conteneurisés. Il favorise la cohérence et la reproductibilité de vos tests d'intégration en veillant à ce que chaque test s'exécute dans un environnement propre et isolé.

Question 2.2 :
```yml
name: CI devops 2023
on:
  push:
    branches: 
      - main
      - develop
  pull_request:

jobs:
  test-backend: 
    runs-on: ubuntu-22.04
    steps:
      - uses: actions/checkout@v2.5.0
      - name: Set up JDK 17
        uses: actions/setup-java@v3
        with:
          java-version: '17'
          distribution: 'adopt'  # Ajout de la distribution de Java
      - name: Build and test with Maven
        run: mvn clean install
        working-directory: Docker/simple-api-student-main
```
Explication :

Ce fichier représente un workflow GitHub Actions nommé "CI devops 2023". Ce workflow est configuré pour être déclenché lorsqu'un push est effectué sur les branches main ou develop, ainsi que lorsqu'une pull request est ouverte.

Voici une explication détaillée du fichier :

1. **name**: Le nom du workflow GitHub Actions, qui est "CI devops 2023".

2. **on**: Cette section spécifie les événements qui déclenchent le workflow. Dans ce cas, le workflow est déclenché à chaque push sur les branches main ou develop, ainsi que lors de l'ouverture d'une pull request.

3. **jobs**: Cette section définit les tâches à exécuter dans le cadre du workflow. Dans ce cas, il y a une seule tâche appelée "test-backend".

4. **test-backend**: Cette tâche est configurée pour s'exécuter sur une machine virtuelle Ubuntu 22.04 (`runs-on: ubuntu-22.04`). Elle contient une liste d'étapes à suivre.

5. **steps**: Cette section contient une liste d'étapes à exécuter dans le cadre de la tâche "test-backend".

6. **actions/checkout@v2.5.0**: La première étape utilise l'action "checkout" pour récupérer le code source du référentiel GitHub.

7. **actions/setup-java@v3**: La deuxième étape  utilise l'action "setup-java" pour configurer JDK 17. L'option `java-version: '17'` spécifie la version de Java à utiliser, et `distribution: 'adopt'` spécifie la distribution de Java.

8. **run: mvn clean install**: La troisième étape exécute la commande `mvn clean install` pour nettoyer le projet Maven et construire l'application. Cette commande est exécutée dans le répertoire spécifié par `working-directory: Docker/simple-api-student-main`.

9. **working-directory: Docker/simple-api-student-main**: Cette option spécifie le répertoire dans lequel la commande `mvn clean install` sera exécutée. Cela garantit que la commande est exécutée dans le répertoire contenant le fichier `pom.xml` du projet Maven.

En résumé, ce workflow GitHub Actions configure une tâche qui nettoie et construit un projet Maven, en utilisant JDK 17, sur une machine virtuelle Ubuntu 22.04, déclenchée par des actions sur les branches main ou develop et les pull requests.