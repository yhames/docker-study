# Multi-Container

- [Multi-Container](#multi-container)
  - [Set up multi-container](#set-up-multi-container)
  - [Add Requirements Features](#add-requirements-features)

![docker_multi-container.png](images/docker_multi-container.png)

## Set up multi-container

```bash
# create Docker Network
$ docker network create goals-net

# Run MongoDB
$ docker run -d --name mongodb --network goals-net --rm mongo

# Run Backend
$ docker run -d --name goals-backend -p 80:80 --network goals-net goals-node

# Run Frontend
$ docker run -d --name goals-frontend -p 3000:3000 -it goals-react
```

## Add Requirements Features

```bash
# Add Volume for MongoDB
# Add Environment Variables for Security Authentication in MongoDB
$ docker run -d --name mongodb -v data:/data/db -e MONGO_INITDB_ROOT_USERNAME=root -e MONGO_INITDB_ROOT_PASSWORD=password --network goals-net --rm mongo

# Add Bind Mount for Backend Live Reload
# Add Named Volume for Logs
# Add Anonymous Volume for Node Modules to use the node_modules in the container
# the order of the volume is important! less specific path first will be overwritten by more specific path
$ docker run -d --name goals-backend -p 80:80 -v {path_to_backend}:/app -v logs:/app/logs -v /app/node_modules -e MONGODB_USERNAME=root -e MONGODB_PASSWORD=password --network goals-net goals-node

# Add Bind Mount for Frontend Live Reload
$ docker run -d --name goals-frontend -p 3000:3000 -v {path_to_frontend/src}:/app/src -it goals-react
```