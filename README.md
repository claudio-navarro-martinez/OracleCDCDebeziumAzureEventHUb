# OracleCDCDebeziumAzureEventHUb

docker pull store/oracle/database-enterprise:12.2.0.1

CONNECT sys/top_secret AS SYSDBA
alter system set db_recovery_file_dest_size = 10G;
alter system set db_recovery_file_dest = '/u01/app/oracle/data/oratest1/recovery_area' scope=spfile;
shutdown immediate
startup mount
alter database archivelog;
alter database open;
-- Should now "Database log mode: Archive Mode"
archive log list

ALTER SESSION SET CONTAINER=cdb$root;
ALTER DATABASE ADD SUPPLEMENTAL LOG DATA;

ALTER SESSION SET CONTAINER=orclpdb1
ALTER DATABASE ADD SUPPLEMENTAL LOG DATA; 

*** aunque en la vida real hay que hacerlo tabla a tabla

    ALTER TABLE OT.PRODUCT_CATEGORIES ADD SUPPLEMENTAL LOG DATA (ALL) COLUMNS;
    ALTER TABLE OT.EMPLOYEES ADD SUPPLEMENTAL LOG DATA (ALL) COLUMNS;
    ALTER TABLE OT.REGIONS ADD SUPPLEMENTAL LOG DATA (ALL) COLUMNS;
    ALTER TABLE OT.COUNTRIES ADD SUPPLEMENTAL LOG DATA (ALL) COLUMNS;
    ALTER TABLE OT.LOCATIONS ADD SUPPLEMENTAL LOG DATA (ALL) COLUMNS;
    ALTER TABLE OT.WAREHOUSES ADD SUPPLEMENTAL LOG DATA (ALL) COLUMNS;
    ALTER TABLE OT.PRODUCTS ADD SUPPLEMENTAL LOG DATA (ALL) COLUMNS;
    ALTER TABLE OT.CUSTOMERS ADD SUPPLEMENTAL LOG DATA (ALL) COLUMNS;
    ALTER TABLE OT.CONTACTS ADD SUPPLEMENTAL LOG DATA (ALL) COLUMNS;
    ALTER TABLE OT.ORDERS ADD SUPPLEMENTAL LOG DATA (ALL) COLUMNS;
    ALTER TABLE OT.ORDER_ITEMS ADD SUPPLEMENTAL LOG DATA (ALL) COLUMNS;
    ALTER TABLE OT.INVENTORIES ADD SUPPLEMENTAL LOG DATA (ALL) COLUMNS;


sqlplus sys/Oradoc_db1@//localhost:1521/ORCLCDB.localdomain as sysdba
CREATE TABLESPACE logminer_tbs DATAFILE '/u01/app/oracle/data/oratest1/logminer_tbs.dbf'
    SIZE 25M REUSE AUTOEXTEND ON MAXSIZE UNLIMITED;
  exit;

sqlplus sys/Oradoc_db1@//localhost:1521/ORCLPDB1.localdomain as sysdba
  CREATE TABLESPACE logminer_tbs DATAFILE '/u02/app/oracle/oradata/ORCL/orclpdb1/logminer_tbs.dbf'
    SIZE 25M REUSE AUTOEXTEND ON MAXSIZE UNLIMITED;
  exit;

sqlplus sys/Oradoc_db1@//localhost:1521/ORCLCDB.localdomain as sysdba

CREATE USER c##dbzuser IDENTIFIED BY dbz
    DEFAULT TABLESPACE logminer_tbs
    QUOTA UNLIMITED ON logminer_tbs
    CONTAINER=ALL;

  GRANT CREATE SESSION TO c##dbzuser CONTAINER=ALL;
  GRANT SET CONTAINER TO c##dbzuser CONTAINER=ALL;
  GRANT SELECT ON V_$DATABASE to c##dbzuser CONTAINER=ALL;
  GRANT FLASHBACK ANY TABLE TO c##dbzuser CONTAINER=ALL;
  GRANT SELECT ANY TABLE TO c##dbzuser CONTAINER=ALL;
  GRANT SELECT_CATALOG_ROLE TO c##dbzuser CONTAINER=ALL;
  GRANT EXECUTE_CATALOG_ROLE TO c##dbzuser CONTAINER=ALL;
  GRANT SELECT ANY TRANSACTION TO c##dbzuser CONTAINER=ALL;
  GRANT LOGMINING TO c##dbzuser CONTAINER=ALL;

  GRANT CREATE TABLE TO c##dbzuser CONTAINER=ALL;
  GRANT LOCK ANY TABLE TO c##dbzuser CONTAINER=ALL;
  GRANT CREATE SEQUENCE TO c##dbzuser CONTAINER=ALL;

  GRANT EXECUTE ON DBMS_LOGMNR TO c##dbzuser CONTAINER=ALL;
  GRANT EXECUTE ON DBMS_LOGMNR_D TO c##dbzuser CONTAINER=ALL;

  GRANT SELECT ON V_$LOG TO c##dbzuser CONTAINER=ALL;
  GRANT SELECT ON V_$LOG_HISTORY TO c##dbzuser CONTAINER=ALL;
  GRANT SELECT ON V_$LOGMNR_LOGS TO c##dbzuser CONTAINER=ALL;
  GRANT SELECT ON V_$LOGMNR_CONTENTS TO c##dbzuser CONTAINER=ALL;
  GRANT SELECT ON V_$LOGMNR_PARAMETERS TO c##dbzuser CONTAINER=ALL;
  GRANT SELECT ON V_$LOGFILE TO c##dbzuser CONTAINER=ALL;
  GRANT SELECT ON V_$ARCHIVED_LOG TO c##dbzuser CONTAINER=ALL;
  GRANT SELECT ON V_$ARCHIVE_DEST_STATUS TO c##dbzuser CONTAINER=ALL;
  ==============sin cdb

    CREATE USER OT IDENTIFIED BY dbz;
    

  CREATE USER cdbzuser IDENTIFIED BY dbz
    DEFAULT TABLESPACE logminer_tbs
    QUOTA UNLIMITED ON logminer_tbs;

  GRANT CREATE SESSION TO cdbzuser ;
  GRANT SET CONTAINER  TO cdbzuser ;
  GRANT SELECT ON V_$DATABASE to cdbzuser ;
  GRANT FLASHBACK ANY TABLE TO cdbzuser ;
  GRANT SELECT ANY TABLE TO cdbzuser ;
  GRANT SELECT_CATALOG_ROLE TO cdbzuser ;
  GRANT EXECUTE_CATALOG_ROLE TO cdbzuser ;
  GRANT SELECT ANY TRANSACTION TO cdbzuser ;
  GRANT LOGMINING TO cdbzuser ;

  GRANT CREATE TABLE TO cdbzuser ;
  GRANT LOCK ANY TABLE TO cdbzuser ;
  GRANT CREATE SEQUENCE TO cdbzuser ;

  GRANT EXECUTE ON DBMS_LOGMNR TO cdbzuser ;
  GRANT EXECUTE ON DBMS_LOGMNR_D TO cdbzuser ;

  GRANT SELECT ON V_$LOG TO cdbzuser ;
  GRANT SELECT ON V_$LOG_HISTORY TO cdbzuser ;
  GRANT SELECT ON V_$LOGMNR_LOGS TO cdbzuser ;
  GRANT SELECT ON V_$LOGMNR_CONTENTS TO cdbzuser ;
  GRANT SELECT ON V_$LOGMNR_PARAMETERS TO cdbzuser ;
  GRANT SELECT ON V_$LOGFILE TO cdbzuser ;
  GRANT SELECT ON V_$ARCHIVED_LOG TO cdbzuser ;
  GRANT SELECT ON V_$ARCHIVE_DEST_STATUS TO cdbzuser;

======================= kafka directory ==================
claudio-Latitude-E7470:
/home/claudio/kafka.azure/kafka_2.12-3.0.0/config

=================================
 ====== connect-distributed.properties
bootstrap.servers=cnmeh2.servicebus.windows.net:9093 
group.id=dbzconnect8-cluster-group

# connect internal topic names, auto-created if not exists
config.storage.topic=dbz8-connect-cluster-configs
offset.storage.topic=dbz8-connect-cluster-offsets
status.storage.topic=dbz8-connect-cluster-status

# internal topic replication factors - auto 3x replication in Azure Storage
config.storage.replication.factor=1
offset.storage.replication.factor=1
status.storage.replication.factor=1

key.converter=org.apache.kafka.connect.json.JsonConverter
value.converter=org.apache.kafka.connect.json.JsonConverter

key.converter.schemas.enable=true
value.converter.schemas.enable=true

offset.flush.interval.ms=1000

# required EH Kafka security settings
security.protocol=SASL_SSL
sasl.mechanism=PLAIN
sasl.jaas.config=org.apache.kafka.common.security.plain.PlainLoginModule required username="$ConnectionString" password="Endpoint=sb://cnmeh2.servicebus.windows.net/;SharedAccessKeyName=RootManageSharedAccessKey;SharedAccessKey=mwz/SQsJyfrfi+q1cIIMOC+DSkz5HXHNp8vcPayzMTQ=";

producer.security.protocol=SASL_SSL
producer.sasl.mechanism=PLAIN
producer.sasl.jaas.config=org.apache.kafka.common.security.plain.PlainLoginModule required username="$ConnectionString" password="Endpoint=sb://cnmeh2.servicebus.windows.net/;SharedAccessKeyName=RootManageSharedAccessKey;SharedAccessKey=mwz/SQsJyfrfi+q1cIIMOC+DSkz5HXHNp8vcPayzMTQ=";

consumer.security.protocol=SASL_SSL
consumer.sasl.mechanism=PLAIN
consumer.sasl.jaas.config=org.apache.kafka.common.security.plain.PlainLoginModule required username="$ConnectionString" password="Endpoint=sb://cnmeh2.servicebus.windows.net/;SharedAccessKeyName=RootManageSharedAccessKey;SharedAccessKey=mwz/SQsJyfrfi+q1cIIMOC+DSkz5HXHNp8vcPayzMTQ=";

topic.creation.enable = true
plugin.path=/home/claudio/kafka_2.12-3.0.0/libs
internal.value.converter=org.apache.kafka.connect.json.JsonConverter
offset.flush.timeout.ms=5000
internal.key.converter=org.apache.kafka.connect.json.JsonConverter


==========================================
./bin/connect-distributed.sh ./config/connect-distributed.properties

===========================================
{
    "name": "dbz8-oracle-connector",
    "config": {
        "connector.class" : "io.debezium.connector.oracle.OracleConnector",
        "database.hostname" : "81.202.200.160",
        "database.port" : "49154",
        "database.user" : "c##dbzuser",
        "database.password" : "dbz",
        "database.dbname" : "ORCLCDB.localdomain",
        "database.pdb.name" : "ORCLPDB1",
        "database.connection.adapter": "logminer",
        "database.server.name" : "server1",
        "tasks.max" : "1",
        "database.history.kafka.topic": "schema-changesdbz.inventory",
        "database.history.kafka.bootstrap.servers": "cnmeh2.servicebus.windows.net:9093",
        "snapshot.mode" : "initial",
        "transforms.unwrap.delete.handling.mode": "rewrite",
        "topic.creation.default.partitions": "1",
        "database.history.consumer.sasl.jaas.config": "org.apache.kafka.common.security.plain.PlainLoginModule required username=\"$ConnectionString\" password=\"Endpoint=sb://cnmeh2.servicebus.windows.net/;SharedAccessKeyName=RootManageSharedAccessKey;SharedAccessKey=mwz/SQsJyfrfi+q1cIIMOC+DSkz5HXHNp8vcPayzMTQ=\";",
        "database.history.consumer.security.protocol": "SASL_SSL",
        "database.history.consumer.sasl.mechanism": "PLAIN",
        "database.history.producer.sasl.jaas.config": "org.apache.kafka.common.security.plain.PlainLoginModule required username=\"$ConnectionString\" password=\"Endpoint=sb://cnmeh2.servicebus.windows.net/;SharedAccessKeyName=RootManageSharedAccessKey;SharedAccessKey=mwz/SQsJyfrfi+q1cIIMOC+DSkz5HXHNp8vcPayzMTQ=\";",
        "database.history.producer.security.protocol": "SASL_SSL",
        "database.history.producer.sasl.mechanism": "PLAIN",
        "topic.creation.default.replication.factor": "3",
        "schema.include.list" : "OT"
    }
}

======================================================================================
Before adding new conenctor it is needed to add the following topics:
schema-changesdbz.inventory

debezium , will create the following for our example:
server1
server1.OT.EMPLOYEES 
server1.OT.LOCATIONS
server1.OT.PRODUCT_CATEGORIES
server1.OT.REGIONS 
server1.OT.COUNTRIES 
server1.OT.WAREHOUSES 
server1.OT.PRODUCTS 
server1.OT.CUSTOMERS 
server1.OT.CONTACTS 
server1.OT.ORDERS 
server1.OT.ORDER_ITEMS 
server1.OT.INVENTORIES 

Add new connector
    curl -X POST -H "Content-Type: application/json" --data @mydbz-oracle-conenctor.properties http://localhost:8083/connectors

check status conenctor
    curl -s http://localhost:8083/connectors/dbz-oracle-connector/status
    curl -H "Accept:application/json" localhost:8083/connectors/
    curl -i -X GET -H "Accept:application/json" localhost:8083/connectors/dbz-oracle-connector
update configuration
    curl -i -X PUT -H "Accept:application/json" -H "Content-Type:application/json" --data @a2.oracle.cdc.properties localhost:8083/connectors/oracdc-testa3/config 
delete connector    
    curl -i -X DELETE localhost:8083/connectors/dbz-oracle-connector/

kafkacat -b cnmeh2.servicebus.windows.net:9093 -X security.protocol=sasl_ssl -X sasl.mechanism=PLAIN 
      -X sasl.username='$ConnectionString' 
      -X sasl.password='Endpoint=sb://cnmeh2.servicebus.windows.net/;SharedAccessKeyName=RootManageSharedAccessKey;SharedAccessKey=mwz/SQsJyfrfi+q1cIIMOC+DSkz5HXHNp8vcPayzMTQ=' 
      -L

kafkacat -b cnmeh2.servicebus.windows.net:9093 -X security.protocol=sasl_ssl -X sasl.mechanism=PLAIN \
      -X sasl.username='$ConnectionString' \
      -X sasl.password='Endpoint=sb://cnmeh2.servicebus.windows.net/;SharedAccessKeyName=RootManageSharedAccessKey;SharedAccessKey=mwz/SQsJyfrfi+q1cIIMOC+DSkz5HXHNp8vcPayzMTQ=' \
      -C -t server1.ot.employees -o beggining


============================================== Provisioning Oracle DB in Docker ==========================================
Oracle Database Server Docker Image Documentation
Oracle Database Server 12c R2 is an industry leading relational database server. The Oracle Database Server Docker Image contains the Oracle Database Server 12.2.0.1 Enterprise Edition running on Oracle Linux 7. This image contains a default database in a multitenant configuration with one pdb.

For more information on Oracle Database Server 12c R2 refer to http://docs.oracle.com/en/database/

Using this image
Accepting the terms of service
From the store.docker.com website accept Terms of Service for Oracle Database Enterprise Edition.

Login to Docker Store
Login to Docker Store with your credentials

$ docker login

Starting an Oracle Database Server instance
Starting an Oracle database server instance is as simple as executing

$ docker run -d -it --name <Oracle-DB> store/oracle/database-enterprise:12.2.0.1

where <Oracle-DB> is the name of the container and 12.2.0.1 is the Docker image tag.

The database server is ready to use when the STATUS field shows (healthy) in the output of docker ps.

Connecting to the Database Server Container
The default password to connect to the database with sys user is Oradoc_db1.

Connecting from within the container
The database server can be connected to by executing SQL*Plus,

$ docker exec -it <Oracle-DB> bash -c "source /home/oracle/.bashrc; sqlplus /nolog"

Connecting from outside the container
The database server exposes port 1521 for Oracle client connections over SQLNet protocol and port 5500 for Oracle XML DB. SQLPlus or any JDBC client can be used to connect to the database server from outside the container.

To connect from outside the container start the container with -P or -p option as,

$ docker run -d -it --name <Oracle-DB> -P store/oracle/database-enterprise:12.2.0.1

option -P indicates the ports are allocated by Docker. The mapped port can be discovered by executing

$ docker port <Oracle-DB> 1521/tcp -> 0.0.0.0:<mapped host port>

Using this <mapped host port> and <ip-address of host> create tnsnames.ora in the directory pointed to by environment variable TNS_ADMIN.

ORCLCDB=(DESCRIPTION=(ADDRESS=(PROTOCOL=TCP)(HOST=<ip-address of host>)(PORT=<mapped host port>))
    (CONNECT_DATA=(SERVER=DEDICATED)(SERVICE_NAME=ORCLCDB.localdomain)))
ORCLPDB1=(DESCRIPTION=(ADDRESS=(PROTOCOL=TCP)(HOST=<ip-address> of host)(PORT=<mapped host port>))
    (CONNECT_DATA=(SERVER=DEDICATED)(SERVICE_NAME=ORCLPDB1.localdomain)))
To connect from outside the container using SQL*Plus,

$ sqlplus sys/Oradoc_db1@ORCLCDB as sysdba

Custom Configurations
The Oracle database server container also provides custom configuration parameters for starting up the container. All the custom configuration parameters are optional. The following list of custom configuration parameters can be provided in the ENV file (ora.conf).

DB_SID
This parameter changes the ORACLE_SID of the database. The default value is set to ORCLCDB.

DB_PDB
This parameter modifies the name of the PDB. The default value is set to ORCLPDB1.

DB_MEMORY
This parameter sets the memory requirement for the Oracle server. This value determines the amount of memory to be allocated for SGA and PGA. The default value is set to 2GB.

DB_DOMAIN
This parameter sets the domain to be used for database server. The default value is localdomain.

To start an Oracle database server with custom configuration parameters

$ docker run -d -it --name <Oracle-DB> -P --env-file ora.conf store/oracle/database-enterprise:12.2.0.1

Ensure custom values for DB_SID, DB_PDB and DB_DOMAIN are updated in the tnsnames.ora.

Caveats
This Docker image has the following restrictions.

Supports a single instance database.
Dataguard is not supported.
Database options and patching are not supported.
Changing default password for SYS user
The Oracle database server is started with a default password Oradoc_db1. The password used during the container creation is not secure and should be changed. To change the password connect to the database with SQL*Plus and execute

alter user sys identified by <new-password>;

Resource Requirements
The minimum requirements for the container is 8GB of disk space and 2GB of memory.

Database Logs
The database alert log can be viewed with

$ docker logs <Oracle-DB>

where

Reusing existing database
This Oracle database server image uses Docker data volumes to store data files, redo logs, audit logs, alert logs and trace files. The data volume is mounted inside the container at /ORCL. To start a database with a data volume using docker run command,

$ docker run -d -it --name <Oracle-DB> -v OracleDBData:/ORCL store/oracle/database-enterprise:12.2.0.1

OracleDBData is the data volume that is created by Docker and mounted inside the container at /ORCL. The persisted data files can be reused with another container by reusing the OracleDBData data volume.

Using host system directory for data volume
To use a directory on the host system for the data volume,

$ docker run -d -it --name <Oracle-DB> -v /data/OracleDBData:/ORCL store/oracle/database-enterprise:12.2.0.1

where /data/OracleDBData is a directory in the host system.

Oracle Database Server 12.2.0.1 Enterprise Edition Slim Variant
The Slim Variant (12.2.0.1-slim tag) of EE has reduced disk space (4GB) requirements and a quicker container startup. This image does not support the following features - Analytics, Oracle R, Oracle Label Security, Oracle Text, Oracle Application Express and Oracle DataVault. To use the slim variant

$ docker run -d -it --name <Oracle-DB> store/oracle/database-enterprise:12.2.0.1-slim

where <Oracle-DB> is the name of the container and 12.2.0.1-slim is the Docker image tag.

