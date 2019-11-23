# Command reference
My collection of useful commands.

## GIT
### Remove unnecessary files from git
Ths example removes Mac OS X files `.DS_Store` from all folders in the repo. Make sure to add it to `.gitignore` also!  
If you also want to remove files from disk, remove the `--cached` option.
```sh
find . -name .DS_Store -print0 | xargs -0 git rm -f --ignore-unmatch --cached
git commit -am "Removed unnecessary files"
git push
```

## MarkLogic
### Database configuration in json format
Get the json structure of database configurations: http://localhost:8002/manage/v2/. Navigate to the information you need and add `&format=json` to the URL.

### Run MarkLogic from Docker Hub
Run MarkLogic from the Docker Hub Community version. MarkLogic data will be persisted in the current (project) folder.
```sh
docker run -d -it \
    -p 8000-8020:8000-8020 \
    -v `pwd`/MarkLogic:/var/opt/MarkLogic \
    -e MARKLOGIC_INIT=true \
    -e MARKLOGIC_ADMIN_USERNAME=admin \
    -e MARKLOGIC_ADMIN_PASSWORD=admin \
    --name MarkLogic  \
    store/marklogicdb/marklogic-server:10.0-2-dev-centos
```
### Shell into container
`docker exec -it MarkLogic sh`

### Add converters to the Docker Container
In case of using the MarkLogic Docker image, you'll have to download and install the converters into the docker container. To get the installer navigate to http://developer.marklogic.com and grab the `Curl URL` for the OS you need (most likely CentOS).
```sh
docker exec -it MarkLogic sh
curl --output /tmp/MarkLogicConverters.rpm <Curl URL>
sudo yum install libgcc libgcc.i686 libstdc++ libstdc++.i686
sudo rpm -i /tmp/MarkLogicConverters.rpm
```

## Docker
### Saving an image to a TAR file
`docker save marklogic > marklogic-9.0-8.tar`

### Load an image from TAR file
```sh
docker load --input marklogic-9.0-8.tar
docker load < marklogic-9.0-8.tar
```

### Run an image as a container
`docker run -d --name=marklogic -p 8000-8002:8000-8002 marklogic:9.0-8`

### Shell into container
`docker exec -it <id> sh`

### Copy from/into container
```sh
docker cp <file> <container>:<path>
docker cp <container>:<file> <path>
```

### List all containers
```sh
docker container ls -a
docker ps -a
```

### List all images
`docker image ls -a`

### Removing containers and images
```sh
docker container rm <id>
docker image rm <id>
```