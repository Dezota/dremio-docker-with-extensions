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