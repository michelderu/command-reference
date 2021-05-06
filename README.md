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

## DataStax
### Run DSE as a docker container
Start DSE with Spark enabled.
Port mappings:
1. 7077 - Spark Master
2. 7080 - Spark Master UI
3. 9042 - Native Clients Port
4. 9077 - Always On SQL
5. 9091 - DSE Studio
6. 8888 - OPS Center
7. 10000 - Spark SQL ODBC/JDBC

```sh
docker run \
    -e DS_LICENSE=accept \
    -v `pwd`/dse/data/cassandra:/var/lib/cassandra \
    -v `pwd`/dse/data/spark:/var/lib/spark \
    -v `pwd`/dse/data/dsefs:/var/lib/dsefs \
    -v `pwd`/dse/log/cassandra:/var/log/cassandra \
    -v `pwd`/dse/log/spark:/var/log/spark \
    -p 7077:7077 \
    -p 7080:7080 \
    -p 9077:9077 \
    -p 9042:9042 \
    -p 10000:10000 \
    --name my-dse \
    -d datastax/dse-server:6.8.7-1 \
    -k
```
### Find out status
```sh
docker exec -it my-dse nodetool status
```

### Open interactive bash
Regular user vs root user
```sh
docker exec -it my-dse bash
docker exec -it --user root my-dse bash
```

### Open interactive CQLsh
```sh
docker exec -it my-dse cqlsh
```

### View logs
```sh
docker logs my-dse
```

## MarkLogic
### Database configuration in json format
Get the json structure of database configurations: http://localhost:8002/manage/v2/. Navigate to the information you need and add `&format=json` to the URL.

### Log information from the management endpoint (also useful in case of Encryption At Rest)
http://localhost:8002/manage/v2/logs?format=html&filename=ErrorLog.txt&start=2019-11-16%2012:00:00

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

### Gradle
#### Create an offline gradle package
See https://github.com/marklogic-community/ml-gradle/tree/master/examples/disconnected-project-using-plugins-and-gradlew

## Docker
### Build an image from a Dockerfile
`docker build -t <tag-name> .`

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

### Reverse engineer docker run command
Basics:
```sh
docker inspect foo/bar
```
Or using rekcod without installing the npm package.   
Replace `$CONTAINER_NAME` with the docker container name:
```sh
docker run -it --rm --volume /var/run/docker.sock:/var/run/docker.sock --privileged docker sh -c "apk add --no-cache nodejs nodejs-npm && npm i -g rekcod && rekcod $CONTAINER_NAME"
```
Or using rekcod as an npm package. 
Replace `$CONTAINER_NAME` with the docker container name:
```sh 
npm i -g rekcod
rekcod $CONTAINER_NAME
```

## Shell tools
### Show open TCP/UDP ports
```sh
sudo lsof -i -P -n
sudo lsof -i -P -n | grep LISTEN
```
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
  Free plan allows 60 lookups per minute
  IMPORTANT: Make sure to run this lookup single-threaded!
  When running as a custom step in Data Hub Framework:
    - Make sure to set batchSize to the total amount of documents in your collection
    - And set threadCount to 1. This makes sure the service lookups don't exceed 60 lookups per minute
    - Also make sure the default time limit for your MarkLogic App Server is set accordingly (ex. 1000 docs -> 1000 / 60 = 17 minutes)
*/
function geocode (address, api_key, log=false) {
  address = xdmp.urlEncode(address);
  
  // First see if we cached this address already
  let geodata;
  let cache = cts.search(cts.andQuery([cts.collectionQuery("geodata"), cts.jsonPropertyValueQuery('address', address)]));
  if (!fn.empty(cache)) {
    if (log) {console.log('GEOCODE: CACHED RESULT')};
    geodata = fn.head(cache).root.geodata;
  } else {
    // Call the geocoding service and sleep for 1000 ms
    let result = xdmp.httpGet('https://eu1.locationiq.com/v1/search.php?key=' + api_key + '&format=json&q=' + address);
    xdmp.sleep(1000);
    
    result = result.toArray();
    if (result[0].code == '200') {
      // When the HTTP response equals 200, take the result from the service
      if (log) {console.log('GEOCODE: MATCH (' + result[0].code + ' / ' + result[0].message + ')')};
      geodata = result[1].root[0];
    } else {
      // When there is another HTTP response, return an empty result
      if (log) {console.log('GEOCODE: ADDRES NOT FOUND (' + result[0].code + ' / ' + result[0].message + ')')};
      geodata = xdmp.toJSON({});
    }
    
    // Cache the data for future use
    let jsInsert = `
      declareUpdate();
      let geodoc = {
        "address": "` + address + `",
        "geodata": ` + geodata + `
      };
      xdmp.documentInsert(sem.uuidString(), geodoc, {collections: 'geodata'});`
    xdmp.eval(jsInsert, null);
  }

  return {
    coordinate: {
      "lat": geodata.lat,
      "lon": geodata.lon
    },
    "class": geodata.class,
    "type": geodata.type
  };
}

```

## Java
### Install on MacOS
Find versions:
```sh
brew update
brew search openjdk
```
Find minor version:
```sh
brew info adoptopenjdk8
```
Install a specific version:
```sh
brew install --cask adoptopenjdk8
```

### Java version switching
Find versions and path locations:
```sh
/usr/libexec/java_home -V
```
Or find a spexific version:
```sh
/usr/libexec/java_home -v 15
```
Set the Java version you want to use in `.bashrc`:
```sh
export JAVA_HOME=/Library/Java/JavaVirtualMachines/jdk-15.jdk/Contents/Home
```
Or add all options for easy switching (example):
```sh
export JAVA_8_HOME=$(/usr/libexec/java_home -v1.8)
export JAVA_9_HOME=$(/usr/libexec/java_home -v9)
export JAVA_10_HOME=$(/usr/libexec/java_home -v10)
export JAVA_11_HOME=$(/usr/libexec/java_home -v11)
export JAVA_12_HOME=$(/usr/libexec/java_home -v12)
export JAVA_13_HOME=$(/usr/libexec/java_home -v13)
export JAVA_14_HOME=$(/usr/libexec/java_home -v14)
export JAVA_15_HOME=$(/usr/libexec/java_home -v15)

alias java8='export JAVA_HOME=$JAVA_8_HOME'
alias java9='export JAVA_HOME=$JAVA_9_HOME'
alias java10='export JAVA_HOME=$JAVA_10_HOME'
alias java11='export JAVA_HOME=$JAVA_11_HOME'
alias java12='export JAVA_HOME=$JAVA_12_HOME'
alias java13='export JAVA_HOME=$JAVA_13_HOME'
alias java14='export JAVA_HOME=$JAVA_14_HOME'
alias java15='export JAVA_HOME=$JAVA_15_HOME'

# default to Java 15
java15
```
And then switch by:
```sh
java8
java -version
```

# Concepts

## Kubernetes
From https://medium.com/google-cloud/kubernetes-101-pods-nodes-containers-and-clusters-c1509e409e16
### Nodes
A node is the smallest unit of computing hardware in Kubernetes. It is a representation of a single machine in your cluster. In most production systems, a node will likely be either a physical machine in a datacenter, or virtual machine hosted on a cloud provider like Google Cloud Platform.
### Cluster
In Kubernetes, nodes pool together their resources to form a more powerful machine.  
It shouldn’t matter to the program, or the programmer, which individual machines are actually running the code.
### Persistent volumes
To store data permanently, Kubernetes uses Persistent Volumes.  
This can be thought of as plugging an external hard drive in to the cluster.  
Persistent Volumes provide a file system that can be mounted to the cluster, without being associated with any particular node.
### Containers
Programs running on Kubernetes are packaged as Linux containers.  
Containerization allows you to create self-contained Linux execution environments.  
Creating a container can be done programmatically, allowing powerful CI and CD pipelines to be formed.  
If each container has a tight focus, updates are easier to deploy and issues are easier to diagnose.  
### Pods
Unlike other systems you may have used in the past, Kubernetes doesn’t run containers directly; instead it wraps one or more containers into a higher-level structure called a pod.  
Any containers in the same pod will share the same resources and local network.  
Pods are used as the unit of replication in Kubernetes.  
Pods can hold multiple containers, but you should limit yourself when possible. Because pods are scaled up and down as a unit, all containers in a pod must scale together, regardless of their individual needs.
### Deployments
Pods are usually managed by one more layer of abstraction: the deployment.  
A deployment’s primary purpose is to declare how many replicas of a pod should be running at a time. When a deployment is added to the cluster, it will automatically spin up the requested number of pods, and then monitor them. If a pod dies, the deployment will automatically re-create it.  
### Ingress
By default, Kubernetes provides isolation between pods and the outside world.  
There are multiple ways to add ingress to your cluster. The most common ways are by adding either an Ingress controller, or a LoadBalancer.  

