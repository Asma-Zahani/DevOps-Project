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
```
Ajouter utilisateur au groupe docker :
```
sudo usermod -aG docker $USER
```
Reconnecter ensuite la session.

## 7. Installation Java 17

```bash
sudo apt update
sudo apt install -y openjdk-17-jdk
```
Vérification :
```
java -version
```

## 8. Installation Maven

```bash
sudo apt update
sudo apt install -y maven
```
Vérification :
```
mvn -version
```

## 9. Lancement plateforme CI/CD

Démarrer les services principaux :
```
docker compose up -d
```
Services lancés :

- Jenkins
- SonarQube
- Nexus

## 10. Installation Harbor

Installer Harbor séparément :
```
docs/LAB_HARBOR.md
```
Après chaque modification du fichier harbor.yml :

```
cd harbor
sudo ./prepare
sudo docker compose down
sudo docker compose up -d
```

## 11. Tableau des accès

| Service   | URL             |
| --------- | --------------- |
| Jenkins   | http://@IP:8080 |
| SonarQube | http://@IP:9000 |
| Nexus     | http://@IP:8081 |
| Harbor    | http://@IP:8082 |

## 12. Nexus password initial

```bash
docker exec nexus cat /nexus-data/admin.password
```


## 12. Configuration Jenkins

### Plugins recommandés :
- Pipeline
- Docker Pipeline
- SonarQube Scanner
- Maven Integration

### Credentials Jenkins

Créer :
| ID                     | Type              |
| ---------------------- | ----------------- |
| sonar-token            | Secret text       |
| nexus-credentials      | Username/password |
| harbor-credentials     | Username/password |

### SonarQube Server dans Jenkins :
```
Manage Jenkins → System → SonarQube servers
```
Ajouter :
| Champ    | Valeur            |
| -------- | ----------------- |
| Nom      | `SonarQube`       |
| URL      | `http://@IP:9000` |

### Maven Tool
```
Manage Jenkins → Tools
```
Ajouter :
| Champ    | Valeur            |
| -------- | ----------------- |
| Nom      | `M3`              |

## 14. Configuration de `settings.xml` pour Maven

Créez un fichier `settings.xml` avec le contenu suivant :

```xml
<settings>
  <servers>
    <server>
      <id>nexus</id>
      <username>admin</username>
      <password>password</password>
    </server>
  </servers>
</settings>
```

- Pour un utilisateur local placez le fichier dans :
```
~/.m2/settings.xml
```
- Pour Jenkins placez le fichier dans :
```
/var/lib/jenkins/.m2/settings.xml
```
et assurez-vous que Jenkins en est propriétaire :
```
sudo chown -R jenkins:jenkins /var/lib/jenkins/.m2
```

## 15. Permissions Docker Jenkins

```bash
sudo usermod -aG docker jenkins
```
Puis :
```
sudo systemctl restart jenkins
```

## 16. Configuration Docker insecure registry (Harbor HTTP)

Créer/modifier :
```
sudo nano /etc/docker/daemon.json
```
Ajouter :
```
{
  "insecure-registries": ["@IP:8082"]
}
```
Puis :
```
sudo systemctl restart docker
```
