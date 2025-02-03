# DevOps_CPE

docker network create app-network

docker volume create pgdata

docker run -d --name postgres-container \
    --network app-network \
    -v pgdata:/var/lib/postgresql/data \
    -e POSTGRES_DB=mydb \
    -e POSTGRES_USER=myuser \
    -e POSTGRES_PASSWORD=mypassword

# test de connexion à la base 
docker exec -it postgres-container psql -U usr -d db

# Question TP
1-1 : 
On utilise le flag -e pour définir les variables d’environnement lors de l’exécution du conteneur
Cela a plusieurs avantages :
- Sécurité : Éviter de stocker des informations sensibles (ex. mots de passe) dans Dockerfile qui pourraient être exposés dans un repo.
- Flexibilité : Permet de modifier les valeurs sans reconstruire l’image
