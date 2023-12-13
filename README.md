# PostgreSQL-playground

## prerequisites
Docker installed; refer to the [Docker installation](https://docs.docker.com/engine/install/) 

## Setup the docker container

```
docker pull postgres:16

docker run -d  -p 5432:5432 --name database -e POSTGRES_PASSWORD=postgres postgres:16
```

### execute a sql command

```
list all databases
docker exec -it database psql -h localhost -U postgres -p 5432 -c "\l"

create a database
docker exec -it database psql -h localhost -U postgres -p 5432 -c "CREATE DATABASE main_database"
```

*  [Table inheritance](https://github.com/MarcGinger/PostgreSQL-playground/tree/main/table-inheritance) 

*  [Table partitioning](https://github.com/MarcGinger/PostgreSQL-playground/tree/main/table-partitioning) 



## Cleanup
## Tear down docker container

```
docker stop database

docker rm -f database

docker image rm postgres
```