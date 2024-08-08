# Oracle Tips

## Oracle Docker

A docker container for running Oracle 21c:

1. [steveswinsburg oracle21c-docker](https://github.com/steveswinsburg/oracle21c-docker) 

> **Background**
>> A while back, Oracle introduced the concept of container databases (CDB) and pluggable databases (PDB). Containers are used for multi-tenancy and contain pluggable databases. Pluggable databases are what you are probably used to, a self contained database that you connect to. By default, this image creates one CDB, and one PDB within that CDB.

> **Before you begin**
> * `git clone https://github.com/oracle/docker-images` : C:\devw\docker-images
> * Download the Oracle Database 21c binary **LINUX.X64_213000_db_home.zip** from https://www.oracle.com/database/technologies/oracle21c-linux-downloads.html
> * Put the zip in the OracleDatabase/SingleInstance/dockerfiles/21.3.0 directory. **Do not unzip it.**
> * In Docker Desktop, ensure you have a large enough amount of memory allocated. These instructions will set the total memory to 4000MB, so make sure Docker has a value higher than that.


> **Building**

> Better to use GitBash to run **buildContainerImage.sh** (if the cmd prompt doesnt have bash) in C:\devw\docker-images 
>> cd OracleDatabase/SingleInstance/dockerfiles
>> ./buildContainerImage.sh -v 21.3.0 -e

> **Running**
>> Make sure you set the **-v command** appropriately for your system.
>> * `docker run --name oracle21c -p 1521:1521 -p 5500:5500 -e ORACLE_PDB=orcl -e ORACLE_PWD=password -e INIT_SGA_SIZE=3000 -e INIT_PGA_SIZE=1000 -v C:/oracleApp/:/opt/oracle/oradata  -d  oracle/database:21.3.0-ee` 

**Commands after to test**

> * `wsl -l -a`
> A docker container for running Oracle 21c
 
- docker exec -it oracle21c sqlplus sys/password@//localhost:1521/ORCL as sysdba
- docker ps -a
- docker stop <container_id_or_name>
- docker rm <container_id_or_name>
- docker logs <container-id>

C:/oracleApp/:/opt/oracle/orada



> **links**
[oracle21c-docker](https://github.com/steveswinsburg/oracle21c-docker)

[Clone oracle/docker-images](https://github.com/oracle/docker-images/tree/main/OracleDatabase/SingleInstance/dockerfiles)

[oracle12c-docker](https://github.com/steveswinsburg/oracle12c-docker)

> Install Terminal Windows from Windows Store

## ORA-28002: the password will expire within 7 days

> Just run the code `ALTER USER SYSTEM2 IDENTIFIED BY SYSTEM2;`

[How to solve ORA-28002: the password will expire warning](https://www.youtube.com/watch?v=euLA7_z4eDc&t=113s)

[How to Fix ORA-28002 The Password Will Expire in 7 Days Errors](https://blogs.oracle.com/sql/post/how-to-fix-ora-28002-the-password-will-expire-in-7-days-errors)

### ORA-65066: As alterações especificadas devem se aplicar a todos os containers
[must apply to all containers](https://nripendraoracle.wordpress.com/2015/03/18/change-password-for-developer-day-ora-65066-the-specified-changes-must-apply-to-all-containers/)

[how to resolve ora-65040](https://logic.edchen.org/how-to-resolve-ora-65040-operation-not-allowed-from-within-a-pluggable-database/)

```sql
// 1.
alter user system identified by system ; //65066. 00000 -  "The specified changes must apply to all containers"
//2. 
alter user system identified by system container=all; //65040. 00000 -  "operation not allowed from within a pluggable database"
//3. To be able to perform such operations, we should switch to the root container.
show con_name; 
alter session set container=CDB$ROOT;
show con_name;
alter session set container=ORCL;
```

## DUMP

### Data Pump feature introduced in SQL Developer
This tutorial covers the Data Pump feature introduced in SQL Developer Release 3.1 https://www.oracle.com/webfolder/technetwork/tutorials/obe/db/sqldev/r31/datapump_OBE/datapump.html 

### Docker commands dump
1. docker run -d --name oracle21c -p 1521:1521 -p 5500:5500 -e ORACLE_PDB=ORCL -e ORACLE_SID=ORCL -e ORACLE_PWD=password oracle/database:21.3.0-ee
2. docker exec -it oracle21c sqlplus sys/password@//localhost:1521/ORCL as sysdba
3. docker exec -it oracle21c impdp system2/SYSTEM@//localhost:1521/ORCL directory=MY_DIRECTORY dumpfile=CLdump_EOC_B2B_23032023.dmp show=Y
4. docker ps -a
5. docker stop <container_id_or_name> e docker rm <container_id_or_name>
6. docker logs <container-id>

### ORA-56935: Existing Datapump Jobs Are Using A Different Version

[ORA-56935](https://dbaclass.com/article/ora-56935-existing-datapump-jobs-are-using-a-different-version/)

### 
```sql
-- DROP USER
DROP USER CBIO18_POSTPAID CASCADE;   

-- DROP TABLESPACE
DROP TABLESPACE CBIO18_POSTPAID including contents and datafiles;

-- TO CREATE

CREATE  TABLESPACE CBIO18_POSTPAID
   datafile 'CBIO18_POSTPAID_01.dbf' SIZE 500M AUTOEXTEND ON
   NEXT 100M MAXSIZE 1G; 

alter session set "_ORACLE_SCRIPT"=true;
-- USER SQL
CREATE USER CBIO18_POSTPAID IDENTIFIED BY CBIO18_POSTPAID temporary tablespace temp default tablespace CBIO18_POSTPAID; 

GRANT ALL PRIVILEGES TO CBIO18_POSTPAID;

 
impdp CBIO18_POSTPAID/CBIO18_POSTPAID@orcl schemas=CBIO18_POSTPAID directory=D_DUMP dumpfile=CBIO18_POSTPAID.dmp logfile=CBIO18_POSTPAID.log 

expdp CBIO18_POSTPAID/CBIO18_POSTPAID@orcl schemas=CBIO18_POSTPAID directory=D_DUMP dumpfile=CBIO18_POSTPAID.dmp logfile=CBIO18_POSTPAID.log 
```

Final code:

```sql
----------------------------------------------------------------------------------------------------
----------------------------------------CBIO18_POSTPAID-------------------------------------------
----------------------------------------------------------------------------------------------------

-- TO DELETE

-- DROP USER
DROP USER CBIO18_POSTPAID CASCADE;   

-- DROP TABLESPACE
DROP TABLESPACE CBIO18_POSTPAID including contents and datafiles;

-- TO CREATE

CREATE  TABLESPACE CBIO18_POSTPAID
   datafile 'CBIO18_POSTPAID_01.dbf' SIZE 500M AUTOEXTEND ON
   NEXT 100M MAXSIZE 1G; 

show con_name;

alter session set "_ORACLE_SCRIPT"=true;
-- USER SQL
CREATE USER CBIO18_POSTPAID IDENTIFIED BY CBIO18_POSTPAID temporary tablespace temp default tablespace CBIO18_POSTPAID; 

GRANT ALL PRIVILEGES TO CBIO18_POSTPAID;

 select * from dba_directories where directory_name='DATA_PUMP_DIR';  
 
impdp CBIO18_POSTPAID/CBIO18_POSTPAID@orcl schemas=CBIO18_POSTPAID directory=MY_DIRECTORY dumpfile=CBIO18_POSTPAID.dmp logfile=CBIO18_POSTPAID.log 

expdp CBIO18_POSTPAID/CBIO18_POSTPAID@orcl schemas=CBIO18_POSTPAID directory=D_DUMP dumpfile=CBIO18_POSTPAID.dmp logfile=CBIO18_POSTPAID.log 

docker exec -it oracle21c impdp CBIO18_POSTPAID/CBIO18_POSTPAID@//localhost:1521/ORCL directory=MY_DIRECTORY dumpfile=CBIO18_POSTPAID.dmp show=Y
```

Step by step:

> 1. Connect in the SYSDBA in SQLDeveloper
> 2. Run the commands to create the user, find the commands in SQL file in the DUMP Zip to know what is the original name of user.
```SQL
-- DROP USER
DROP USER CBIO18_POSTPAID CASCADE;   

-- DROP TABLESPACE
DROP TABLESPACE CBIO18_POSTPAID including contents and datafiles;

-- TO CREATE

CREATE  TABLESPACE CBIO18_POSTPAID
   datafile 'CBIO18_POSTPAID_01.dbf' SIZE 500M AUTOEXTEND ON
   NEXT 100M MAXSIZE 1G; 
   
CREATE USER CBIO18_POSTPAID IDENTIFIED BY CBIO18_POSTPAID temporary tablespace temp default tablespace CBIO18_POSTPAID; 

GRANT ALL PRIVILEGES TO CBIO18_POSTPAID;
```
> 3. Put the DUMP file on c:/oracleApp : CBIO18_POSTPAID.DMP
> 4. Run the commands to import dump on Terminal Docker of Oracle21c
```SQL
impdp CBIO18_POSTPAID/CBIO18_POSTPAID@orcl schemas=CBIO18_POSTPAID directory=MY_DIRECTORY dumpfile=CBIO18_POSTPAID.dmp logfile=CBIO18_POSTPAID.log 
```
> 5. Open the metadata in EOC and tried the Upgrade System.
> 5.1 Maybe you will need first configurate the metadata with th JAR misssing so:
>> a. From [Oracle Cloud](https://www.oracle.com/cloud/sign-in.html?redirect_uri=https%3A%2F%2Fcloud.oracle.com%2F)
>> * Type **tdecloud** and choose OracleIdentifyCloudService
>> * Choose at the top menu on the right the **US East (Ashburn)**
>> * Search for **bucket_tools**
>> * Expand the AIR and go to EOC to get the last package to download, the same deployed in production, like EOC_CLPE_FS_R4A_CH_2023120501_EOC_ENT495.zip
>> * Extract the JARs from Metadata/Templates and copy then to your branck.
>> b. From [Artifactory](http://10.49.3.120:8080/artifactory/webapp/#/home)
>> * Search for the package like FS_R4A_CH_2023120501_EOC_ENT495
>> * Click to download it
