# Docker

## Copy data into container and from container

The commands are just taken from the [docs](https://docs.docker.com/engine/reference/commandline/cp/):

```bash
# from host file system into the container
docker cp ./some_file CONTAINER:/work
# from container into host filesystem
docker cp CONTAINER:/var/logs/ /tmp/app_logs
# from container to stdout; note cp command produces a tar stream
docker cp CONTAINER:/var/logs/app.log - | tar x -O | grep "ERROR"
```

## Run host file with executable from docker

To sync host and the docker container executable, one has to use the to bind [mount the host file via `-v` and set the working directory via `-w`](https://docs.docker.com/engine/reference/commandline/run/#volume)

```bash
# run executable mark in docker container with flag --title-from-h1
# on host file file_in_current_host_directory.md
# does not run on git bash windows...
docker run --rm -i --read-only -v "$(pwd)":"$(pwd)" -w "$(pwd)" kovetskiy/mark:latest mark \
    --title-from-h1 \
    -f file_in_current_host_directory.md
```

## Backup

### Container

See also the [official page](https://docs.docker.com/desktop/backup-and-restore/):

1. Run `docker commit my_container` to create locally a new image.
2. Either run `docker push` to push the new image to your registry (remote) or `docker image save -o  my_backup.tar my_new_image` to create the image as a local file.

To replay the backup, depending on your backup method, either make a `docker pull` to get it from the registry or a `docker image load -i my_backup.tar` followed by a `docker run` to run the image.

### Volumes

See also the [official page](https://docs.docker.com/storage/volumes/#back-up-restore-or-migrate-data-volumes/):

1. Create a new container `my_store_container` via

    ```bash
    docker run -v /dbdata --name my_store_container ubuntu /bin/bash
    ```

2. [Read the docs](https://docs.docker.com/storage/volumes/#back-up-restore-or-migrate-data-volumes/)...

    ```bash
    docker run --rm --volumes-from my_store_container -v $(pwd):/backup ubuntu tar cvf /backup/backup.tar /dbdata
    ```

## Docker Desktop

### Diagnostics

Docker-Desktop comes with some diagnostic tools that reside in `C:\Program Files\Docker\Docker\resources\com.docker.diagnose.exe`, e.g. `com.docker.diagnose.exe check`.
Also see the [official docs](https://docs.docker.com/desktop/troubleshoot/overview/).

### Move docker-desktop-data to another device

This recipe is taken from [kimcuonthenet](https://dev.to/kimcuonthenet/move-docker-desktop-data-distro-out-of-system-drive-4cg2) resp. [Stackoverflow](https://stackoverflow.com/questions/40465979/change-docker-native-images-location-on-windows-10-pro)

1. Quit Docker/Docker-Desktop
2. Open `powershell` or `cmd` and execute `wsl --shutdown`
3. Execute `wsl --export docker-desktop-data D:\docker-desktop-data.tar`
4. Execute `wsl --unregister docker-desktop-data`
5. Execute `wsl --import docker-desktop-data D:\docker-desktop-data D:\docker-desktop-data.tar --version 2`

## Run two docker-compose files in one command

Docker has some [combinable rules](https://docs.docker.com/compose/extends/). The simplest is to specify each yml with the `-f` flag:

```bash
docker compose -f postgres-compose.yml -f oracle-compose.yml up -d
```

## Installing / being root

For installing stuff, one might need to have root privileges. One way to do this is to log in as root:

```bash
docker exec -it -u root my_container_name bash
```

## Run command from docker container

One often needs to run binaries from within docker containers.
There is a basic distinction how to do this:

- `docker run`: run a command in a *new* Docker container
- `docker exec`: execute a command on an *already running* Docker container

To use the binaries, it is important to use the flag `--interactive` to keep `stdin` open and being able to interact with the binary:

```bash
# bad practice using the client of the container database
# for postgres: see https://www.cybertec-postgresql.com/en/docker-sudden-death-for-postgresql/
# get interactive shell
docker exec --interactive docker-container-name sqlplus "connection_url"
# execute command from stdin
echo "select 1 from dual;" | docker exec --interactive docker-container-name sqlplus "connection_url"
```
