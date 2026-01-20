# Gestion Produits

## Description
Projet Spring Boot pour gérer des produits avec interface web, base de données et tests automatisés.

**Fonctionnalités** :
* CRUD complet (Créer, Lire, Modifier, Supprimer)
* Interface web avec Thymeleaf
* API REST documentée (Swagger)
* Tests automatisés (Unitaires, Intégration, Selenium)
* Containerisation Docker
* Pipeline CI/CD avec Jenkins
* Déploiement Kubernetes
* Monitoring avec Prometheus et Grafana

**Technologies** :
* Java 17, Spring Boot 3.x, Spring Data JPA
* MySQL (production) / H2 (tests)
* Thymeleaf, Bootstrap
* JUnit 5, Mockito, Selenium WebDriver
* Docker, Kubernetes, Jenkins
* Prometheus, Grafana, Actuator
* Swagger (springdoc-openapi)

---

## Prérequis

* Java 17
* Maven 3.8+
* Docker & Docker Compose
* Chrome (pour tests Selenium)
* Minikube (pour Kubernetes)
* kubectl (pour déploiement)

---

## Installation

### 1. Cloner le projet
```bash
git clone https://github.com/SalmaElFathi/gestion-produits-spring.git
cd gestion-produits
```

### 2. Configurer l'environnement
```bash
cp .env.example .env
# Éditer .env avec vos valeurs
```

### 3. Lancer avec Docker Compose

⚠️ **Note** : Docker Compose est utilisé uniquement pour les tests locaux.

```bash
docker-compose up --build
```

L'application sera accessible sur : **http://localhost:5000**

---

## Tests

Le projet contient **3 types de tests** :

| Type | Description | Commande |
|------|-------------|----------|
| **Unitaires** | Logique métier (Mockito) | `mvn test -Dgroups=Unitaire` |
| **Intégration** | Couches applicatives (H2) | `mvn test -Dgroups=Integration` |
| **Selenium** | Tests end-to-end (Chrome) | `mvn verify -Dgroups=selenium` |

**Lancer tous les tests** :
```bash
mvn verify
```

---

## Documentation API

**Swagger UI** : http://localhost:5000/swagger-ui.html

**Endpoints Actuator** :
* Health : http://localhost:5000/actuator/health
* Prometheus : http://localhost:5000/actuator/prometheus

---

## Configuration Jenkins

### 1. Installation de Jenkins

```bash
# Lancer Jenkins dans un conteneur Docker
docker run -d --name jenkins -p 8080:8080 -p 50000:50000 -v jenkins_home:/var/jenkins_home jenkins/jenkins:lts
```

### 2. Récupérer le mot de passe initial

```bash
docker exec jenkins cat /var/jenkins_home/secrets/initialAdminPassword
```

### 3. Installer Chrome et Docker dans Jenkins

```bash
# Accéder au conteneur Jenkins
docker exec -it -u root jenkins bash

# Installer Chrome et Docker (dans le conteneur)
wget -q -O - https://dl-ssl.google.com/linux/linux_signing_key.pub | gpg --dearmor -o /usr/share/keyrings/google-chrome.gpg && \
echo "deb [arch=amd64 signed-by=/usr/share/keyrings/google-chrome.gpg] http://dl.google.com/linux/chrome/deb/ stable main" > /etc/apt/sources.list.d/google-chrome.list && \
apt-get update && \
apt-get install -y google-chrome-stable docker.io && \
usermod -aG docker jenkins && \
google-chrome --version && \
docker --version && \
echo "✅ Installation réussie !"

# Quitter le conteneur
exit

# Redémarrer Jenkins
docker restart jenkins
```

### 4. Configurer les outils dans Jenkins

**Aller dans** : `Manage Jenkins` → `Global Tool Configuration`

| Outil | Configuration | Notes |
|-------|--------------|-------|
| **Maven** | Nom : `Maven` | Installation automatique activée |
| **JDK** | Nom : `JDK17` | Installation automatique activée |
| **Git** | Nom : `Git` | Installation par défaut |

### 5. Configurer les credentials dans Jenkins

**Aller dans** : `Manage Jenkins` → `Credentials` → `System` → `Global credentials`

| Credential | Type | ID | Description |
|-----------|------|-----|-------------|
| **GitHub** | Username/Password | `github-credentials` | Token GitHub personnel |
| **Docker Hub** | Username/Password | `dockerhub-credentials` | Identifiants Docker Hub |

### 6. Créer le pipeline

1. Créer un nouveau job : **New Item** → **Pipeline**
2. Dans **Pipeline** → **Definition** : choisir `Pipeline script from SCM`
3. **SCM** : Git
4. **Repository URL** : `https://github.com/SalmaElFathi/gestion-produits-spring.git`
5. **Credentials** : Sélectionner `github-credentials`
6. **Script Path** : `Jenkinsfile`

### 7. Personnaliser le Jenkinsfile

⚠️ **Important** : Modifier le nom de l'image Docker dans le `Jenkinsfile` :

```groovy
environment {
    DOCKER_IMAGE = 'VOTRE-USERNAME-DOCKERHUB/gestion-produits'  // ← Changer ici
}
```

---

## Configuration Kubernetes

⚠️ **Note** : Cette configuration est pour un environnement de développement local avec Minikube.

### 1. Installer Minikube

```bash
# Télécharger Minikube
# https://minikube.sigs.k8s.io/docs/start/

# Démarrer Minikube avec Docker
minikube start --driver=docker
```

### 2. Installer kubectl dans Jenkins

```bash
# Installer kubectl dans le conteneur Jenkins
docker exec -u root jenkins sh -c "curl -LO https://dl.k8s.io/release/v1.28.0/bin/linux/amd64/kubectl && chmod +x kubectl && mv kubectl /usr/local/bin/"

# Vérifier l'installation
docker exec jenkins kubectl version --client
```

### 3. Configurer kubectl pour Jenkins

```bash
# Créer le dossier .kube dans Jenkins
docker exec -u root jenkins mkdir -p /root/.kube

# Copier la configuration kubectl dans Jenkins (Windows)
docker cp %USERPROFILE%\.kube\config jenkins:/root/.kube/config

# Donner les permissions
docker exec -u root jenkins chmod 600 /root/.kube/config
```

### 4. Configurer le contexte Minikube

```bash
# Générer la configuration Minikube aplatie (Windows)
minikube kubectl -- config view --flatten > %USERPROFILE%\.kube\config

# Copier à nouveau dans Jenkins
docker cp %USERPROFILE%\.kube\config jenkins:/root/.kube/config
```

### 5. Vérifier la connexion

```bash
# Tester depuis Jenkins
docker exec jenkins kubectl get nodes
```

### 6. Déployer sur Kubernetes

```bash
# Appliquer les manifests Kubernetes
kubectl apply -f k8s/

# Vérifier les déploiements
kubectl get deployments
kubectl get services
kubectl get pods

# Accéder à l'application
minikube service gestion-produits-service --url
```

---

## CI/CD Pipeline

Le pipeline Jenkins automatise :
1. Checkout Code
2. Build Maven
3. Tests (Unitaires, Intégration, Selenium)
4. Package Application
5. Build Docker Image
6. Push to Docker Hub
7. Deploy to Kubernetes
8. Health Check

---

## Monitoring

**Prometheus + Grafana** pour surveiller :
* Métriques JVM (mémoire, threads, GC)
* Performances applicatives (requêtes HTTP, latence)
* Santé des services (health checks)
* Métriques métier (nombre de produits, opérations CRUD)

**Accès** :
* Prometheus : http://localhost:9090
* Grafana : http://localhost:3000

---

## Structure du projet
```
gestion-produits/
├── src/
│   ├── main/java/          # Code source
│   ├── main/resources/     # Configuration, templates
│   └── test/java/          # Tests
├── k8s/                    # Manifests Kubernetes
├── Dockerfile              # Image Docker
├── docker-compose.yml      # Orchestration locale
├── Jenkinsfile             # Pipeline CI/CD
├── .env.example            # Template variables
└── pom.xml                 # Dépendances Maven
```

---

## Notes importantes

* **Port par défaut** : 5000
* **H2** utilisé pour les tests
* **Chrome** et ChromeDriver gérés automatiquement par WebDriverManager
* **Jenkins** : Modifier `DOCKER_IMAGE` dans le Jenkinsfile avec votre username Docker Hub
* **Docker Compose** : Pour tests locaux uniquement

---

## Auteurs

Développé dans le cadre d'un projet académique FST

---

## Licence

MIT License
