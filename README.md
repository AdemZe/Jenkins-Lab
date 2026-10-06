# 🚀 Jenkins Lab — CI/CD & DevSecOps Pipeline



> Laboratoire pédagogique DevOps réalisé autour de **Jenkins**, avec une chaîne CI/CD complète intégrant **GitHub, Maven, SonarQube, OWASP Dependency-Check, Docker, Trivy, Snyk et DockerHub**.
>
> **Jenkins est installé directement sur Ubuntu.** Cette partie du laboratoire est volontairement réalisée **sans Kubernetes / Minikube**, afin de maîtriser Jenkins, ses credentials, ses tools et son fonctionnement de pipeline avant d'introduire une plateforme d'orchestration.

---

## 📌 Sommaire

- [1. Présentation du projet](#1--présentation-du-projet)
- [2. Objectifs](#objectifs)
- [3. Architecture globale](#3--architecture-globale)
- [4. Technologies et outils](#4--technologies-et-outils)
- [5. Application utilisée](#5--application-utilisée)
- [6. Installation de Jenkins](#6--installation-de-jenkins)
- [7. Validation de l'application avant CI/CD](#7--validation-de-lapplication-avant-cicd)
- [8. Configuration de Jenkins](#8--configuration-de-jenkins)
- [9. Installation et configuration de SonarQube](#9--installation-et-configuration-de-sonarqube)
- [10. Pipeline Jenkins](#10--pipeline-jenkins)
- [11. Gestion des versions Java](#11--gestion-des-versions-java)
- [12. Sécurité et scans](#12--sécurité-et-scans)
- [13. Publication DockerHub](#13--publication-dockerhub)
- [14. Résultats](#14--résultats)
- [15. Problèmes et points importants](#15--problèmes-et-points-importants)
- [16. Structure recommandée du dépôt](#16--structure-recommandée-du-dépôt)
- [17. Commandes utiles](#17--commandes-utiles)
- [18. Ce que j'ai appris](#18--ce-que-jai-appris)
- [19. Améliorations futures](#19--améliorations-futures)
- [20. Conclusion](#20--conclusion)

---

# 1. 📖 Présentation du projet



Ce laboratoire a pour objectif de construire une **pipeline CI/CD et DevSecOps de bout en bout** autour d'une application Java Spring Boot legacy nommée **Shopping Cart**.

L'idée principale est de transformer une opération manuelle de build en un processus automatisé dans lequel Jenkins :

1. récupère le code depuis GitHub ;
2. vérifie l'environnement Java ;
3. compile, teste et package l'application avec Maven ;
4. analyse la qualité du code avec SonarQube ;
5. attend le résultat du Quality Gate ;
6. analyse les dépendances avec OWASP Dependency-Check ;
7. publie le rapport de sécurité ;
8. construit une image Docker ;
9. scanne l'image avec Trivy ;
10. scanne également le conteneur avec Snyk ;
11. publie finalement l'image sur DockerHub.

Le dépôt utilisé par le pipeline est :

**https://github.com/AdemZe/Jenkins-Lab.git**

Le principe général du laboratoire est donc :

```text
SOURCE
  │
  ▼
BUILD & TEST
  │
  ▼
CODE QUALITY
  │
  ▼
DEPENDENCY SECURITY
  │
  ▼
CONTAINERIZATION
  │
  ▼
CONTAINER SECURITY
  │
  ▼
IMAGE REGISTRY
```

---

# 2. 🎯 Objectifs

Ce projet a été construit pour pratiquer les compétences suivantes :

- administration de Jenkins sur Ubuntu ;
- création et lecture d'une **Declarative Pipeline** ;
- intégration GitHub ;
- gestion sécurisée des **Credentials** Jenkins ;
- configuration des **Tools** Jenkins ;
- utilisation du **Maven Wrapper** ;
- gestion de plusieurs versions de Java ;
- intégration SonarQube ;
- utilisation d'un **Quality Gate** ;
- analyse des dépendances avec OWASP Dependency-Check ;
- création d'images Docker ;
- scan de vulnérabilités avec Trivy ;
- scan de conteneurs avec Snyk ;
- publication d'images sur DockerHub ;
- lecture et diagnostic du résultat d'une pipeline CI/CD.

L'objectif n'est pas seulement de faire fonctionner les commandes, mais de comprendre **pourquoi chaque étape existe et comment les outils s'enchaînent**.

---

# 3. 🏗️ Architecture globale

## 3.1 Vue d'ensemble

```text
                          ┌────────────────────┐
                          │       GitHub       │
                          │   Source Code      │
                          └─────────┬──────────┘
                                    │
                                    │ checkout
                                    ▼
                          ┌────────────────────┐
                          │      Jenkins       │
                          │   CI/CD Server     │
                          └─────────┬──────────┘
                                    │
                 ┌──────────────────┼──────────────────┐
                 │                  │                  │
                 ▼                  ▼                  ▼
          ┌────────────┐      ┌────────────┐      ┌────────────┐
          │ Java 8 +   │      │ SonarQube  │      │   OWASP    │
          │ Maven      │      │ Quality    │      │ Dependency │
          │ Build/Test │      │ Analysis   │      │   Check    │
          └─────┬──────┘      └─────┬──────┘      └─────┬──────┘
                │                   │                   │
                └───────────────────┼───────────────────┘
                                    ▼
                          ┌────────────────────┐
                          │    Docker Build    │
                          └─────────┬──────────┘
                                    │
                           ┌────────┴────────┐
                           ▼                 ▼
                     ┌──────────┐       ┌──────────┐
                     │  Trivy   │       │   Snyk   │
                     │  Image   │       │Container │
                     │   Scan   │       │   Scan   │
                     └────┬─────┘       └────┬─────┘
                          │                  │
                          └────────┬─────────┘
                                   ▼
                          ┌────────────────────┐
                          │     DockerHub       │
                          │    Image Registry   │
                          └────────────────────┘
```

## 3.2 Principe de fonctionnement

La pipeline sépare volontairement les responsabilités :

- **Jenkins** orchestre ;
- **Maven** construit et teste ;
- **SonarQube** contrôle la qualité du code ;
- **OWASP Dependency-Check** contrôle les dépendances ;
- **Docker** construit l'artefact de déploiement ;
- **Trivy** et **Snyk** analysent l'image ;
- **DockerHub** stocke l'image finale.

---

# 4. 🧰 Technologies et outils

| Technologie / outil | Rôle |
|---|---|
| **Jenkins** | Automatisation et orchestration CI/CD |
| **GitHub** | Gestion du code source |
| **Java 8** | Build du projet Spring Boot legacy |
| **Java 21** | Runtime utilisé pour SonarQube et OWASP dans cette pipeline |
| **Maven Wrapper** | Build reproductible du projet |
| **SonarQube** | Analyse de qualité du code |
| **PostgreSQL** | Base de données de SonarQube |
| **OWASP Dependency-Check** | Détection de vulnérabilités dans les dépendances |
| **Docker** | Construction de l'image applicative |
| **Trivy** | Scan de vulnérabilités de l'image Docker |
| **Snyk** | Analyse complémentaire de sécurité du conteneur |
| **DockerHub** | Registry pour publier l'image |

---

# 5. 🛒 Application utilisée

Le laboratoire utilise l'application **Shopping Cart**, une application Spring Boot legacy.

Avant de brancher l'application à Jenkins, elle a été validée localement :

- compilation Maven ;
- exécution des tests ;
- génération du JAR ;
- démarrage de l'application ;
- validation d'une API REST ;
- validation de l'interface Web.

Cette validation locale permet de distinguer un problème applicatif d'un problème Jenkins.

---

# 6. 🔧 Installation de Jenkins

## 6.1 Installation du service Jenkins

Jenkins est installé comme service système directement sur Ubuntu.

![Installation et état du service Jenkins](./screenshots/01_jenkins_installation_and_service.png)

### Ce que montre la capture

La capture montre notamment :

- l'installation du paquet Jenkins ;
- la création du service `jenkins.service` ;
- le service en état **active (running)** ;
- Jenkins à l'écoute sur le port **8080**.

### Commandes utilisées

```bash
sudo apt install -y jenkins
sudo systemctl status jenkins
sudo systemctl enable jenkins
sudo ss -ltnp | grep 8080
```

### Pourquoi cette étape ?

Avant toute configuration de pipeline, Jenkins doit être opérationnel en tant que service système.

---

## 6.2 Récupération du mot de passe administrateur initial

![Mot de passe administrateur initial](./screenshots/02_initial_admin_password.png)

Le mot de passe initial est récupéré depuis le fichier généré par Jenkins :

```bash
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```

Cette valeur sert uniquement à finaliser la première installation.

> ⚠️ **Sécurité :** cette valeur est sensible. La capture sert ici de documentation du laboratoire. Avant de rendre le dépôt public, il faut vérifier qu'aucun secret encore valide n'apparaît dans les captures.

---

## 6.3 Écran Getting Started

![Jenkins Getting Started](./screenshots/03_jenkins_getting_started.png)

L'écran **Getting Started** correspond à la phase initiale de configuration de Jenkins et de ses plugins de base.

Cette étape introduit un concept essentiel : **Jenkins est extensible grâce à des plugins**.

Dans ce laboratoire, les plugins permettent notamment :

- le fonctionnement des pipelines ;
- l'intégration Git ;
- l'utilisation des credentials ;
- l'intégration Maven ;
- les fonctionnalités Docker ;
- l'intégration SonarQube ;
- la publication du rapport OWASP.

---

## 6.4 Création du premier administrateur

![Création du premier administrateur Jenkins](./screenshots/04_create_first_admin_user.png)

L'écran **Create First Admin User** permet de créer le compte administrateur utilisé pour gérer Jenkins.

Les informations demandées sont typiquement :

```text
Username
Password
Confirm password
Full name
Email
```

Ce compte devient le point d'entrée administratif de l'instance Jenkins.

---

## 6.5 Configuration de l'instance Jenkins

![Configuration de l'instance Jenkins](./screenshots/05_jenkins_instance_configuration.png)

L'écran **Instance Configuration** permet de définir l'URL racine de Jenkins.

Dans le laboratoire, Jenkins fonctionne sur le port **8080**.

### Pourquoi l'URL est importante ?

Jenkins utilise son URL racine pour générer correctement les liens et pour certaines intégrations externes, notamment les mécanismes de webhook.

---

# 7. ✅ Validation de l'application avant CI/CD

Avant d'automatiser le build avec Jenkins, l'application a été vérifiée directement sur la machine de travail.

Cette étape est importante : **un pipeline fiable commence par un projet qui peut déjà être construit et exécuté localement**.

---

## 7.1 Git et Maven Wrapper

![Git et Maven Wrapper](./screenshots/06_git_and_maven_wrapper_setup.png)

La capture montre notamment des commandes liées à :

```bash
git push -u origin main
git branch
chmod +x scripts/mvnw
./scripts/mvnw --version
./scripts/mvnw clean test
```

Le projet utilise le wrapper Maven :

```text
./scripts/mvnw
```

La capture permet également de voir le travail de préparation du dépôt Git ainsi que la permission d'exécution du Maven Wrapper.

### Pourquoi utiliser un Maven Wrapper ?

Le wrapper permet au projet d'utiliser la version Maven attendue par le projet au lieu de dépendre entièrement de l'installation Maven globale de la machine.

---

## 7.2 Tests Maven réussis

![Maven tests réussis](./screenshots/07_maven_test_success.png)

Le résultat affiché est :

```text
Results :

Tests run: 1, Failures: 0, Errors: 0, Skipped: 0

BUILD SUCCESS
```

### Ce que ce résultat valide

- les tests sont exécutables ;
- aucun test n'échoue dans l'exécution montrée ;
- Maven termine correctement ;
- le projet est suffisamment stable pour passer à l'automatisation Jenkins.

---

## 7.3 Exécution locale de l'application Spring Boot

![Exécution locale du JAR Spring Boot](./screenshots/08_local_spring_boot_run.png)

Le JAR généré est lancé avec Java 8 :

```text
shopping-cart-0.0.1-SNAPSHOT.jar
```

La capture montre le démarrage de l'application Spring Boot et de son serveur embarqué, avec l'application disponible sur le port **8070**.

Cette étape permet de vérifier que le résultat du build est réellement exécutable.

---

## 7.4 Validation de l'API REST avec HAL Browser

![Validation REST avec HAL Browser](./screenshots/09_hal_browser_rest_validation.png)

La capture montre une réponse HTTP réussie avec :

```text
200 success
```

Plusieurs ressources sont exposées, notamment :

```text
users
roles
products
profile
```

Cette validation confirme que l'application répond correctement après son démarrage.

---

## 7.5 Validation de l'interface Shopping Cart

![Interface Shopping Cart](./screenshots/10_shopping_cart_ui.png)

La capture montre l'interface utilisateur de l'application avec différents produits et leurs informations de prix et de stock.

Cette dernière vérification permet de confirmer le fonctionnement de l'application au niveau fonctionnel avant de lancer le cycle CI/CD automatisé.

---

# 8. ⚙️ Configuration de Jenkins

## 8.1 Installation des plugins nécessaires

![Plugins Jenkins](./screenshots/11_jenkins_plugins_download.png)

La capture montre le téléchargement / l'installation des composants nécessaires à la pipeline.

Les fonctionnalités nécessaires au laboratoire couvrent notamment :

- Pipeline ;
- Git ;
- Credentials Binding ;
- Maven ;
- SonarQube Scanner ;
- Quality Gates ;
- Docker Pipeline ;
- OWASP Dependency-Check.

### Relation entre plugins et Jenkinsfile

Le Jenkinsfile utilise par exemple :

```groovy
withSonarQubeEnv('SonarQube')
waitForQualityGate(...)
dependencyCheck(...)
dependencyCheckPublisher(...)
withCredentials(...)
withDockerRegistry(...)
checkout(...)
```

Chaque intégration repose donc sur une capacité fournie par Jenkins ou par un plugin.

---

## 8.2 Jenkins Credentials

![Credentials Jenkins](./screenshots/12_jenkins_credentials.png)

Les credentials permettent de stocker des informations sensibles dans Jenkins sans les écrire directement dans le code de la pipeline.

Dans ce laboratoire, les credentials servent notamment à :

| Credential logique | Utilisation |
|---|---|
| **GitHub Credential** | Authentification pour récupérer le dépôt |
| **DockerHub Credential** | Authentification pour le push de l'image |
| **NVD API Key** | Accès à la base NVD pour OWASP Dependency-Check |
| **Snyk Token** | Authentification Snyk |

### Principe

Le Jenkinsfile référence un **credential ID** plutôt que d'écrire la valeur secrète :

```groovy
withCredentials([
    string(
        credentialsId: 'nvd-api-key',
        variable: 'NVD_API_KEY'
    )
])
```

Le secret reste géré par Jenkins et est fourni au stage au moment de l'exécution.

### Schéma simple

```text
Jenkins Credentials Store
          │
          ▼
     credentialsId
          │
          ▼
  Injection contrôlée
          │
          ▼
       Stage
```

---

# 9. 🔎 Installation et configuration de SonarQube

## 9.1 Architecture SonarQube + PostgreSQL

![Docker Compose SonarQube](./screenshots/13_sonarqube_docker_compose.png)

SonarQube est lancé via Docker Compose avec deux services principaux :

```text
sonarqube
sonarqube-db
```

La base de données repose sur PostgreSQL et SonarQube est exposé sur le port **9000**.

La persistance est assurée à l'aide de volumes afin d'éviter de perdre les données lors du redémarrage des conteneurs.

### Architecture

```text
┌───────────────┐
│    Jenkins    │
└───────┬───────┘
        │ analyse
        ▼
┌────────────────┐
│   SonarQube    │ :9000
└───────┬────────┘
        │
        ▼
┌────────────────┐
│   PostgreSQL   │ :5432
└────────────────┘
```

---

## 9.2 Démarrage des conteneurs SonarQube

![Conteneurs et logs SonarQube](./screenshots/14_sonarqube_containers_and_logs.png)

Les commandes utilisées dans cette phase permettent de démarrer et contrôler SonarQube :

```bash
docker compose up -d
docker compose ps
docker compose logs -f sonarqube
```

### Rôle des commandes

`docker compose up -d` démarre les services en arrière-plan.

`docker compose ps` permet de vérifier l'état des conteneurs et les ports exposés.

`docker compose logs -f sonarqube` permet d'observer le démarrage de SonarQube et de diagnostiquer un éventuel problème.

---

# 10. 🔄 Pipeline Jenkins

Le pipeline est défini dans le fichier [`Jenkinsfile`](./Jenkinsfile).

## 10.1 Vue d'ensemble

```text
┌────────────────────┐
│ 1. Clone Repository│
└──────────┬─────────┘
           ▼
┌────────────────────┐
│ 2. Check Java 8    │
└──────────┬─────────┘
           ▼
┌────────────────────┐
│ 3. Maven Package   │
└──────────┬─────────┘
           ▼
┌────────────────────┐
│ 4. SonarQube       │
│    Analysis        │
└──────────┬─────────┘
           ▼
┌────────────────────┐
│ 5. Quality Gate    │
└──────────┬─────────┘
           ▼
┌────────────────────┐
│ 6. OWASP Dependency│
│    Check            │
└──────────┬─────────┘
           ▼
┌────────────────────┐
│ 7. Publish OWASP   │
│    Report           │
└──────────┬─────────┘
           ▼
┌────────────────────┐
│ 8. Docker Build    │
└──────────┬─────────┘
           ▼
┌────────────────────┐
│ 9. Trivy Scan      │
└──────────┬─────────┘
           ▼
┌────────────────────┐
│10. Snyk Scan       │
└──────────┬─────────┘
           ▼
┌────────────────────┐
│11. Docker Push     │
└────────────────────┘
```

## 10.2 Déclaration des outils

Le pipeline déclare :

```groovy
tools {
    jdk 'java8'
    maven 'maven3'
}
```

Ces noms correspondent aux outils configurés dans **Manage Jenkins → Tools**.

Le build utilise ensuite principalement le Maven Wrapper du projet :

```bash
./scripts/mvnw
```

---

## 10.3 Stage 1 — Clone Repository

Le pipeline récupère la branche `main` du dépôt :

```groovy
branches: [[
    name: '*/main'
]]
```

Le checkout utilise un credential Jenkins associé au dépôt Git.

Le pipeline affiche ensuite :

```bash
git branch --show-current
git log -1 --oneline
```

Cela permet d'identifier le commit réellement construit.

---

## 10.4 Stage 2 — Check Java 8

Le pipeline vérifie l'environnement Java :

```bash
echo "JAVA_HOME=$JAVA_HOME"
java -version
./scripts/mvnw -version
```

Cette vérification est importante car le projet est un **Spring Boot legacy** qui doit être construit avec Java 8 dans ce laboratoire.

---

## 10.5 Stage 3 — Maven Package

Le build principal est lancé par :

```bash
./scripts/mvnw clean package
```

La pipeline vérifie ensuite le contenu du répertoire :

```bash
ls -lh target/
ls -lh target/*.jar
```

### Résultat attendu

```text
code source
   ↓
compilation
   ↓
tests
   ↓
package
   ↓
JAR
```

---

## 10.6 Stage 4 — SonarQube Analysis

Pour cette étape, la pipeline sélectionne Java 21 :

```groovy
def java21 = tool(
    name: 'java21',
    type: 'jdk'
)
```

Puis le `JAVA_HOME` est modifié pour ce stage :

```groovy
withEnv([
    "JAVA_HOME=${java21}",
    "PATH+JAVA=${java21}/bin"
])
```

L'intégration Jenkins → SonarQube utilise :

```groovy
withSonarQubeEnv('SonarQube')
```

L'analyse est ensuite lancée avec le plugin Maven SonarQube défini dans le Jenkinsfile.

---

## 10.7 Stage 5 — SonarQube Quality Gate

Après l'analyse, Jenkins attend le résultat du Quality Gate :

```groovy
waitForQualityGate(
    abortPipeline: true
)
```

La pipeline attend au maximum :

```groovy
timeout(
    time: 10,
    unit: 'MINUTES'
)
```

### Principe

```text
SonarQube Analysis
       ↓
Quality Gate
       ↓
PASS ─────────► continuer
       │
       └──── FAIL ─────► pipeline interrompue
```

Dans ce laboratoire, l'étape Quality Gate est donc un contrôle réel du flux.

---

## 10.8 Stage 6 — OWASP Dependency-Check

L'analyse OWASP utilise Java 21 et un credential NVD :

```groovy
withCredentials([
    string(
        credentialsId: 'nvd-api-key',
        variable: 'NVD_API_KEY'
    )
])
```

Le répertoire de sortie est créé avant le scan :

```bash
mkdir -p dependency-check-report
```

Les formats demandés sont :

```text
HTML
XML
JSON
```

avec :

```text
--scan .
--out dependency-check-report
```

### Pourquoi trois formats ?

| Format | Utilité |
|---|---|
| HTML | Consultation humaine |
| XML | Publication Jenkins |
| JSON | Exploitation automatisée |

---

## 10.9 Stage 7 — Publish OWASP Report

La pipeline publie le rapport XML avec :

```groovy
dependencyCheckPublisher(
    pattern: 'dependency-check-report/dependency-check-report.xml'
)
```

Cela permet de retrouver le résultat du scan directement depuis l'interface Jenkins du job.

---

## 10.10 Stage 8 — Docker Build

Le tag Docker est calculé à partir du numéro du build Jenkins :

```groovy
def VERSION = "v${env.BUILD_NUMBER}"
```

L'image principale est construite avec :

```text
adeem07/shopping-cart
```

La pipeline crée deux tags :

```text
adeem07/shopping-cart:vX
adeem07/shopping-cart:latest
```

Le build utilise :

```bash
docker build \
    -f docker/Dockerfile \
    -t ${IMAGE}:${VERSION} \
    -t ${IMAGE}:latest \
    .
```

---

## 10.11 Stage 9 — Trivy Image Scan

Trivy analyse l'image Docker générée précédemment.

La commande définie dans le Jenkinsfile cible :

```text
HIGH
CRITICAL
```

et utilise notamment :

```bash
trivy image \
    --timeout 15m \
    --scanners vuln \
    --severity HIGH,CRITICAL \
    --exit-code 1 \
    ${IMAGE}
```

### Pourquoi `catchError` ?

Le scan est encapsulé dans :

```groovy
catchError(
    buildResult: 'UNSTABLE',
    stageResult: 'UNSTABLE'
)
```

Dans ce laboratoire **éducatif**, la découverte de vulnérabilités HIGH/CRITICAL ne bloque volontairement pas la suite de la pipeline.

---

## 10.12 Stage 10 — Snyk Container Scan

Le token Snyk est injecté de façon sécurisée :

```groovy
withCredentials([
    string(
        credentialsId: 'snyk-token',
        variable: 'SNYK_TOKEN'
    )
])
```

Le pipeline authentifie ensuite Snyk puis lance :

```bash
snyk auth "$SNYK_TOKEN"
snyk container test "$IMAGE_TO_SCAN"
```

Comme pour Trivy, `catchError` transforme le résultat en **UNSTABLE** et laisse la pipeline continuer dans le contexte pédagogique du projet.

---

## 10.13 Stage 11 — Docker Push

La publication DockerHub utilise le credential configuré dans Jenkins :

```groovy
withDockerRegistry(
    credentialsId: '...'
) {
    ...
}
```

Puis les deux tags sont envoyés :

```bash
docker push ${IMAGE}:${VERSION}
docker push ${IMAGE}:latest
```

Le résultat final est donc une image versionnée et une référence `latest` dans DockerHub.

---

# 11. ☕ Gestion des versions Java

La gestion de Java est un point central du laboratoire.

## Java 8

Utilisé pour le projet applicatif :

```text
Build
Test
Package
Exécution de l'application legacy
```

## Java 21

Utilisé spécifiquement pour :

```text
SonarQube Analysis
OWASP Dependency-Check
```

## Vue synthétique

```text
                  Jenkins Pipeline
                         │
             ┌───────────┴───────────┐
             │                       │
          Java 8                  Java 21
             │                       │
       Build / Test             Analyse / Security
       Package                  SonarQube + OWASP
```

Cette séparation évite d'imposer la même version de Java à l'ensemble des outils lorsque le projet legacy et les outils modernes ont des besoins différents.

---

# 12. 🔐 Sécurité et scans

## 12.1 SonarQube

SonarQube intervient au niveau du **code source** et de sa qualité.

Le pipeline enchaîne :

```text
Maven Package
      ↓
SonarQube Analysis
      ↓
Quality Gate
```

---

## 12.2 OWASP Dependency-Check

OWASP se concentre sur les **dépendances du projet**.

```text
Projet Java
   ↓
Dépendances Maven
   ↓
OWASP Dependency-Check
   ↓
Rapports HTML / XML / JSON
```

---

## 12.3 Trivy

Trivy intervient après la création de l'image :

```text
Docker Build
     ↓
Docker Image
     ↓
Trivy
     ↓
HIGH / CRITICAL
```

---

## 12.4 Snyk

Snyk fournit un second contrôle de sécurité du conteneur :

```text
Docker Image
     ↓
Snyk Container Test
     ↓
Vulnerabilities
```

L'intérêt pédagogique est d'utiliser **plus d'un outil de sécurité** et de comparer leurs résultats.

---

## 12.5 Politique actuelle du laboratoire

Trivy et Snyk utilisent volontairement `catchError` afin qu'une découverte de vulnérabilités ne bloque pas le pipeline complet.

```text
Vulnerability found
       ↓
Security Scan
       ↓
UNSTABLE
       ↓
Pipeline continues
```

### Pour une future pipeline de production

Une stratégie plus stricte pourrait, par exemple, bloquer une release lorsqu'un seuil de vulnérabilité défini par la politique de sécurité est dépassé.

---

# 13. 🐳 Publication DockerHub

L'image Docker utilisée dans le laboratoire est :

```text
adeem07/shopping-cart
```

La pipeline construit :

```text
adeem07/shopping-cart:v<BUILD_NUMBER>
adeem07/shopping-cart:latest
```

Puis la pousse vers DockerHub.

Cette logique permet de disposer à la fois :

- d'un tag de version associé au build Jenkins ;
- d'un tag `latest` pour référencer la version courante.

---

# 14. 📊 Résultats

## 14.1 Résultat Jenkins

![Résultat de la pipeline Jenkins](./screenshots/15_jenkins_pipeline_success.png)

La capture montre le job Jenkins avec un résultat de pipeline réussi ainsi que l'accès au rapport **Dependency-Check**.

Le flux exécuté correspond à :

```text
GitHub
   ↓
Checkout
   ↓
Java 8
   ↓
Maven Package
   ↓
SonarQube Analysis
   ↓
Quality Gate
   ↓
OWASP Dependency-Check
   ↓
Publish OWASP Report
   ↓
Docker Build
   ↓
Trivy
   ↓
Snyk
   ↓
DockerHub Push
```

La capture des builds permet également de constater qu'au moins un build précédent et un build réussi sont présents dans l'historique montré.

---

## 14.2 Résultat SonarQube

![Dashboard SonarQube](./screenshots/16_sonarqube_project_dashboard.png)

Le projet `shopping-cart` apparaît dans SonarQube.

La capture montre notamment :

```text
Quality Gate: Passed
Lines of Code: 840
Language: Java, XML
```

Le dashboard présente également les catégories principales de l'analyse :

- Security ;
- Reliability ;
- Maintainability ;
- Coverage ;
- Duplications.

### Point important

**Quality Gate: Passed** signifie que les règles de Quality Gate configurées au moment de l'analyse sont validées. Cela ne signifie pas nécessairement que toutes les catégories affichées sont sans aucun problème.

---

## 14.3 Résultat DockerHub

![Image DockerHub](./screenshots/17_dockerhub_image.png)

La capture montre le repository :

```text
adeem07/shopping-cart
```

ainsi que plusieurs tags, notamment :

```text
latest
v26
v23
```

### Remarque sur les versions

Le Jenkinsfile actuel calcule automatiquement le tag à partir du numéro de build :

```groovy
def VERSION = "v${env.BUILD_NUMBER}"
```

Ainsi, un build Jenkins `#16` produirait logiquement `v16` avec cette version précise du Jenkinsfile.

La capture DockerHub affiche cependant `v26`, `v23` et `latest`. Cette différence indique que la capture DockerHub provient d'une exécution ou d'une version de pipeline différente de celle du Jenkinsfile documenté ici.

Cette remarque est conservée volontairement afin que la documentation reste fidèle aux preuves visibles dans les captures.

---

# 15. 🛠️ Problèmes et points importants

## 15.1 Projet Spring Boot legacy et Java

L'application est ancienne et la compilation est réalisée avec **Java 8** dans ce laboratoire.

Les outils modernes d'analyse utilisent quant à eux **Java 21** dans les stages dédiés.

Le point essentiel à retenir est :

```text
Legacy Application
      ↓
Java 8

Modern Tooling
      ↓
Java 21
```

---

## 15.2 Gestion des vulnérabilités

Des vulnérabilités peuvent être remontées par Trivy et Snyk, notamment dans l'image de base ou les dépendances du projet.

Dans ce lab, elles sont volontairement traitées comme un résultat pédagogique afin de pouvoir observer les rapports sans interrompre systématiquement la chaîne.

---

## 15.3 En cas d'échec de la pipeline

Le premier réflexe doit être de consulter **Console Output** et d'identifier le premier stage réellement en erreur.

Les points de contrôle principaux sont :

```text
GitHub credentials
Git clone
Java 8
Maven / Maven Wrapper
Compilation
Java 21
SonarQube
SonarQube credentials / configuration
Quality Gate / webhook
OWASP / NVD
Docker
Docker build
Trivy
Snyk / token
DockerHub / credentials
Docker push
```

### Méthode de diagnostic

```text
Pipeline FAILED
      ↓
Identifier le premier stage rouge
      ↓
Lire Console Output
      ↓
Vérifier outil / credential / service
      ↓
Tester localement si possible
      ↓
Relancer la pipeline
```

---

# 16. 📁 Structure recommandée du dépôt

La structure recommandée pour publier le projet sur GitHub est :

```text
Jenkins-Lab/
│
├── README.md
├── Jenkinsfile
├── docker/
│   └── Dockerfile
├── scripts/
│   └── mvnw
├── screenshots/
│   ├── 01_jenkins_installation_and_service.png
│   ├── 02_initial_admin_password.png
│   ├── 03_jenkins_getting_started.png
│   ├── 04_create_first_admin_user.png
│   ├── 05_jenkins_instance_configuration.png
│   ├── 06_git_and_maven_wrapper_setup.png
│   ├── 07_maven_test_success.png
│   ├── 08_local_spring_boot_run.png
│   ├── 09_hal_browser_rest_validation.png
│   ├── 10_shopping_cart_ui.png
│   ├── 11_jenkins_plugins_download.png
│   ├── 12_jenkins_credentials.png
│   ├── 13_sonarqube_docker_compose.png
│   ├── 14_sonarqube_containers_and_logs.png
│   ├── 15_jenkins_pipeline_success.png
│   ├── 16_sonarqube_project_dashboard.png
│   └── 17_dockerhub_image.png
└── .gitignore
```

### Fichiers à ne pas versionner

Les artefacts générés par le build peuvent être exclus du dépôt, par exemple :

```gitignore
target/
dependency-check-report/
*.log
.idea/
.vscode/
```

---

# 17. 💻 Commandes utiles

## Jenkins

```bash
sudo systemctl status jenkins
sudo systemctl restart jenkins
sudo journalctl -u jenkins -f
```

## Docker

```bash
docker --version
docker ps
```

## SonarQube

```bash
cd ~/Desktop/sonarqube
docker compose ps
docker compose logs -f sonarqube
```

## Maven Wrapper

```bash
chmod +x scripts/mvnw
./scripts/mvnw -version
./scripts/mvnw clean test
./scripts/mvnw clean package
```

## Exécution locale du JAR

```bash
/usr/lib/jvm/java-8-openjdk-amd64/bin/java \
  -jar target/shopping-cart-0.0.1-SNAPSHOT.jar
```

---

# 18. 🎓 Ce que j'ai appris

Ce laboratoire m'a permis de mettre en pratique une chaîne DevOps complète autour de Jenkins :

### Jenkins

- installation comme service Ubuntu ;
- configuration initiale ;
- plugins ;
- tools ;
- credentials ;
- pipelines déclaratives ;
- lecture des logs et des résultats de build.

### CI/CD

- récupération du code depuis GitHub ;
- automatisation Maven ;
- génération du JAR ;
- orchestration de plusieurs stages ;
- utilisation du numéro de build Jenkins pour versionner l'image.

### DevSecOps

- analyse de qualité avec SonarQube ;
- Quality Gate ;
- analyse des dépendances avec OWASP ;
- scan d'image avec Trivy ;
- scan complémentaire avec Snyk.

### Containerisation

- création d'une image Docker ;
- tags `v<BUILD_NUMBER>` et `latest` ;
- push vers DockerHub.

Le principal apprentissage est la compréhension du **flux complet**, de la récupération du code jusqu'à la publication de l'image.

---

# 19. 🚀 Améliorations futures

Ce laboratoire constitue une base pour aller plus loin.

Les améliorations naturelles seraient :

1. **Automatiser le déclenchement** de Jenkins à chaque changement GitHub avec un webhook.
2. **Publier automatiquement les rapports** de sécurité dans les artefacts du job.
3. **Renforcer la politique de sécurité**, par exemple en bloquant la release pour certains niveaux de vulnérabilité.
4. **Ajouter des tests plus complets** et une étape de couverture.
5. **Ajouter une phase de déploiement** après le push Docker.
6. **Introduire Kubernetes / Minikube ou un environnement Cloud** dans un laboratoire séparé.
7. **Ajouter de la supervision** afin de compléter la chaîne par une dimension observabilité.

---

# 20. 🏁 Conclusion

Ce Jenkins Lab montre une chaîne **CI/CD + DevSecOps complète** construite autour d'un projet Java legacy :

```text
GitHub
  ↓
Jenkins
  ↓
Maven / Java 8
  ↓
SonarQube / Quality Gate
  ↓
OWASP Dependency-Check
  ↓
Docker Build
  ↓
Trivy
  ↓
Snyk
  ↓
DockerHub
```

L'intérêt du projet est d'avoir travaillé non seulement sur l'exécution d'une pipeline, mais aussi sur :

- la configuration d'un serveur Jenkins ;
- la gestion des outils ;
- la gestion des secrets ;
- la compatibilité des versions Java ;
- l'intégration de plusieurs outils DevSecOps ;
- l'interprétation des résultats ;
- le diagnostic des erreurs.

Cette base peut ensuite être étendue vers des workflows plus proches d'un environnement de production avec **webhooks, déploiement automatisé, Kubernetes, Cloud, observabilité et politiques de sécurité plus strictes**.

---

## 👤 Auteur

**Adem Daghrour**  
Projet personnel / laboratoire pédagogique **DevOps & Cloud**

---

