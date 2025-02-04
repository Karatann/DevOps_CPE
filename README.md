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

## Question TP

1-1 :
On utilise le flag -e pour définir les variables d’environnement lors de l’exécution du conteneur
Cela a plusieurs avantages :

- Sécurité : Éviter de stocker des informations sensibles (ex. mots de passe) dans Dockerfile qui pourraient être exposés dans un repo.
- Flexibilité : Permet de modifier les valeurs sans reconstruire l’image

1-2 :
Un volume Docker permet de garder les données de PostgreSQL, même si le conteneur est arrêté ou supprimé. Sans ça, toutes les données stockées dans la base de données seraient perdues à la suppression du conteneur.

1-3 :
On commente le code

1-4 :
On utilise multistage build car à l'aide de ce procédé l'image n'est générée qu'à partir du dernier from, séparant l'environnement de build et l'environnement de run. Cela permet de créer des images plus légères (ici sans supporter le jdk)

1-5 :
Un reverse proxy est essentiel pour les raisons suivantes :
Abstraction des services backend : Cache l'URL réelle et le port du backend, ce qui rend la communication plus sécurisée et flexible

Docker Compose est un outil essentiel pour orchestrer des applications multi-conteneurs. Voici pourquoi :

Simplification des déploiements :

Permet de définir et de gérer plusieurs conteneurs dans un seul fichier docker-compose.yml.
Automatisation du démarrage, de l’arrêt et de la configuration des services.
Gestion centralisée :

Configure les réseaux, volumes et dépendances entre les services en un seul endroit.
Évite les commandes Docker complexes et longues.
Portabilité :

Partagez le fichier docker-compose.yml pour garantir un environnement identique sur différentes machines ou équipes.
Scalabilité :

Facilite la montée en charge avec une simple commande (docker-compose up --scale).
Interopérabilité :

Intégration facile avec des outils CI/CD pour automatiser les pipelines de déploiement.

TP2

2-1 :
Testcontainers est une bibliothèque Java qui facilite l'utilisation de conteneurs Docker pour les tests, les avantages sont :
Tests d'intégration réalistes : Permet de tester des interactions avec des bases de données, des files d'attente, ou d'autres services externes dans un environnement Docker.
Évite les dépendances locales : Pas besoin d'installer PostgreSQL ou d'autres services localement.
Facilité d'utilisation : Configure et démarre automatiquement les conteneurs nécessaires pendant les tests.

2-3 For what purpose do we need to push docker images?

Avantages de pousser les images Docker sur Docker Hub :

- Accessibilité globale : Les images sont disponibles pour toute l'équipe et les environnements (production, staging, etc.).
- Standardisation : Partage d'environnements cohérents grâce à des conteneurs Docker reproductibles.
- Automatisation des déploiements : Les plateformes de déploiement (comme Kubernetes, AWS ECS) peuvent récupérer automatiquement les images Docker à partir de Docker Hub.
- Versionnement : Les images Docker peuvent être taguées (latest, v1.0, etc.) pour une gestion simplifiée des versions.

  comment version test backend :
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
        # Step 5: Run the Maven build and perform SonarCloud analysis using the SonarCloud Maven plugin.
        # -B: Enables batch mode for Maven (recommended in CI to avoid interactive prompts).
        # -Dsonar.projectKey: The key identifying your project in SonarCloud.
        # -Dsonar.organization: The organization key in SonarCloud.
        # SONAR_TOKEN: An authentication token stored as a GitHub secret.
```
comment version 


