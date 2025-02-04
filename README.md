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

2-2
