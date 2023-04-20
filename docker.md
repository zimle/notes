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

## Move docker-desktop-data to another device on Windows with WSL2

This receipe is taken from [kimcuonthenet](https://dev.to/kimcuonthenet/move-docker-desktop-data-distro-out-of-system-drive-4cg2) resp. [Stackoverflow](https://stackoverflow.com/questions/40465979/change-docker-native-images-location-on-windows-10-pro)

1. Quit Docker/Docker-Desktop
2. Open `powershell` or `cmd` and execute `wsl --shutdown`
3. Execute `wsl --export docker-desktop-data D:\docker-desktop-data.tar`
4. Execute `wsl --unregister docker-desktop-data`
5. Execute `wsl --import docker-desktop-data D:\docker-desktop-data D:\docker-desktop-data.tar --version 2`

## Run two docker-compose files in one command

Docker has some [composibility rules](https://docs.docker.com/compose/extends/). The simplest is to specify each yml with the `-f` flag:

```bash
docker compose -f postgres-compose.yml -f oracle-compose.yml up -d
```

## Installing / being root

For installing stuff, one might need to have root privileges. One way to do this is to log in as root:

```bash
docker exec -it -u root my_container_name bash
```