# Gestion Produits 

## Description
Projet Spring Boot de gestion des produits avec interface web, base de données, tests automatisés et pipeline CI/CD.

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
* Docker Desktop (avec Kubernetes activé)
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
# Lancer Jenkins avec toutes les configurations nécessaires

docker run -d --name jenkins --restart unless-stopped -p 8080:8080 -p 50000:50000 -v jenkins_home:/var/jenkins_home -v /var/run/docker.sock:/var/run/docker.sock -v /c/Users/HP/.kube:/root/.kube -u root jenkins/jenkins:lts
```

### 2. Récupérer le mot de passe initial
```bash
docker exec jenkins cat /var/jenkins_home/secrets/initialAdminPassword
```

### 3. Installer les outils nécessaires dans Jenkins
```bash
# Accéder au conteneur Jenkins en tant que root
docker exec -it -u root jenkins bash

# Mettre à jour les packages
apt-get update

# Installer Google Chrome
wget -q -O - https://dl-ssl.google.com/linux/linux_signing_key.pub | gpg --dearmor -o /usr/share/keyrings/google-chrome.gpg && \
echo "deb [arch=amd64 signed-by=/usr/share/keyrings/google-chrome.gpg] http://dl.google.com/linux/chrome/deb/ stable main" > /etc/apt/sources.list.d/google-chrome.list && \
apt-get update && \
apt-get install -y google-chrome-stable

# Installer Docker CLI
apt-get install -y docker.io
usermod -aG docker jenkins

# Installer Git
apt-get install -y git

# Installer Maven
apt-get install -y maven

# Installer JDK 17
apt-get install -y openjdk-17-jdk

# Vérifier les installations
google-chrome --version
docker --version
git --version
mvn --version
java --version

echo "✅ Toutes les installations sont terminées !"

# Quitter le conteneur
exit

# Redémarrer Jenkins pour appliquer les changements
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

### 7. Lancer le pipeline

Une fois le pipeline configuré, cliquez simplement sur **"Build Now"** ! 

Le pipeline Jenkins s'occupera automatiquement de :
- ✅ Compiler le code
- ✅ Exécuter tous les tests
- ✅ Construire l'image Docker
- ✅ Pousser l'image sur Docker Hub
- ✅ Déployer sur Kubernetes
- ✅ Configurer Prometheus et Grafana

**Résultat** : Votre application sera accessible via :
- 🌐 **Application** : http://localhost/
- 📊 **Prometheus** : http://localhost/prometheus
- 📈 **Grafana** : http://localhost/grafana

---

## Configuration Kubernetes (Docker Desktop)

### 1. Activer Kubernetes dans Docker Desktop

1. Ouvrir **Docker Desktop**
2. Aller dans **Settings** → **Kubernetes**
3. Cocher **Enable Kubernetes**
4. Cliquer sur **Apply & Restart**

### 2. Installer l'Ingress Controller

⚠️ **Important** : Cette étape est requise **une seule fois** avant le premier déploiement.
```bash
# Installer l'Ingress Controller pour Docker Desktop
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.8.1/deploy/static/provider/cloud/deploy.yaml

# Vérifier l'installation
kubectl get pods -n ingress-nginx
```

Attendez que tous les pods soient en statut `Running` avant de lancer le pipeline Jenkins.

### 3. Vérifier le déploiement (après le pipeline)
```bash
# Vérifier que tout est déployé correctement
kubectl get deployments
kubectl get services
kubectl get pods
kubectl get ingress
```

---

## CI/CD Pipeline

Le pipeline Jenkins automatise **tout le processus** :

1.  **Checkout Code** - Récupère le code depuis GitHub
2.  **Build Maven** - Compile le projet
3.  **Tests** - Exécute les tests (Unitaires, Intégration, Selenium)
4.  **Package** - Crée le fichier JAR
5.  **Build Image** - Construit l'image Docker
6. ⬆ **Push Docker Hub** - Envoie l'image sur Docker Hub
7.  **Deploy Kubernetes** - Déploie sur Kubernetes (MySQL, App, Prometheus, Grafana, Ingress)
8.  **Health Check** - Vérifie que tout fonctionne


---

## Monitoring

**Prometheus + Grafana** pour surveiller :
* Métriques JVM (mémoire, threads, GC)
* Performances applicatives (requêtes HTTP, latence)
* Santé des services (health checks)
* Métriques métier (nombre de produits, opérations CRUD)

**Accès via Ingress** :
* Prometheus : http://localhost/prometheus
* Grafana : http://localhost/grafana

---

## Structure du projet
```
gestion-produits/
├── src/
│   ├── main/java/          # Code source
│   ├── main/resources/     # Configuration, templates
│   └── test/java/          # Tests
├── k8s/                    # Manifests Kubernetes
│   ├── deployment.yaml
│   ├── service.yaml
│   ├── ingress.yaml
│   ├── mysql-deployment.yaml
│   ├── prometheus-config.yaml
│   └── grafana.yaml
├── Dockerfile              # Image Docker
├── docker-compose.yml      # Orchestration locale
├── Jenkinsfile             # Pipeline CI/CD (automatisation complète)
├── .env.example            # Template variables
└── pom.xml                 # Dépendances Maven
```

---

## Notes importantes

* **Ingress Controller** : À installer une seule fois avant le premier déploiement
* **Port par défaut** : 5000
* **H2** utilisé pour les tests
* **Chrome** et ChromeDriver gérés automatiquement par WebDriverManager
* **Docker Desktop** : Kubernetes doit être activé
* **Ingress** : Tous les services sont accessibles via `localhost` avec différents chemins
* **Docker Compose** : Pour tests locaux uniquement


