# How to install Cassandra using Docker on Windows

## Table of Contents
- [How to install Cassandra using Docker on Windows](#how-to-install-cassandra-using-docker-on-windows)
  - [Table of Contents](#table-of-contents)
  - [How to install \[1\] \[2\]](#how-to-install-1-2)
  - [References](#references)

## How to install [[1]](https://www.youtube.com/watch?v=roDo45cqHMM) [[2]](https://www.youtube.com/watch?v=_YlHsxCW9ig)

- Download and install [Docker Desktop](https://www.docker.com/products/docker-desktop/)

- [Get Started with Apache Cassandra](https://cassandra.apache.org/_/quickstart.html)

- Get Cassandra image

```cmd
docker pull cassandra:latest
```

- Check if the image is pulled or not

```cmd
docker images
```

- Run Container Using Image

```cmd
docker run -p 7000:7000 -p 7001:7001 -p 7199:7199 -p 9042:9042 -p 9160:9160 --name cassandra -d cassandra:latest

or 

docker network create cassandra

docker run --rm -d --name cassandra --hostname cassandra --network cassandra cassandra
```

- Check container is running or not, get the CONSTAINER_ID

```cmd
docker ps
```

- Get inside the container and Start cqlsh (cassandra shell)

```cmd
docker exec -it CONSTAINER_ID bash
cqlsh
```

- Create & Describe Keyspace / create table and insert record

```sql
create keyspace test with replication={'class':'SimpleStrategy', 'replication_factor':1};

desc keyspaces;

use test;

create table student(ID int primary key, name text, city text, fees varint, contact varint);

insert into student(id, name, city) values(1, 'ABC', 'XYZ');

select * from student;
```

## References

1. [How to install Cassandra using Docker on Windows](https://www.youtube.com/watch?v=roDo45cqHMM)