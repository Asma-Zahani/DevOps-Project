# CI-CD-Project

Projet CI/CD pédagogique pour démonstration universitaire.

## 1. Description projet

Architecture proposée:

```
Gitea ↓ Jenkins ↓ Maven ↓ SonarQube ↓ Nexus ↓ Docker ↓ Harbor
```

Ce repository contient :
- l'application Spring Boot (packaging WAR) dans le dossier `app/`
- fichiers CI/CD : `Jenkinsfile`, `docker-compose.yml`
- documentation et labs dans `docs/`

## 2. Prérequis

- Ubuntu 22.04
- Docker
- Docker Compose
- Java 17
- Maven

## 3. Trouver IP machine

Exécuter :

```bash
ip a
```

## 4. Remplacement global @IP

Remplacer tous les `@IP` par votre IP :

```bash
grep -rl '@IP' . | xargs sed -i 's/@IP/VOTRE_IP/g'
```

## 5. Paramètre kernel SonarQube

```bash
sudo sysctl -w vm.max_map_count=262144
```

## 6. Installation Docker (Ubuntu)

```bash
sudo apt update
sudo apt install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add the repository to Apt sources:
sudo tee /etc/apt/sources.list.d/docker.sources <<EOF
Types: deb
URIs: https://download.docker.com/linux/ubuntu
Suites: $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}")
Components: stable
Signed-By: /etc/apt/keyrings/docker.asc
EOF
sudo apt update
sudo apt install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
sudo usermod -aG docker $USER
```

## 7. Installation Java 17

```bash
sudo apt update
sudo apt install -y openjdk-17-jdk
```

## 8. Installation Maven

```bash
sudo apt update
sudo apt install -y maven
```

## 9. Lancement plateforme

Ordre obligatoire :

1. Lancer la plateforme core (Jenkins, SonarQube, Nexus, MySQL) :

```bash
docker compose up -d
```

2. Installer Harbor séparément (voir `docs/INSTALL_HARBOR.md`).
3. Configurer Jenkins (plugins, credentials, SonarQube server, M3 tool).

## 10. Tableau des accès

| Service   | URL             |
| --------- | --------------- |
| Jenkins   | http://@IP:8080 |
| SonarQube | http://@IP:9000 |
| Nexus     | http://@IP:8081 |
| Harbor    | http://@IP:8082 |

## 11. Nexus password initial

```bash
docker exec nexus cat /nexus-data/admin.password
```

## 12. Configuration Jenkins

Plugins recommandés :
- SonarQube Scanner
- Docker Pipeline
- Pipeline

Credentials à créer :
- `sonar-token` (token Sonar)
- `nexus-credentials` (user/password Nexus)
- `harbor-credentials` (user/password Harbor)

SonarQube Server dans Jenkins :
- Nom: `SonarQube`
- URL: `http://@IP:9000`

Maven (Tools) :
- Nom: `M3`

## 13. settings.xml Maven

Créer `~/.m2/settings.xml` avec :

```xml
<settings>
  <servers>
    <server>
      <id>nexus-snapshots</id>
      <username>admin</username>
      <password>password</password>
    </server>
  </servers>
</settings>
```

## 14. Permissions Docker Jenkins

```bash
sudo usermod -aG docker jenkins
```

## 15. Build manuel Docker

```bash
docker build -t reservia-app:v1 app
```

## 16. Push Harbor manuel

```bash
docker login @IP:8082
docker tag reservia-app:v1 @IP:8082/reservia/reservia-app:v1
docker push @IP:8082/reservia/reservia-app:v1
```
