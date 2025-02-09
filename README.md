# DevOps_CPE

### 1. Network + Volume

```bash
docker network create app-network
docker volume create pgdata
```

### 2. Connecteur avec information de connexion

```bash
docker run -d --name postgres-container \
    --network app-network \
    -v pgdata:/var/lib/postgresql/data \
    -e POSTGRES_DB=mydb \
    -e POSTGRES_USER=myuser \
    -e POSTGRES_PASSWORD=mypassword
```

### 3. Test de connexion à la bdd

```bash
docker exec -it postgres-container psql -U usr -d db
```

# Question TP

### 1-1 :
On utilise le flag -e pour définir les variables d’environnement lors de l’exécution du conteneur
Cela a plusieurs avantages :

- Sécurité : Éviter de stocker des informations sensibles (ex. mots de passe) dans Dockerfile qui pourraient être exposés dans un repo.
- Flexibilité : Permet de modifier les valeurs sans reconstruire l’image

### 1-2 :
Un volume Docker permet de garder les données de PostgreSQL, même si le conteneur est arrêté ou supprimé. Sans ça, toutes les données stockées dans la base de données seraient perdues à la suppression du conteneur.

### 1-3 :
On commente le code

### 1-4 :
On utilise multistage build car à l'aide de ce procédé l'image n'est générée qu'à partir du dernier from, séparant l'environnement de build et l'environnement de run. Cela permet de créer des images plus légères (ici sans supporter le jdk)

### 1-5 :
Un reverse proxy est essentiel pour les raisons suivantes :
Abstraction des services backend -> cache l'URL réelle et le port du backend ce qui rend la communication plus sécurisée et flexible

### 1-6 :
Docker Compose est un outil essentiel pour orchestrer des applications multi-conteneurs car :

- Simplification des déploiements :
    Permet de définir et de gérer plusieurs conteneurs dans un seul fichier docker-compose.yml.
    Automatisation du démarrage, de l’arrêt et de la configuration des services.

- Gestion centralisée :
    Configure les réseaux, volumes et dépendances entre les services en un seul endroit.
    Évite les commandes Docker complexes et longues.

- Portabilité :
    Partagez le fichier docker-compose.yml pour garantir un environnement identique sur différentes machines ou équipes.

- Interopérabilité :
    Intégration facile avec des outils CI/CD pour automatiser les pipelines de déploiement.

  ### 1-7 :
  Version commenté du docker compose
  
```bash
services:
  backtp1:
    build: "backend/simple-api-student"  # Construction du conteneur à partir du dossier backend/simple-api-student
    environment:   # Utilisation d'une variable d'environnement
      - URL=${URL}
      - POSTGRES_USER=${POSTGRES_USER} 
      - POSTGRES_PASSWORD=${POSTGRES_PASSWORD}  
    networks:
      - my-network  # Connexion au réseau défini ci-dessous
    depends_on:
      - tp1db  # Démarre après la base de données tp1db pour éviter des erreurs de connexion

  tp1db:
    build: "db"  # Construction du conteneur de la base de données à partir du dossier "db"
    environment:  # Utilisation d'une variable d'environnement
      - POSTGRES_DB=${POSTGRES_DB} 
      - POSTGRES_USER=${POSTGRES_USER} 
      - POSTGRES_PASSWORD=${POSTGRES_PASSWORD}  
    networks:
      - my-network  # Connexion au réseau pour la communication avec les autres services
    volumes:
      - /pgdata:/var/lib/postgresql/data  # Persistance des données de la base de données 

  httptp1:
    build: "http"  # Construction du conteneur HTTP depuis le dossier "http"
    ports:
      - "80:80"  # Redirection du port 80 de l’hôte vers le conteneur*
    networks:
      - my-network  # Connexion au réseau pour accéder au backend
    depends_on:
      - backtp1  # Démarre après le backend pour s'assurer qu'il est disponible

  frontend:
    build: ./develops-front-main  # Construction du conteneur du front-end à partir du dossier "develops-front-main"
    container_name: frontend  # Nom explicite du conteneur
    networks:
      - my-network  # Connexion au même réseau pour la communication avec le backend
    depends_on:
      - backtp1  # Démarre après le backend pour éviter des erreurs de chargement de l'interface utilisateur

networks:
    my-network:  # Définition d'un réseau unique pour permettre la communication entre les services
```

# TP2

### 2-1 :
Testcontainers est une bibliothèque Java qui facilite l'utilisation de conteneurs Docker pour les tests, les avantages sont :
- Tests d'intégration réalistes : Permet de tester des interactions avec des bases de données ou d'autres services externes dans un environnement Docker.
- Évite les dépendances locales : Pas besoin d'installer PostgreSQL ou d'autres services localement.
- Facilité d'utilisation : Configure et démarre automatiquement les conteneurs nécessaires pendant les tests.

### 2-3 For what purpose do we need to push docker images?

Avantages de pousser les images Docker sur Docker Hub :
- Accessibilité globale : Les images sont disponibles pour toute l'équipe.
- Standardisation : Partage d'environnements cohérents grâce à des conteneurs Docker reproductibles.
- Versionnement : Les images Docker peuvent être taguées (latest, v1.0, etc...) pour une gestion simplifiée des versions.

  ## comment version test backend :
```bash
name: Test Backend
# The name of the workflow. This will appear in the GitHub Actions interface.

on:
  push:
    branches:
      - main
      # This workflow is triggered whenever code is pushed to the 'main' branch.
  pull_request:
    # This workflow is also triggered on pull requests to the repository.

jobs:
  test-backend:
    runs-on: ubuntu-22.04
    # The job will run on an Ubuntu 22.04 environment.

    steps:
      - name: Checkout code
        uses: actions/checkout@v4
        # Step 1: Checkout the code from the repository to the runner's workspace.

      - name: Set up JDK 21
        uses: actions/setup-java@v4
        with:
          java-version: "21"
          distribution: "corretto"
        # Step 2: Set up Java Development Kit (JDK) version 21 using the Corretto distribution.

      - name: Build and test with Maven
        run: mvn clean verify
        working-directory: backend/simple-api-student
        # Step 3: Use Maven to clean, build, and test the backend project located in 'backend/simple-api-student'.

  build:
    name: Build and analyze
    # This is another job in the workflow, named 'Build and analyze'.

    runs-on: ubuntu-latest
    # This job runs on the latest Ubuntu environment.

    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0 # Shallow clones should be disabled for better relevancy of analysis.
        # Step 1: Checkout the repository with all history to ensure accurate analysis.

      - name: Set up JDK 21
        uses: actions/setup-java@v4
        with:
          java-version: 21
          distribution: "corretto" # Alternative Java distributions are available (e.g., temurin, zulu, etc.).
        # Step 2: Set up JDK 21 for this job using the Corretto distribution.

      - name: Cache SonarQube packages
        uses: actions/cache@v4
        with:
          path: ~/.sonar/cache
          key: ${{ runner.os }}-sonar
          restore-keys: ${{ runner.os }}-sonar
        # Step 3: Cache the SonarQube packages to speed up subsequent SonarCloud analysis runs.

      - name: Cache Maven packages
        uses: actions/cache@v4
        with:
          path: ~/.m2
          key: ${{ runner.os }}-m2-${{ hashFiles('**/pom.xml') }}
          restore-keys: ${{ runner.os }}-m2
        # Step 4: Cache Maven dependencies to avoid downloading them every time, which reduces build time.

      - name: Build and analyze
        env:
          SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}
        run: mvn -B verify org.sonarsource.scanner.maven:sonar-maven-plugin:sonar -Dsonar.projectKey=Karatann_DevOps_CPE -f backend/simple-api-student/pom.xml
```
## comment version push image

```bash
name: Build and Push Docker Images
# The name of the workflow. This will appear in the GitHub Actions interface.

on:
  workflow_run:
    workflows:
      - Test Backend
      # This workflow is triggered after the 'Test Backend' workflow is completed.
    types:
      - completed
      # It will only trigger when the 'Test Backend' workflow has fully completed.

jobs:
  build-and-push-docker-image:
    if: ${{ github.event.workflow_run.conclusion == 'success' && github.ref == 'refs/heads/main' }}
    # This job runs only if:
    # 1. The 'Test Backend' workflow concluded successfully.
    # 2. The current branch is 'main'.

    runs-on: ubuntu-22.04
    # The job runs on an Ubuntu 22.04 environment.

    steps:
      - name: Checkout code
        uses: actions/checkout@v4
        # Step 1: Clone the repository to the runner's workspace.

      - name: Login to DockerHub
        uses: docker/login-action@v2
        with:
          username: ${{ secrets.DOCKERHUB_USERNAME }}
          password: ${{ secrets.DOCKERHUB_TOKEN }}
        # Step 2: Log in to DockerHub using the credentials stored as GitHub Secrets.

      - name: Build image and push backend
        uses: docker/build-push-action@v3
        with:
          context: ./backend/simple-api-student
          # Context: The directory containing the Dockerfile for the backend.
          tags: ${{ secrets.DOCKERHUB_USERNAME }}/tp-devops-backend:latest
          # Tag: The name of the Docker image, with 'latest' as the version.
          push: true
          # Push: Automatically push the built image to DockerHub.

      - name: Build and push database image
        uses: docker/build-push-action@v3
        with:
          context: ./db
          # Context: The directory containing the Dockerfile for the database.
          tags: ${{ secrets.DOCKERHUB_USERNAME }}/tp-devops-database:latest
          # Tag: The name of the Docker image for the database.
          push: true
          # Push: Automatically push the database image to DockerHub.

      - name: Build and push frontend image
        uses: docker/build-push-action@v3
        with:
          context: ./http
          # Context: The directory containing the Dockerfile for the frontend.
          tags: ${{ secrets.DOCKERHUB_USERNAME }}/tp-devops-frontend:latest
          # Tag: The name of the Docker image for the frontend.
          push: true
          # Push: Automatically push the frontend image to DockerHub.
```
# TP3

### Version commenté inventory

```bash
# Définition du groupe global "all", qui s'applique à tous les hôtes
all:
  vars:
    ansible_user: admin  # Définit l'utilisateur SSH par défaut
    ansible_ssh_private_key_file: ~/id_rsa  # Spécifie la clé privée SSH à utiliser pour se connecter
  children:
    prod:  # Groupe d'hôtes de production
      hosts:
        delattre.thib.takima.cloud  # Hôte distant sur lequel Ansible va exécuter ses tâches
```
### Version commenté premier playbook
```bash
- hosts: all
  gather_facts: true
  become: true

  tasks:
    # Install prerequisites for Docker
    - name: Install required packages
      apt:
        name:
          - apt-transport-https
          - ca-certificates
          - curl
          - gnupg
          - lsb-release
          - python3-venv
        state: latest
        update_cache: yes

    # Add Docker’s official GPG key
    - name: Add Docker GPG key
      apt_key:
        url: https://download.docker.com/linux/debian/gpg
        state: present

    # Set up the Docker stable repository
    - name: Add Docker APT repository
      apt_repository:
        repo: "deb [arch=amd64] https://download.docker.com/linux/debian {{ ansible_facts['distribution_release'] }} stable"
        state: present
        update_cache: yes

    # Install Docker
    - name: Install Docker
      apt:
        name: docker-ce
        state: present

    # Install Python3 and pip3
    - name: Install Python3 and pip3
      apt:
        name:
          - python3
          - python3-pip
        state: present

    # Create a virtual environment for Python packages
    - name: Create a virtual environment for Docker SDK
      command: python3 -m venv /opt/docker_venv
      args:
        creates: /opt/docker_venv # Only runs if this directory doesn’t exist

    # Install Docker SDK for Python in the virtual environment
    - name: Install Docker SDK for Python in virtual environment
      command: /opt/docker_venv/bin/pip install docker

    # Ensure Docker is running
    - name: Make sure Docker is running
      service:
        name: docker
        state: started
      tags: docker*
```
### Version commenté docker playbook + task
```bash
- hosts: all
  gather_facts: true
  become: true

  roles:
    - docker
```
```bash
---
# tasks/main.yml

# 1. Installer les prérequis pour Docker
- name: Install required packages
  apt:
    name:
      - apt-transport-https
      - ca-certificates
      - curl
      - gnupg
      - lsb-release
      - python3-venv
    state: latest
    update_cache: yes

# 2. Ajouter la clé GPG officielle de Docker
- name: Add Docker GPG key
  apt_key:
    url: https://download.docker.com/linux/debian/gpg
    state: present

# 3. Ajouter le dépôt APT stable de Docker
- name: Add Docker APT repository
  apt_repository:
    repo: "deb [arch=amd64] https://download.docker.com/linux/debian {{ ansible_facts['distribution_release'] }} stable"
    state: present
    update_cache: yes

# 4. Installer Docker
- name: Install Docker
  apt:
    name: docker-ce
    state: present

# 5. Installer Python3 et pip3
- name: Install Python3 and pip3
  apt:
    name:
      - python3
      - python3-pip
    state: present

- name: Create a virtual environment for Docker SDK
  command: python3 -m venv /opt/docker_venv
  args:
    creates: /opt/docker_venv # Ne s'exécute que si le répertoire n'existe pas

- name: Install Docker SDK for Python in virtual environment
  command: /opt/docker_venv/bin/pip install docker

- name: Make sure Docker is running
  service:
    name: docker
    state: started
  tags: docker
```
