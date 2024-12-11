# Hexadecimal Encode and Decode in Oracle using PLSQL and Java


## Decoding hexadecimal to JSON

Here's how you can implement the reverse process of decoding hexadecimal to JSON in Oracle using PL/SQL and Java stored procedures:

1. **Java Code for Hex to JSON:**

This Java class will convert a hexadecimal string back into a JSON string:

```sql
CREATE OR REPLACE AND RESOLVE JAVA SOURCE NAMED "HexaDecoder" AS
import java.nio.charset.StandardCharsets;
import java.util.Arrays;

public class HexaDecoder {
    public static String hexToJSON(String hexString) {
        // Remove "0x" if present
        if (hexString.startsWith("0x")) {
            hexString = hexString.substring(2);
        }
        
        // Ensure the hex string length is even
        if (hexString.length() % 2 != 0) {
            throw new IllegalArgumentException("Hex string must have even number of characters");
        }
        
        // Convert hex to byte array
        byte[] bytes = new byte[hexString.length() / 2];
        for (int i = 0; i < bytes.length; i++) {
            int index = i * 2;
            int j = Integer.parseInt(hexString.substring(index, index + 2), 16);
            bytes[i] = (byte) j;
        }
        
        // Convert byte array to String
        return new String(bytes, StandardCharsets.UTF_8);
    }
}
/
```

2. **PL/SQL Wrapper for HexaDecoder:**

This PL/SQL function will call the Java method to decode hexadecimal to JSON:

```sql
CREATE OR REPLACE FUNCTION ApplyHexaDecode(p_hex VARCHAR2) RETURN VARCHAR2
AS LANGUAGE JAVA 
NAME 'HexaDecoder.hexToJSON(java.lang.String) return java.lang.String';
/
```

**Usage Example:**

You can now use this function to decode hexadecimal back to JSON:

```sql
SELECT ApplyHexaDecode('7B226E616D65223A2274657374222C2276616C7565223A2274657374227D') FROM dual;
```
This should return: `{"name":"test","value":"test"}`

### Notes: 
- Hexadecimal format: Ensure the hexadecimal string is in the correct format. The example assumes no spaces or other characters between hex pairs.
- Character Set: The example uses UTF-8, which is generally the default for JSON. If you're working with a different character set, you might need to adjust this part.
- Error Handling: The Java code includes basic error handling for an odd number of hex characters, but in production, you'd want more comprehensive error handling, possibly returning null or an error message to PL/SQL for further processing.
- Performance: For very large hex strings, consider stream processing or chunking to handle memory constraints.

This setup allows you to decode hexadecimal strings representing JSON back into their string form within Oracle, using the combination of Java stored procedures and PL/SQL.