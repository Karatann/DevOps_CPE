DevOps_CPE
Ce projet illustre l'utilisation de Docker pour mettre en place un environnement avec une base de données PostgreSQL, ainsi que l'application des bonnes pratiques DevOps telles que l'utilisation des volumes, des réseaux Docker, et des builds multistages.

Étapes de mise en place
1. Création du réseau Docker
Le réseau Docker permet la communication entre les conteneurs :

bash
Copier
Modifier
docker network create app-network
2. Création d'un volume pour la persistance des données
Le volume garantit que les données restent accessibles même après la suppression du conteneur :

bash
Copier
Modifier
docker volume create pgdata
3. Lancer le conteneur PostgreSQL avec des paramètres de connexion
Exécutez la commande suivante pour démarrer un conteneur PostgreSQL configuré :

bash
Copier
Modifier
docker run -d --name postgres-container \
    --network app-network \
    -v pgdata:/var/lib/postgresql/data \
    -e POSTGRES_DB=mydb \
    -e POSTGRES_USER=myuser \
    -e POSTGRES_PASSWORD=mypassword \
    postgres:14.1-alpine
4. Tester la connexion à la base de données
Utilisez la commande suivante pour vous connecter à PostgreSQL depuis le conteneur :

bash
Copier
Modifier
docker exec -it postgres-container psql -U myuser -d mydb
Questions du TP
1-1 : Pourquoi utiliser le flag -e pour les variables d'environnement ?
Le flag -e est utilisé pour définir les variables d’environnement lors de l’exécution d’un conteneur.
Avantages :

Sécurité : Empêche le stockage d’informations sensibles (comme les mots de passe) dans le Dockerfile, ce qui réduit les risques de fuite si le fichier est exposé.
Flexibilité : Permet de modifier les paramètres de configuration sans avoir à reconstruire l’image Docker.
1-2 : Pourquoi utiliser un volume Docker ?
Un volume Docker est utilisé pour persister les données de PostgreSQL.
Avantages :

Les données restent accessibles même si le conteneur est arrêté ou supprimé.
Sans volume, toutes les données de la base de données seraient perdues à chaque suppression du conteneur.
1-3 : Commenter le code
Le commentaire du code permet de documenter les étapes importantes, comme dans les commandes ci-dessus ou dans un fichier Dockerfile. Cela facilite la compréhension et la maintenance du projet.

1-4 : Pourquoi utiliser un multistage build ?
Un multistage build est utilisé pour créer des images Docker légères et optimisées.
Avantages :

Séparation des environnements : L’environnement de build (contenant le JDK et les outils de compilation) est distinct de l’environnement d’exécution (contenant uniquement le JRE minimal).
Réduction de la taille de l’image : Seules les ressources nécessaires à l’exécution finale sont incluses.
Amélioration de la sécurité : Les outils et les fichiers de build (comme les sources ou les dépendances) ne sont pas présents dans l’image finale.
