# Dremio Docker Build with Dezota Extensions

This docker build is different from the standard Dremio build in the
following ways:

1. Patch Dremio OSS to support longer Varchar and VarBinary fields (needed
to support GIS geometry length).  Keep in mind that these values only define 
an absolute maximum but will not inflate the size smaller values.

   - Moved default size to 10,485,759 bytes from 31,999 bytes
   - Moved maximum size to 10,485,759 bytes from 65,536 bytes 

2. Incorporate support for ClickHouse as a Datasource from:

   - https://github.com/Dezota/dremio-clickhouse-connector

3. Incorporate GIS functionality

   - https://github.com/Dezota/dremio-gis-extensions

## Building and Running Docker Image

```
cd ./build
make build
make run
```

## Using the Docker Hub Image

Get the image from Docker HUB:
```
docker pull dezota/dremio-oss-with-ext:20.1.0-2
```

Here is the digest for the this version on hub.docker.com:
```
20.1.0-2: digest: sha256:2beed6e1d3ee15617b3f5993027472a7847c5cf5dbf5b0a3444861b00ee85b9a size: 2415
````

*Comment out build line in docker-compose.yml:*
```
# build: ./build
```

Launch Dremio with Dezota Extensions in the Background:
```
docker-compose up -d
```
