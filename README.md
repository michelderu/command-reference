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

## Shell tools
### Split a file on newlines
Takes a file `1000_device_data.json` and splits it into numbered output files.
```sh
awk '{f = "device_data_" NR ".json"; print $0 > f; close(f)}' 1000_device_data.json
```

## Test data generators
### Mockaroo
https://www.mockaroo.com
### Person data
https://www.mobilefish.com/services/random_test_data_generator/random_test_data_generator.php

## Geo Coding an Address
This function uses the great services of https://my.locationiq.com/. Create an account and a project specific public API key.
```javascript
/*
  Geocode an address
  Uses an API KEY from https://my.locationiq.com/
  Free plan allows 1 lookup per second
  IMPORTANT: Make sure to run this lookup single-threaded!
*/
function geocode (address, api_key) {
  // Call the geocoding service and sleep for 1000 ms
  address = xdmp.urlEncode(address);
  let result = xdmp.httpGet('https://eu1.locationiq.com/v1/search.php?key=' + api_key + '&format=json&q=' + address);
  xdmp.sleep(1000);

  // When the HTTP response equals 200, take the lat/lon result
  result = result.toArray();
  if (result[0].code == '200') {
    console.log('GEOCODE: MATCH: ' + result);
    return {
      coordinate: {
        "lat": result[1].root[0].lat,
        "lon": result[1].root[0].lon
      },
      "class": result[1].root[0].class,
      "type": result[1].root[0].type
    };
  } else {
    console.log('GEOCODE: ADDRES NOT FOUND (' + result[0].code + ' / ' + result[0].message + ')');
    return {};
  }
}
```