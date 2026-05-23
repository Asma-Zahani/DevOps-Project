# LAB_HARBOR.md – Gestion des Images Docker avec Harbor

> **Étudiant C – Expert Harbor**
> **Projet : Construction d'une Chaîne CI/CD Sécurisée**
> Classe : Mastère CCDAD – M1 | Enseignant : Hedi MAGROUN

---

## Objectif du lab

Mettre en place un registre privé Docker avec Harbor afin de stocker, 
gérer et distribuer des images Docker dans une chaîne CI/CD sécurisée.

---

# 1. Le Besoin – Pourquoi utiliser Harbor ?

Harbor c'est pour stocker les images Docker de manière sécurisée,

docker utilise DockerHub qui est un registre public.
Harbor permet de créer un registre privé.

### Différence DockerHub vs Harbor

| DockerHub          | Harbor                          |
|--------------------|---------------------------------|
| registre public    | registre privé                  |
| hébergé par Docker | hébergé localement              |
| images publiques   | images privées sécurisées       |

---

# 2. Concepts Clés

### 2.1 Docker Image

c'est un modéle en lecture seule qui contient tout ce qu'il faut pour faire tourner 
une application (Dockerfile).

### 2.2 container

c'est une image qui tourne. c'est l'instance vivante de l'image on lance 
un container depuis l'image.

```bash
docker ps
```

pour voir les containers qui tourne.

### 2.3 registry

c'est un serveur de stockage d'image docker. c'est l'endroit ou on envoie les images
pour les conserver, les partager et les récupérer depuis n'importe quel machine.

**action : pull , push**

### 2.4 Harbor

harbor est un registre privé open-source développé pour les environnements DevOps et 
CI/CD.

### 2.5 Architecture de fonctionnement

```text
Dockerfile + code source
            │
            ▼
docker build
(creation image Docker)
            │
            ▼
Harbor Registry
(stockage image)
            │
            ▼
docker pull
(téléchargement image)
            │
            ▼
container Docker exécuté
```

---

# 3. Mise en Place Technique – Installation

## 3.1 Prérequis

- Docker installé
- Docker Compose installé

verification :

```bash
docker --version
docker compose version
```
---

## 3.2 Téléchargement de Harbor

donc d'apres github j'ai installer Harbor:

```bash
wget https://github.com/goharbor/harbor/releases/download/v2.14.4/harbor-online-installer-v2.14.4.tgz
```
![Docker installe + Harbor](images/harbor/docker-installe-harbor-git.png)
extraction :

```bash
tar xvf harbor-online-installer-v2.14.4.tgz
```
![Extract harbor](images/harbor/extract-Harbor.png)
---

## 3.3 Configuration Harbor

création du fichier de configuration :

```bash
cp harbor.yml.tmpl harbor.yml
```

le fichier `harbor.yml.tmpl` représente le modèle de configuration fourni par Harbor.

le fichier `harbor.yml` correspond à la configuration réelle utilisée lors de l’installation.

modification :

```bash
nano harbor.yml
```

configuration utilisée :

```yaml
hostname: @IP

http:
  port: 8088
```
![configuration 1](images/harbor/config-Harbor1.png)
![Extract harbor1](images/harbor/config-Harbor2.png)

dans harbor.yml j'ai décommenté la partie de https, car lors de poursuivre la 
documentation d'instalatio harbor il m'a donné une etape d'utiliser HTTPS 
mais il est pour les grands entreprise puisque je vais travailler dans mon local
je vais utiliser HTTP pour simplifier les choses.

---

## 3.4 Installation Harbor

```bash
sudo ./install.sh
```
![Instalation Harbor](images/harbor/instalation-harbor.png)

verification des conteneurs :

```bash
docker ps
```

aussi puisque j'utilise http donc il faut ajouter harbor comme insecure registry
car Docker refuse de communiquer avec un registre non sécurisé.

dans `/etc/docker/daemon.json` :

```json
{ "insecure-registries" : ["@IP:8088"]}
```
![Insecure registry](images/harbor/insecure-registry.png)

donc apres ca j'ai redemarer docker

apres l'instalation de nouveau se reussi
j'ai acceder a l'interface web avec:

```text
http://@IP:8088
```

### Interface Harbor
![Interface Harbor](images/harbor/dashboard-harbor-apres-instalation.png)
---

# 4. Création d’un Projet Harbor

depuis l’interface web Harbor :

```text
Projects → New Project
```

Projet créé :

```text
dev-project
```

Type :

```text
Private
```

### Projet Harbor créé
![Projet Harbor](images/harbor/cree-projet-harbor.png)
---

# 5. Création et Push d’une Image Docker

## 5.1 Création d’une application simple

Fichier `index.html` :

```html
<h1>salut Harbor CI/CD Projet</h1>
<p>Docker image cree par moi Nour</p>
```

---

## 5.2 Dockerfile

```dockerfile
FROM nginx:alpine

COPY index.html /usr/share/nginx/html/index.html
```

---

## 5.3 Build de l’image Docker

```bash
docker build -t myapp:v1 .
```

### Build image Docker
![Docker Build](images/harbor/creation-app-simple-build.png)
![Application tournée](images/harbor/app.png)
---

## 5.4 Login Harbor

```bash
docker login @IP:8088
```

---

## 5.5 Tag de l’image

```bash
docker tag myapp:v1 @IP:8088/dev-project/myapp:v1
```
![Login + tag](images/harbor/login-tag-image.png)
---

## 5.6 Push de l’image vers Harbor

```bash
docker push @IP:8088/dev-project/myapp:v1
```

Après le push, l’image apparaît dans :

```text
Projects → dev-project → Repositories
```

### Push image vers Harbor
![Docker Push](images/harbor/push_image-harbor.png)
### Harbor Repository
![Harbor Repository](images/harbor/push-dash-harbor.png)
---

## 5.7 Supprimer l’image de chez moi

=> j'ai supprimer l'image cree pour tester le pull

```bash
docker rmi myapp:v1
```

---

## 5.8 Pull de l’image

```bash
docker pull @IP:8088/dev-project/myapp:v1
```

### Pull image depuis Harbor

![Docker Pull](images/harbor/supp-pull-image.png)
---

# 6. Captures d’écran

Les captures d’écran utilisées dans ce lab se trouvent dans :

```text
images/harbor/
```

---

# 7. Documentation Officielle

- Harbor Documentation : https://goharbor.io/docs/
- Docker Documentation : https://docs.docker.com/

---

# 8. Intégration CI/CD avec Jenkins + Docker + Harbor

## Objectif

Automatiser grâce à Jenkins Pipeline :

- le build du projet
- la création de l'image Docker
- le push automatique vers Harbor

---

## 8.1 Architecture CI/CD finale

```text
Développeur
    │
    ▼
git push
    │
    ▼
Gitea
    │ webhook
    ▼
Jenkins Pipeline
    │
    ├── Maven Build
    ├── SonarQube Analysis
    ├── Nexus Deploy
    ├── Docker Build
    └── Push Harbor
```

---

## 8.2 Dockerfile du Projet Spring Boot

Le projet Spring Boot génère un fichier `.war`.
Tomcat est utilisé pour exécuter ce fichier.

### Dockerfile utilisé

```dockerfile
FROM tomcat:10.1
COPY target/*.war /usr/local/tomcat/webapps/ROOT.war
EXPOSE 8080
```

### Explication

| Instruction | Rôle |
|---|---|
| `FROM tomcat:10.1` | Utilisation de l'image officielle Tomcat, un serveur Java permettant d'exécuter les applications `.war` |
| `COPY target/*.war` | Copie le fichier `.war` généré par Maven dans le dossier `webapps` de Tomcat |
| `ROOT.war` | Permet d'accéder directement à l'application depuis `http://ip-machine:8080` sans écrire le nom du projet |
| `EXPOSE 8080` | Ouvre le port 8080 du conteneur |

---

## 8.3 Jenkins Pipeline

Le pipeline Jenkins automatise toutes les étapes DevOps.

### Jenkinsfile utilisé

```groovy
pipeline {
   
        stage('Docker Build') {
            steps {
                sh 'docker build -t reservia-app:v1 app'
            }
        }
        stage('Push Harbor') {
            steps {
                sh 'docker tag reservia-app:v1 @IP:8088/dev-project/reservia-app:v1'
                sh 'docker push @IP:8088/dev-project/reservia-app:v1'
            }
        }
    
```

---

## 8.4 Explication des stages Jenkins

| Stage | Rôle |
|---|---|
| `Docker Build` | Crée l'image Docker à partir du Dockerfile |
| `Push Harbor` | Tag et push l'image Docker vers le registre Harbor |

---

## 8.5 Login Harbor depuis Jenkins

Avant le push Harbor, Jenkins doit être authentifié auprès du registre.

Se connecter avec l'utilisateur Jenkins :

```bash
sudo su - jenkins
docker login @IP:8088
```


