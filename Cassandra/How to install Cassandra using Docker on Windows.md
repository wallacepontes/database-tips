# How to install Cassandra using Docker on Windows

## Table of Contents
- [How to install Cassandra using Docker on Windows](#how-to-install-cassandra-using-docker-on-windows)
  - [Table of Contents](#table-of-contents)
  - [How to install \[1\] \[2\]](#how-to-install-1-2)
  - [Explicitly map the ports](#explicitly-map-the-ports)
  - [run a container with specific ports](#run-a-container-with-specific-ports)
  - [Data outside the container](#data-outside-the-container)
  - [The --rm flag tells Docker to automatically remove the container when it stops](#the---rm-flag-tells-docker-to-automatically-remove-the-container-when-it-stops)
  - [LOAD DATA WITH CQLSH \[1\]](#load-data-with-cqlsh-1)
  - [References](#references)

## How to install [[1]](https://www.youtube.com/watch?v=roDo45cqHMM) [[2]](https://www.youtube.com/watch?v=_YlHsxCW9ig)

- **Step 1: Download and install**
  - [Docker Desktop](https://www.docker.com/products/docker-desktop/)
  - [Get Started with Apache Cassandra](https://cassandra.apache.org/_/quickstart.html)

- **Step 2: Get Cassandra image**

  ```cmd
  docker pull cassandra:latest
  ```

- **Step 3: Check if the image is pulled or not**

  ```cmd
  docker images
  ```

- **Step 4: Run Container Using Image**
  - Run container without the data outside it and making the ports accessible from outside the container network with `-p 7001:7001 -p 7199:7199 -p 9042:9042 -p 9160:9160`

  ```cmd
  docker run -p 7000:7000 -p 7001:7001 -p 7199:7199 -p 9042:9042 -p 9160:9160 --name cassandra -d cassandra:latest
  ```

  - Run container without `-p`, the ports are only accessible within the container network (in this case, `cassandra`), and not from localhost on the host machine. The `--rm` flag creates a temporary container that will be removed after it exits.

  ```cmd
  docker network create cassandra

  docker run --rm -d --name cassandra --hostname cassandra --network cassandra cassandra
  ```

  - **Recommended:** Run container with the data outside it, explicitly map the ports with `-p` flag and the flag `--rm` avoided.

  ```cmd
  docker network create cassandra

  docker run -d -p 7000:7000 -p 7001:7001 -p 7199:7199 -p 9042:9042 -p 9160:9160 --name cassandra --hostname cassandra --network cassandra -v C:\cassandra_data:/var/lib/cassandra cassandra
  ```

- **Step 5: Check container is running or not, get the CONSTAINER_ID**

  ```cmd
  docker ps
  ```

- **Step 6: Get inside the container and Start cqlsh (cassandra shell)**

  ```cmd
  docker exec -it CONSTAINER_ID bash
  cqlsh
  ```

- **Step 7: Create & Describe Keyspace / create table and insert record**

  ```sql
  create keyspace test with replication={'class':'SimpleStrategy', 'replication_factor':1};

  desc keyspaces;

  use test;

  create table student(ID int primary key, name text, city text, fees varint, contact varint);

  insert into student(id, name, city) values(1, 'ABC', 'XYZ');

  select * from student;
  ```

## Explicitly map the ports

You’re absolutely correct. When using `-p` with Docker, it maps the specified port on the host to the same port in the container, making it accessible from outside the container network. Without `-p`, the ports are only accessible within the container network (in this case, `cassandra`), and not from `localhost` on the host machine.

Try running Cassandra with the following command to explicitly map the ports:

```bash
docker run -d --name cassandra -p 7000:7000 -p 7001:7001 -p 7199:7199 -p 9042:9042 -p 9160:9160 cassandra:latest
```

Afterward, you should be able to connect with `dsbulk` or `cqlsh` from your host machine at `127.0.0.1:9042`. Let me know if this resolves the issue!

## run a container with specific ports

To run a container with specific ports exposed to the host machine, you can use the `-p` flag in the `docker run` command. For example, running a container with the following ports exposed:

```bash
docker run -d -p 7001:7001 -p 7199:7199 -p 9042:9042 -p 9160:9160 <container_name>
```

This will bind the container's internal ports (`7001`, `7199`, `9042`, and `9160`) to the host machine's corresponding ports. You can verify if these ports are accessible from outside the container and the host machine by following these steps:

1. **Check if the Container is Running**:
   First, ensure the container is running and has the ports exposed:

   ```bash
   docker ps
   ```

   This will list the running containers and show the ports mapped between the container and the host. You should see something like:

   ```
   CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS             PORTS                                          NAMES
   <container_id>      <image_name>         "/bin/bash"              <time>              Up <time>          0.0.0.0:7001->7001/tcp, 0.0.0.0:7199->7199/tcp, 0.0.0.0:9042->9042/tcp, 0.0.0.0:9160->9160/tcp   <container_name>
   ```

   This confirms the container ports are mapped to the host.

2. **Check if the Ports are Listening on the Host**:
   Run the following command on the host machine to check if the specified ports are open and listening:

   ```bash
   netstat -tuln | grep -E '7001|7199|9042|9160'
   ```

   or use `ss` if `netstat` is not available:

   ```bash
   ss -tuln | grep -E '7001|7199|9042|9160'
   ```

   This will show if the ports are in a `LISTEN` state. You should see something like:

   ```
   tcp    LISTEN     0      128    0.0.0.0:7001              0.0.0.0:*
   tcp    LISTEN     0      128    0.0.0.0:7199              0.0.0.0:*
   tcp    LISTEN     0      128    0.0.0.0:9042              0.0.0.0:*
   tcp    LISTEN     0      128    0.0.0.0:9160              0.0.0.0:*
   ```

3. **Test External Connectivity**:
   You can now test if these ports are accessible externally. Use `nc`, `telnet`, or a browser for HTTP-based ports:

   ```bash
   # Using netcat (for any protocol)
   nc -zv <host_ip> 7001
   nc -zv <host_ip> 7199
   nc -zv <host_ip> 9042
   nc -zv <host_ip> 9160
   ```

   If you're testing HTTP or web-based services, use `curl` to check if the ports respond:

   ```bash
   curl http://<host_ip>:7001
   curl http://<host_ip>:7199
   curl http://<host_ip>:9042
   curl http://<host_ip>:9160
   ```

4. **Check Kubernetes Port Availability (if applicable)**:
   If your Docker container is part of a Kubernetes pod or running within the cluster, use `kubectl` to check for the port mappings within Kubernetes, and whether the container is exposed externally (via `NodePort` or `LoadBalancer`):

   ```bash
   kubectl get pods -o wide
   kubectl get svc -o wide
   ```

If all these steps confirm that the ports are open and responding on the host machine, then your container ports are successfully exposed to the outside. Let me know if you need more assistance!

## Data outside the container

Running `docker rm cassandra` will only remove the container itself, not the data stored in your volume (`C:\cassandra_data`). The data is preserved because it's saved outside the container in the mounted volume. When you start a new Cassandra container with the same volume mount, it will continue using the data stored in `C:\cassandra_data`.

If you want to ensure the data persists, here’s a quick summary:

1. **Stop and remove the container**:
   ```bash
   docker stop cassandra
   docker rm cassandra
   ```

2. **Recreate the container** (data will remain intact in `C:\cassandra_data`):
   ```bash
   docker run -d -p 7000:7000 -p 7001:7001 -p 7199:7199 -p 9042:9042 -p 9160:9160 --name cassandra --hostname cassandra --network cassandra -v C:\cassandra_data:/var/lib/cassandra cassandra
   ```

Your data should remain safe using this approach. Let me know if you need further guidance!

##  The --rm flag tells Docker to automatically remove the container when it stops

You're correct. The `--rm` flag tells Docker to automatically remove the container when it stops, which means any data inside the container is lost unless you've configured persistent storage.

To retain your Cassandra data across restarts, you can set up **persistent volumes**. This way, your keyspace, tables, and data will persist even if the container is stopped or removed. Here’s how:

**Step 1.** Start Cassandra with a Persistent Volume

1. Choose or create a directory on your host to store Cassandra data. For example, on Windows, you might use `C:\cassandra_data`.

2. Run Cassandra with a volume mapped to the data directory in the container:

   ```bash
   docker run -d --name cassandra_container -v C:\cassandra_data:/var/lib/cassandra cassandra
   ```

   - `-d`: Runs the container in detached mode.
   - `--name cassandra_container`: Assigns a name to the container so you can refer to it easily.
   - `-v C:\cassandra_data:/var/lib/cassandra`: Maps `C:\cassandra_data` on your host to `/var/lib/cassandra` in the container, where Cassandra stores its data files.

**Step 2.** Verify Data Persistence

Now, if you stop or remove the container, the data will persist in `C:\cassandra_data`. When you restart Cassandra, it will use the data stored in this directory:

1. **Stop the container**:

   ```bash
   docker stop cassandra_container
   ```

2. **Restart the container** without data loss:

   ```bash
   docker start cassandra_container
   ```

This setup ensures your keyspace, tables, and data are retained between container restarts or recreations. Let me know if you have questions on configuring or working with this setup!

## LOAD DATA WITH CQLSH [[1]](https://cassandra.apache.org/_/quickstart.html)

The CQL shell, or cqlsh, is one tool to use in interacting with the database. We’ll use it to load some data into the database using the script you just saved.

```cmd
docker run --rm --network cassandra -v "$(pwd)/data.cql:/scripts/data.cql" -e CQLSH_HOST=cassandra -e CQLSH_PORT=9042 -e CQLVERSION=3.4.6 nuvo/docker-cqlsh
```

> Note: The cassandra server itself (the first docker run command you ran) takes a few seconds to start up. The above command will throw an error if the server hasn’t finished its init sequence yet, so give it a few seconds to spin up.

This command doesn't mencioned the stop and remove the current container.

You’re correct; the command given on the Cassandra site is meant to **run cqlsh in a separate container temporarily** and connect it to the Cassandra container that’s already running. The command uses the `--rm` flag, which ensures that this additional container is removed automatically after it finishes executing. You don't need to stop or remove your Cassandra server container to use this command.

Here’s a breakdown of what each part of this command does:

- `docker run --rm` creates a temporary container that will be removed after it exits.
- `--network cassandra` connects to a Docker network named `cassandra`. This network should already exist and should include the Cassandra container.
- `-v "$(pwd)/data.cql:/scripts/data.cql"` mounts the `data.cql` script file from your current directory (`$(pwd)`) into the container’s `/scripts/data.cql`.
- `-e CQLSH_HOST=cassandra` and `-e CQLSH_PORT=9042` set environment variables for the `cqlsh` command to connect to the host (`cassandra`) and port (`9042`) of the Cassandra container.
- `-e CQLVERSION=3.4.6` specifies the CQL version to be used.

The command runs `nuvo/docker-cqlsh`, which is a Docker image containing `cqlsh`. This allows you to execute CQL commands from a script or interact with Cassandra without needing `cqlsh` installed locally. 

>**Notes:**
>
>1. **Ensure the Cassandra container is fully up and running** before running this command to avoid connection issues.
>2. **No need to stop/remove the Cassandra container** – this command only creates a temporary container to run `cqlsh` and is independent of the Cassandra server container.

## References

1. [How to install Cassandra using Docker on Windows](https://www.youtube.com/watch?v=roDo45cqHMM)