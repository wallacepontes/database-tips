# How to import and export data

## Table of Contents
- [How to import and export data](#how-to-import-and-export-data)
  - [Table of Contents](#table-of-contents)
  - [Peer-to-peer architecture](#peer-to-peer-architecture)
  - [Tools for importing and exporting data](#tools-for-importing-and-exporting-data)
  - [CQLSH (Cassandra Query Language Shell):](#cqlsh-cassandra-query-language-shell)
    - [Export data from a table in your Cassandra container](#export-data-from-a-table-in-your-cassandra-container)
    - [Error indicates that the `/data` directory doesn't exist](#error-indicates-that-the-data-directory-doesnt-exist)
    - [Copy the File from the Container to the Host](#copy-the-file-from-the-container-to-the-host)
    - [The --rm flag tells Docker to automatically remove the container when it stops](#the---rm-flag-tells-docker-to-automatically-remove-the-container-when-it-stops)
    - [LOAD DATA WITH CQLSH \[1\]](#load-data-with-cqlsh-1)
  - [DataStax Bulk Loader (DSBulk):](#datastax-bulk-loader-dsbulk)
    - [the `datastax/dsbulk` image](#the-datastaxdsbulk-image)
  - [Use Cassandra Bulk Loader (cassandra-loader or sstableloader)](#use-cassandra-bulk-loader-cassandra-loader-or-sstableloader)
  - [Spring Batch and Spring Data for Apache Cassandra](#spring-batch-and-spring-data-for-apache-cassandra)
    - [Steps to Insert Data into Cassandra using Spring Batch](#steps-to-insert-data-into-cassandra-using-spring-batch)
    - [Key Points:](#key-points)
  - [Kafka and Cassandra Integration: Apache Kafka as an intermediary between the stage tables and Cassandra could be an option.](#kafka-and-cassandra-integration-apache-kafka-as-an-intermediary-between-the-stage-tables-and-cassandra-could-be-an-option)
  - [Decrypt your encrypted data in Cassandra](#decrypt-your-encrypted-data-in-cassandra)
  - [Moving unencrypted data from a legacy system into Cassandra](#moving-unencrypted-data-from-a-legacy-system-into-cassandra)

## Peer-to-peer architecture

You can load data into one pod in Cassandra, and the other pods should automatically sync through Cassandra’s replication mechanism, depending on your cluster’s configuration. Cassandra uses a peer-to-peer architecture, meaning each node (or pod) can replicate data across the cluster to ensure consistency.
To confirm this setup will work seamlessly:

1. Check the replication factor for your keyspace. This factor determines how many copies of your data exist across the nodes.
2. Ensure the consistency level you plan to use aligns with your data accuracy and availability needs (e.g., QUORUM, ALL).

If your replication factor and consistency level are set correctly, loading data into one pod should indeed propagate it across the others as designed by the Cassandra engine.

## Tools for importing and exporting data

Apache Cassandra has several tools that can be used for importing and exporting data. Here are a few common ones:

1. **CQLSH (Cassandra Query Language Shell)**:
   - You can use `COPY` commands in CQLSH to import and export data.
     - **Exporting data**: `COPY keyspace_name.table_name TO 'file.csv';`
     - **Importing data**: `COPY keyspace_name.table_name FROM 'file.csv';`
   - It works well for smaller datasets, but for large datasets, performance can be an issue.

2. **SSTableloader**:
   - This tool is used for bulk importing data directly into Cassandra tables.
   - It can stream the data into the cluster, ensuring it's evenly distributed across nodes.
   - Best for importing large datasets or moving data from one Cassandra cluster to another.

3. **DataStax Bulk Loader (DSBulk)**:
   - DSBulk is a more advanced tool provided by DataStax that allows efficient import and export of data at scale.
     - **Export**: `dsbulk unload -k keyspace_name -t table_name -url file_path`
     - **Import**: `dsbulk load -k keyspace_name -t table_name -url file_path`
   - Supports CSV, JSON, and other formats, and it’s highly optimized for handling large amounts of data.

4. **Apache Spark with Cassandra Connector**:
   - If you have more complex data processing needs or need to import/export data across multiple systems, you can use Apache Spark with the Cassandra Connector to move data to and from Cassandra.

Each of these tools has its use case, with CQLSH being easy for small-scale operations and tools like SSTableloader or DSBulk better suited for larger datasets.

## CQLSH (Cassandra Query Language Shell):

- You can use COPY commands in CQLSH to import and export data.
  - Exporting data: COPY keyspace_name.table_name TO 'file.csv';
  - Importing data: COPY keyspace_name.table_name FROM 'file.csv';
- It works well for smaller datasets, but for large datasets, performance can be an issue.

### Export data from a table in your Cassandra container

**To export data from the `student` table in your Cassandra container to a CSV file accessible from your Windows `C:\` directory, follow these steps:**

**Step 1:** Use `COPY` Command in Cassandra

1. Start by entering the Cassandra container and launching `cqlsh`:

   ```bash
   docker exec -it <container_name> cqlsh
   ```

2. In `cqlsh`, use the `COPY` command to export the data:

   ```sql
   COPY test.student (id, name, city) TO '/data/student.csv' WITH HEADER = TRUE;
   ```

   This command creates a file `student.csv` in the `/data` directory of the container.

**Step 2:** Mount the `C:\` Directory to Access the CSV

To make the CSV file accessible on your Windows machine, you need to have a directory in your `C:\` drive mounted to the container. For example, if you want to map `C:\cassandra_data` to `/data` inside the container, you can start the container with:

   ```bash
   docker run -d --name cassandra_container -v C:\cassandra_data:/data cassandra
   ```

   If the container is already running, you’ll need to recreate it with the volume flag, or alternatively, copy the CSV file from the container to your host system after running the `COPY` command.

**Step 3:** Verify the File in Windows

1. After the `COPY` command completes, check `C:\cassandra_data` in your Windows file explorer.
2. You should find `student.csv` there with your exported data.

This method allows you to access the CSV export directly on your Windows file system.

### Error indicates that the `/data` directory doesn't exist

**The error indicates that the `/data` directory doesn't exist inside the Cassandra container. You’ll need to create the directory first, then retry the `COPY` command. Here’s how to do it:**

**Step 1:** Create the `/data` Directory Inside the Container

1. **Open a shell in the Cassandra container**:

   ```bash
   docker exec -it <container_name> bash
   ```

2. **Create the `/data` directory**:

   ```bash
   mkdir /data
   ```

3. **Exit the container shell**:

   ```bash
   exit
   ```

**Step 2:** Run the `COPY` Command Again in `cqlsh`

Now, go back to `cqlsh` and try running the `COPY` command again:

```sql
COPY test.student (id, name, city) TO '/data/student.csv' WITH HEADER = TRUE;

COPY test.student (id, name, city, fees, contact) FROM '/data/student2.csv' WITH HEADER = TRUE;

select * from test.student;
```

**Step 3:** Map `/data` to a Local Windows Directory (Optional)

If you want to make this CSV file accessible in a specific directory on your Windows system (e.g., `C:\cassandra_data`), you need to mount this directory when you start the container:

1. **Stop and remove the current container** (if needed):

   ```bash
   docker stop <container_name>
   docker rm <container_name>
   ```

2. **Start the container with volume mapping**:

   ```bash
   docker run -d --name <container_name> -v C:\cassandra_data:/data cassandra
   ```

After running the `COPY` command, you should find `student.csv` in `C:\cassandra_data` on your Windows system.

### Copy the File from the Container to the Host

You can use a similar approach to copy files from a Docker container to your host system without stopping or removing the container. You can use Docker’s `cp` command to copy files directly from the container to your local machine. Here’s how:

**Step 1:** Copy the File from the Container to the Host

Suppose you want to copy the `student.csv` file from `/data` in the Cassandra container to a directory on your host machine (e.g., `C:\cassandra_data`).

1. Run the `docker cp` command:

   ```bash
   docker cp <container_name>:/data/student.csv C:\cassandra_data\student.csv
   ```

   - `<container_name>`: Replace this with the name or ID of your Cassandra container.
   - `:/data/student.csv`: The path inside the container where the file is located.
   - `C:\cassandra_data\student.csv`: The destination path on your host machine.

This command will copy `student.csv` from the container to the specified directory on your Windows host.

**Example of the Complete Workflow:**

1. **Export the data to a CSV file** inside the container using `cqlsh`:

   ```sql
   COPY test.student (id, name, city) TO '/data/student.csv' WITH HEADER = TRUE;
   ```

2. **Copy the CSV file to your host system**:

   ```bash
   docker cp <container_name>:/data/student.csv C:\cassandra_data\student.csv

   docker cp C:\cassandra_data\student2.csv cassandra:/data/student2.csv
   ```

After this, `student.csv` should be available on your Windows machine under `C:\cassandra_data`. This method is straightforward and doesn’t require mounting directories or restarting containers.

###  The --rm flag tells Docker to automatically remove the container when it stops

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

### LOAD DATA WITH CQLSH [[1]](https://cassandra.apache.org/_/quickstart.html)

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


## DataStax Bulk Loader (DSBulk):

- DSBulk is a more advanced tool provided by DataStax that allows efficient import and export of data at scale.
  - Export: dsbulk unload -k keyspace_name -t table_name -url file_path
  - Import: dsbulk load -k keyspace_name -t table_name -url file_path
- Supports CSV, JSON, and other formats, and it’s highly optimized for handling large amounts of data.

### the `datastax/dsbulk` image

The error you’re seeing indicates that the `datastax/dsbulk` image is not available in the Docker registry, which is likely because an official `dsbulk` image doesn’t exist. Instead, we can install `dsbulk` locally and run it on your host system, or create a custom Docker image for it.

Here are both options:

**Option 1:** Install and Use `dsbulk` Locally

1. **Download dsbulk** from the [DataStax Bulk Loader GitHub page](https://github.com/datastax/dsbulk).

2. **Extract the downloaded file** and navigate to the `dsbulk` directory.

3. Run `dsbulk` commands directly from your command line (you'll need to point it to your Cassandra container using the host IP or container name).

   Here’s an example command to load data from `C:\cassandra_data\student_import.csv` into the `student` table:

   ```bash
   dsbulk load -h <Cassandra_host_name_or_IP> -k test -t student -url C:\cassandra_data\student_import.csv -b
   ```

**Option 2:** Create a Custom Docker Image for `dsbulk`

If you’d prefer to keep it all within Docker, here’s how to create a simple Docker image for `dsbulk`:

1. **Create a Dockerfile** to set up `dsbulk`:

   In a new directory, create a file named `Dockerfile` with the following contents:

   ```Dockerfile
   FROM openjdk:8-jre
   RUN apt-get update && apt-get install -y wget unzip
   RUN wget https://github.com/datastax/dsbulk/releases/download/v1.7.0/dsbulk-1.7.0.tar.gz
   RUN tar -xzf dsbulk-1.7.0.tar.gz && mv dsbulk-1.7.0 /opt/dsbulk
   ENV PATH="/opt/dsbulk/bin:$PATH"
   ```

2. **Build the Docker image**:

   ```bash
   docker build -t custom-dsbulk .
   ```

3. **Run dsbulk commands** using the custom image:

   ```bash
   docker run --rm -v C:\cassandra_data:/data --network cassandra custom-dsbulk load \
     -h <Cassandra_host_name_or_IP> \
     -k test \
     -t student \
     -url /data/student_import.csv \
     -b
   ```

This custom image setup allows you to use `dsbulk` within Docker similarly to how it’s described in the quick start guides, but with the flexibility to run it as needed without requiring a pre-made image.

## Use Cassandra Bulk Loader (cassandra-loader or sstableloader)

- sstableloader is Cassandra's built-in tool for bulk data loading, which is efficient and optimized for large datasets.
  - Convert your data into SSTable format.
  - Use sstableloader to load the data into the cluster.
- Fast, bypasses CQL overhead and requires extra steps to convert data into SSTable format.

## Spring Batch and Spring Data for Apache Cassandra

You can use **Spring Batch** to insert data directly into Apache Cassandra by integrating **Spring Data Cassandra** into your batch processing job. Here's an outline of how you can achieve this:

### Steps to Insert Data into Cassandra using Spring Batch

1. **Add Dependencies**:
   Include the necessary dependencies for Spring Batch and Spring Data Cassandra in your `pom.xml` (for Maven) or `build.gradle` (for Gradle).

   **Maven Example**:
   ```xml
   <dependency>
       <groupId>org.springframework.boot</groupId>
       <artifactId>spring-boot-starter-batch</artifactId>
   </dependency>
   <dependency>
       <groupId>org.springframework.boot</groupId>
       <artifactId>spring-boot-starter-data-cassandra</artifactId>
   </dependency>
   ```

2. **Configure Cassandra**:
   Set up your Cassandra connection properties in the `application.properties` or `application.yml` file:
   
   ```properties
   spring.data.cassandra.contact-points=localhost
   spring.data.cassandra.port=9042
   spring.data.cassandra.keyspace-name=mykeyspace
   spring.data.cassandra.username=cassandra
   spring.data.cassandra.password=cassandra
   ```

3. **Create an Entity Class**:
   Define your Cassandra entity that represents the table where the data will be inserted.

   ```java
   @Table("my_table")
   public class MyEntity {
       @PrimaryKey
       private String id;

       private String name;

       // Getters and Setters
   }
   ```

4. **Set up a Repository**:
   Use Spring Data Cassandra’s repository pattern to interact with the database. Create a repository interface to insert data into Cassandra.

   ```java
   public interface MyEntityRepository extends CassandraRepository<MyEntity, String> {
   }
   ```

5. **Create the Spring Batch Job**:
   Define a Spring Batch job that will read data (from a CSV, database, or any other source) and write it to Cassandra using the repository.

   - **Reader**: Can be any Spring Batch `ItemReader`, such as `FlatFileItemReader` (for reading from CSV) or `JdbcCursorItemReader` (for reading from a relational database).
   - **Writer**: The `ItemWriter` will use your Cassandra repository to insert data.

   Example:

   ```java
   @Configuration
   public class BatchConfig {

       @Autowired
       private MyEntityRepository myEntityRepository;

       @Bean
       public Job importJob(JobBuilderFactory jobBuilderFactory, StepBuilderFactory stepBuilderFactory, ItemReader<MyEntity> reader) {
           Step step = stepBuilderFactory.get("step")
                   .<MyEntity, MyEntity>chunk(100)
                   .reader(reader)
                   .writer(writer())
                   .build();

           return jobBuilderFactory.get("importJob")
                   .start(step)
                   .build();
       }

       @Bean
       public ItemWriter<MyEntity> writer() {
           return new ItemWriter<MyEntity>() {
               @Override
               public void write(List<? extends MyEntity> items) throws Exception {
                   myEntityRepository.saveAll(items);
               }
           };
       }

       // Reader definition here (could be FlatFileItemReader or any other reader)
   }
   ```

6. **Run the Job**:
   You can now run your Spring Batch job, and the data will be written directly into the Cassandra database.

### Key Points:
- **ItemReader**: Reads data from any source (file, DB, etc.).
- **ItemProcessor** (Optional): You can transform the data if needed.
- **ItemWriter**: Writes data to Cassandra using the `CassandraRepository`.
- Spring Data Cassandra handles all the low-level communication with Cassandra for you.

This approach leverages both Spring Batch’s batch processing capabilities and Spring Data Cassandra’s seamless interaction with the Cassandra database, allowing you to efficiently insert data in a batch-oriented way.

## Kafka and Cassandra Integration: Apache Kafka as an intermediary between the stage tables and Cassandra could be an option.

## Decrypt your encrypted data in Cassandra

To decrypt your encrypted data in **Cassandra** using the `ciphertextkey` field, you’ll need to:

1. **Read the encrypted data and the `ciphertextkey` from the table**.
2. **Use the appropriate decryption algorithm** (e.g., AES, RSA) along with the key to decrypt the encrypted data.

**Steps to Decrypt Encrypted Data in Cassandra Using Spring Batch**

Assuming the data encryption algorithm is known, and you have the `ciphertextkey` stored for each record, here's how you can integrate decryption into your Spring Batch job.

**Step 1: Fetch Encrypted Data**:
First, read the encrypted data from Cassandra, which includes both the `ciphertextkey` and the encrypted data field.

Example of an entity class that represents the encrypted data and the `ciphertextkey`:

```java
@Table("my_table")
public class MyEncryptedEntity {

    @PrimaryKey
    private String id;

    private String encryptedData;

    private String ciphertextkey;

    // Getters and Setters
}
```

**Step 2: Create a Decryption Method**:
Implement a method to decrypt the data using the `ciphertextkey` and the encryption algorithm used. For example, if AES encryption was used, you can write a decryption method using Java’s built-in `javax.crypto` package.

Here’s a sample decryption method using AES:

```java
import javax.crypto.Cipher;
import javax.crypto.spec.SecretKeySpec;
import java.util.Base64;

public class DecryptionUtil {

    private static final String ALGORITHM = "AES";

    public static String decrypt(String encryptedData, String ciphertextkey) throws Exception {
        byte[] decodedKey = Base64.getDecoder().decode(ciphertextkey);
        SecretKeySpec secretKey = new SecretKeySpec(decodedKey, ALGORITHM);
        
        Cipher cipher = Cipher.getInstance(ALGORITHM);
        cipher.init(Cipher.DECRYPT_MODE, secretKey);

        byte[] encryptedBytes = Base64.getDecoder().decode(encryptedData);
        byte[] decryptedBytes = cipher.doFinal(encryptedBytes);

        return new String(decryptedBytes);
    }
}
```

This method assumes that the `ciphertextkey` is stored in Base64 format and that AES encryption is used. Adjust it as necessary for your encryption algorithm.

**Step 3: Process the Data in Spring Batch**:
In Spring Batch, use an **ItemProcessor** to decrypt the data before writing it to the target location.

Example of a processor that decrypts the data:

```java
import org.springframework.batch.item.ItemProcessor;

public class DecryptionProcessor implements ItemProcessor<MyEncryptedEntity, MyDecryptedEntity> {

    @Override
    public MyDecryptedEntity process(MyEncryptedEntity item) throws Exception {
        // Decrypt the data
        String decryptedData = DecryptionUtil.decrypt(item.getEncryptedData(), item.getCiphertextkey());

        // Map the decrypted data to a new entity
        MyDecryptedEntity decryptedEntity = new MyDecryptedEntity();
        decryptedEntity.setId(item.getId());
        decryptedEntity.setDecryptedData(decryptedData);

        return decryptedEntity;
    }
}
```

In this processor:
- `MyEncryptedEntity` represents the data read from Cassandra, containing encrypted data.
- `MyDecryptedEntity` represents the entity with the decrypted data.

**Step 4: Write the Decrypted Data**:
Once the data is decrypted, you can either store it back in another Cassandra table or process it further. Here’s an example of writing the decrypted data back to a new Cassandra table:

```java
@Table("my_decrypted_table")
public class MyDecryptedEntity {

    @PrimaryKey
    private String id;

    private String decryptedData;

    // Getters and Setters
}
```

Then use the **ItemWriter** in Spring Batch to save the decrypted data:

```java
@Bean
public ItemWriter<MyDecryptedEntity> writer() {
    return new ItemWriter<MyDecryptedEntity>() {
        @Override
        public void write(List<? extends MyDecryptedEntity> items) throws Exception {
            // Assuming MyDecryptedEntityRepository is a CassandraRepository
            myDecryptedEntityRepository.saveAll(items);
        }
    };
}
```

**Step 5: Integrate the Processor into the Spring Batch Job**:

Finally, integrate everything into your batch job configuration:

```java
@Bean
public Job decryptJob(JobBuilderFactory jobBuilderFactory, StepBuilderFactory stepBuilderFactory, 
                      ItemReader<MyEncryptedEntity> reader, ItemWriter<MyDecryptedEntity> writer) {

    Step step = stepBuilderFactory.get("decryptStep")
            .<MyEncryptedEntity, MyDecryptedEntity>chunk(100)
            .reader(reader)
            .processor(new DecryptionProcessor())  // Decrypts data
            .writer(writer)  // Writes decrypted data
            .build();

    return jobBuilderFactory.get("decryptJob")
            .start(step)
            .build();
}
```

> **Summary:**
> 
> 1. **Read the encrypted data** from Cassandra using `ItemReader`.
> 2. **Decrypt the data** using an `ItemProcessor`, utilizing a decryption utility like `javax.crypto`.
> 3. **Write the decrypted data** back to Cassandra or any other destination using `ItemWriter`.

This approach works seamlessly in Spring Batch, allowing you to process large volumes of encrypted data in a batch job and securely decrypt them using the corresponding `ciphertextkey` stored in the table.

## Moving unencrypted data from a legacy system into Cassandra

For the purpose of your migration, where you are moving unencrypted data from a legacy system into Cassandra and need to **encrypt the data** before storing it in Cassandra tables, you can follow a process similar to the decryption example, but in reverse. Here’s how to do it:

**Steps to Encrypt Data from Legacy System and Store in Cassandra using Spring Batch**

1. **Extract Data from Legacy System**:
   You can use an **ItemReader** in Spring Batch to read the unencrypted data from the legacy system. This could be a relational database, flat files (CSV), or any other data source.

2. **Encrypt Data**:
   Use an **ItemProcessor** to encrypt the data before it is written into Cassandra. The encryption key (`ciphertextkey`) should be generated or provided as part of the migration process.

3. **Store Encrypted Data in Cassandra**:
   Use an **ItemWriter** to write the encrypted data along with the generated encryption key into Cassandra tables.

**Step 1.** **Encryption Logic**:
Let’s define an encryption utility that uses a strong algorithm like **AES** for encrypting the data. You can customize this based on your encryption needs.

Here’s an example of an encryption method using AES:

```java
import javax.crypto.Cipher;
import javax.crypto.KeyGenerator;
import javax.crypto.SecretKey;
import javax.crypto.spec.SecretKeySpec;
import java.util.Base64;

public class EncryptionUtil {

    private static final String ALGORITHM = "AES";

    public static String encrypt(String data, String secretKey) throws Exception {
        SecretKeySpec secretKeySpec = new SecretKeySpec(Base64.getDecoder().decode(secretKey), ALGORITHM);
        Cipher cipher = Cipher.getInstance(ALGORITHM);
        cipher.init(Cipher.ENCRYPT_MODE, secretKeySpec);

        byte[] encryptedBytes = cipher.doFinal(data.getBytes());
        return Base64.getEncoder().encodeToString(encryptedBytes);
    }

    public static String generateSecretKey() throws Exception {
        KeyGenerator keyGen = KeyGenerator.getInstance(ALGORITHM);
        keyGen.init(256); // AES-256
        SecretKey secretKey = keyGen.generateKey();
        return Base64.getEncoder().encodeToString(secretKey.getEncoded());
    }
}
```

- `encrypt()`: Takes plain text data and a secret key, encrypts it, and returns the encrypted data.
- `generateSecretKey()`: Generates a new AES key (you can use this method to generate a unique key per record or a shared key across records).

**Step 2.** **Read Unencrypted Data**:
You will need an **ItemReader** to read the unencrypted data from your legacy system. For example, if the legacy system is a relational database, you could use `JdbcCursorItemReader`.

```java
@Bean
public JdbcCursorItemReader<LegacyEntity> reader(DataSource dataSource) {
    JdbcCursorItemReader<LegacyEntity> reader = new JdbcCursorItemReader<>();
    reader.setDataSource(dataSource);
    reader.setSql("SELECT id, data FROM legacy_table");
    reader.setRowMapper(new LegacyEntityRowMapper());
    return reader;
}
```

Where `LegacyEntity` represents the unencrypted data from the legacy system.

**Step 3.** **Encrypt Data in an ItemProcessor**:
Now, use an **ItemProcessor** to encrypt the data and generate a `ciphertextkey` for each record before writing it to Cassandra.

```java
import org.springframework.batch.item.ItemProcessor;

public class EncryptionProcessor implements ItemProcessor<LegacyEntity, EncryptedEntity> {

    @Override
    public EncryptedEntity process(LegacyEntity item) throws Exception {
        // Generate encryption key
        String secretKey = EncryptionUtil.generateSecretKey();

        // Encrypt the data
        String encryptedData = EncryptionUtil.encrypt(item.getData(), secretKey);

        // Map to the encrypted entity
        EncryptedEntity encryptedEntity = new EncryptedEntity();
        encryptedEntity.setId(item.getId());
        encryptedEntity.setEncryptedData(encryptedData);
        encryptedEntity.setCiphertextkey(secretKey);  // Store the key

        return encryptedEntity;
    }
}
```

- `LegacyEntity`: The entity containing unencrypted data from the legacy system.
- `EncryptedEntity`: The entity that will be written to Cassandra with the encrypted data and `ciphertextkey`.

**Step 4.** **Write Encrypted Data to Cassandra**:
Once the data is encrypted, you can use an **ItemWriter** to store the encrypted data in Cassandra.

```java
@Bean
public ItemWriter<EncryptedEntity> cassandraWriter(MyEncryptedEntityRepository repository) {
    return new ItemWriter<EncryptedEntity>() {
        @Override
        public void write(List<? extends EncryptedEntity> items) throws Exception {
            repository.saveAll(items);
        }
    };
}
```

The `MyEncryptedEntityRepository` should be a **Spring Data Cassandra repository** to handle storing data in Cassandra.

```java
public interface MyEncryptedEntityRepository extends CassandraRepository<EncryptedEntity, String> {
}
```

**Step 5.** **Define the Encrypted Entity Class**:
The encrypted data and the encryption key should be stored in Cassandra. Define a class that represents this:

```java
@Table("encrypted_table")
public class EncryptedEntity {

    @PrimaryKey
    private String id;

    private String encryptedData;

    private String ciphertextkey;

    // Getters and Setters
}
```

**Step 6.** **Configure Spring Batch Job**:
Now, configure the Spring Batch job to perform the migration:

```java
@Bean
public Job migrationJob(JobBuilderFactory jobBuilderFactory, StepBuilderFactory stepBuilderFactory, 
                        ItemReader<LegacyEntity> reader, ItemProcessor<LegacyEntity, EncryptedEntity> processor,
                        ItemWriter<EncryptedEntity> writer) {

    Step step = stepBuilderFactory.get("encryptionStep")
            .<LegacyEntity, EncryptedEntity>chunk(100)
            .reader(reader)
            .processor(processor)
            .writer(writer)
            .build();

    return jobBuilderFactory.get("migrationJob")
            .start(step)
            .build();
}
```

> **Summary of the Process:**
>
> 1. **Read unencrypted data** from the legacy system using an **ItemReader**.
> 2. **Encrypt the data** using an **ItemProcessor** that leverages an encryption utility. Also, generate and store the `ciphertextkey`.
> 3. **Write the encrypted data** and the encryption key into Cassandra using an **ItemWriter**.

This process ensures that your data from the legacy system is securely encrypted before being stored in the Cassandra database.

