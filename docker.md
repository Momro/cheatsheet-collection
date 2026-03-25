# docker-cheatsheat

```
# Install docker
sudo apt install docker.io docker-compose -y

# add yourself to the docker admin group
sudo usermod -aG docker $(whoami)

# activate docker group change
newgrp docker
```

Test if docker was installed correctly with a simple "hello world"-container:
```
docker run --rm hello-world
```

Run docker container, and stop it

```
docker-compose up -d
docker-compose down
```

See what's running
```
docker ps
```

Inspect a container's log file (have to be in that folder)

```
docker-compose logs -f
```

Update container

```
docker-compose down
docker-compose pull
docker-compose up -d
```
