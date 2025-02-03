# DevOps_CPE

docker network create app-network

docker run -d --name postgres-container \
    --network app-network \
    -v pgdata:/var/lib/postgresql/data \
    -e POSTGRES_DB=mydb \
    -e POSTGRES_USER=myuser \
    -e POSTGRES_PASSWORD=mypassword \
