# Avro PL/SQL

## Topic of contents
- [Avro PL/SQL](#avro-plsql)
  - [Topic of contents](#topic-of-contents)
  - [Avro encoding and decoding](#avro-encoding-and-decoding)
  - [Manage those dependencies in Oracle](#manage-those-dependencies-in-oracle)

## Avro encoding and decoding
To make the Avro encoding and decoding more generic, where you can provide the schema as a JSON string along with either JSON data to encode or binary data to decode, you can modify the Java stored procedures to accept these inputs. Here's how you can do it:

1. Generic Encode Function:

```sql
CREATE OR REPLACE AND RESOLVE JAVA SOURCE NAMED "GenericAvroEncoder" AS
import org.apache.avro.Schema;
import org.apache.avro.generic.GenericData;
import org.apache.avro.generic.GenericRecord;
import org.apache.avro.io.DatumWriter;
import org.apache.avro.io.EncoderFactory;
import org.apache.avro.io.BinaryEncoder;
import org.json.JSONObject;
import java.io.ByteArrayOutputStream;
import java.nio.ByteBuffer;

public class GenericAvroEncoder {
    public static ByteBuffer encodeAvro(String schemaJson, String jsonData) throws Exception {
        Schema schema = new Schema.Parser().parse(schemaJson);
        GenericRecord record = new GenericData.Record(schema);
        JSONObject json = new JSONObject(jsonData);

        // Populate the record with JSON data
        for (Schema.Field field : schema.getFields()) {
            String fieldName = field.name();
            Object value = json.has(fieldName) ? json.get(fieldName) : null;
            
            // Handle null values or type conversion if needed
            if (value != null && field.schema().getType() == Schema.Type.UNION) {
                Schema actualType = field.schema().getTypes().get(1); // Assuming null is first, actual type second
                if (actualType.getType() == Schema.Type.LONG && value instanceof Integer) {
                    value = ((Integer) value).longValue();
                }
            }

            record.put(fieldName, value);
        }

        ByteArrayOutputStream out = new ByteArrayOutputStream();
        BinaryEncoder encoder = EncoderFactory.get().binaryEncoder(out, null);
        DatumWriter<GenericRecord> writer = new GenericData().createDatumWriter(schema);
        writer.write(record, encoder);
        encoder.flush();
        out.close();

        return ByteBuffer.wrap(out.toByteArray());
    }
}
/
```
2. Generic Decode Function:
```sql
CREATE OR REPLACE AND RESOLVE JAVA SOURCE NAMED "GenericAvroDecoder" AS
import org.apache.avro.Schema;
import org.apache.avro.generic.GenericData;
import org.apache.avro.generic.GenericRecord;
import org.apache.avro.io.DatumReader;
import org.apache.avro.io.DecoderFactory;
import org.apache.avro.io.BinaryDecoder;
import java.io.ByteArrayInputStream;
import java.nio.ByteBuffer;
import org.json.JSONObject;

public class GenericAvroDecoder {
    public static String decodeAvro(String schemaJson, ByteBuffer buffer) throws Exception {
        Schema schema = new Schema.Parser().parse(schemaJson);
        DatumReader<GenericRecord> reader = new GenericData().createDatumReader(schema);
        BinaryDecoder decoder = DecoderFactory.get().binaryDecoder(buffer.array(), null);
        GenericRecord record = reader.read(null, decoder);

        JSONObject json = new JSONObject();
        for (Schema.Field field : schema.getFields()) {
            String fieldName = field.name();
            json.put(fieldName, record.get(fieldName));
        }

        return json.toString();
    }
}
/
```
3. PL/SQL Wrapper Functions:
- Encoding:
```sql
CREATE OR REPLACE FUNCTION EncodeGenericAvro(schemaJson IN VARCHAR2, jsonData IN VARCHAR2) 
RETURN RAW 
AS LANGUAGE JAVA 
NAME 'GenericAvroEncoder.encodeAvro(java.lang.String, java.lang.String) return java.nio.ByteBuffer';
/
```
- Decoding:
```sql
CREATE OR REPLACE FUNCTION DecodeGenericAvro(schemaJson IN VARCHAR2, binaryData IN RAW) 
RETURN VARCHAR2 
AS LANGUAGE JAVA 
NAME 'GenericAvroDecoder.decodeAvro(java.lang.String, java.nio.ByteBuffer) return java.lang.String';
/
```
**Usage:**
- Encoding:
```sql
SELECT EncodeGenericAvro('{"type":"record","name":"MyRecord","fields":[{"name":"field1","type":["null","string"]},{"name":"field2","type":["null","long"]}]}', '{"field1":"value1","field2":123}') FROM dual;
```
- Decoding:
```sql
SELECT DecodeGenericAvro('{"type":"record","name":"MyRecord","fields":[{"name":"field1","type":["null","string"]},{"name":"field2","type":["null","long"]}]}', HEXTORAW('YourHexBinaryDataHere')) FROM dual;
```

**Notes:**
- You'll need to handle JSON parsing and type conversion carefully, especially for complex types or when dealing with unions in Avro schemas.
- The org.json library is used here for simplicity; ensure it's available in your Oracle JVM or replace with another JSON parsing library if necessary.
- Error handling should be more comprehensive in a production environment, catching and reporting specific exceptions that might occur during schema parsing or data conversion.

## Manage those dependencies in Oracle
That's great to hear! Now that you've got a better understanding of the dependencies by working in a Java project environment like VSCode, you can more effectively manage those dependencies in Oracle. Here's a summary of the steps to ensure you can run your Avro-related PL/SQL:

**Steps to Load Dependencies in Oracle:**
1. Identify Required JARs:
    - From your VSCode project, you've identified the JAR files needed for Avro (avro-1.12.0.jar), JSON handling (json-20230227.jar for org.json.JSONObject), and any other dependencies like jackson-databind if used with Avro.
2. Create a Directory for JAR Files:
    - Oracle needs to know where the JAR files are located on the database server:
```sql
CREATE OR REPLACE DIRECTORY jar_dir AS '/path/to/your/jar_files';
```
Ensure this directory is accessible and has the correct permissions.

3. Load JAR Files into Oracle:
    - For each JAR file:
```sql
CREATE OR REPLACE JAVA RESOURCE "avro-1.12.0.jar" USING BFILE(jar_dir, 'avro-1.12.0.jar');
CREATE OR REPLACE JAVA RESOURCE "json-20230227.jar" USING BFILE(jar_dir, 'json-20230227.jar');
-- Repeat for other JARs
```
4. Resolve Dependencies:
    - After loading, resolve and grant permissions:
```sql
CALL dbms_java.loadjava('-genmissing -resolve -s -grant public', 'avro-1.12.0.jar');
CALL dbms_java.loadjava('-genmissing -resolve -s -grant public', 'json-20230227.jar');
-- Repeat for other JARs
```
The `-genmissing` option generates missing classes, `-resolve` resolves dependencies, `-s` specifies schema, and `-grant public` grants execute privileges to PUBLIC.

5. Test Your Java Stored Procedures:
    - After loading the dependencies, try to compile and run your Java stored procedures:
```sql
CREATE OR REPLACE AND RESOLVE JAVA SOURCE NAMED "GenericAvroEncoder" AS
-- Your Java code here
/

CREATE OR REPLACE FUNCTION EncodeGenericAvro(
  schemaJson IN VARCHAR2,
  jsonData IN VARCHAR2 DEFAULT '{}'
) 
RETURN RAW 
AS LANGUAGE JAVA 
NAME 'GenericAvroEncoder.encodeAvro(java.lang.String, java.lang.String) return java.nio.ByteBuffer';
/
```
6. Check for Errors:
    - Use dbms_java.set_output to see any Java output or errors:
```sql
EXECUTE DBMS_JAVA.set_output(1000000);
```
Then execute your functions or procedures to test.

7. Debugging:
    - If you encounter issues, check the Oracle alert log or use dbms_java.set_output for more detailed error messages. You might need to adjust class paths or permissions.

**Additional Tips:**
- Permissions: Ensure you have the necessary permissions (CREATE JAVA, CREATE JAVA RESOURCE, etc.) to perform these operations.
- Java Version: Verify the Java version in Oracle is compatible with your JARs.
- Class Path Issues: If you run into class path problems, consider explicitly setting the class path using dbms_java.set_class_path.

By following these steps, you should be able to successfully integrate your Avro and JSON processing into Oracle's PL/SQL environment.