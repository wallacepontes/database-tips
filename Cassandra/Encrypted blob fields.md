# Encrypted blob fields

## Table of contents
- [Encrypted blob fields](#encrypted-blob-fields)
  - [Table of contents](#table-of-contents)
  - [ciphertextkey](#ciphertextkey)
    - [**1. Understanding `ciphertextkey`**](#1-understanding-ciphertextkey)
    - [**2. Encryption/Decryption Process**](#2-encryptiondecryption-process)
      - [**Encryption**](#encryption)
      - [**Decryption**](#decryption)
    - [**3. Key Components**](#3-key-components)
      - [**a. Key Encryption Algorithm**](#a-key-encryption-algorithm)
      - [**b. Data Encryption Algorithm**](#b-data-encryption-algorithm)
      - [**c. Key Storage**](#c-key-storage)
      - [**d. Initialization Vector (IV)**](#d-initialization-vector-iv)
    - [**4. Inspecting the Table and Fields**](#4-inspecting-the-table-and-fields)
      - [**a. Blob Fields**](#a-blob-fields)
      - [**b. Analyze the `ciphertextkey`**](#b-analyze-the-ciphertextkey)
      - [**c. Decrypt the Blob Fields**](#c-decrypt-the-blob-fields)
    - [**5. Investigating Encryption Details**](#5-investigating-encryption-details)
      - [**a. Encryption Configuration**](#a-encryption-configuration)
      - [**b. Key Management System**](#b-key-management-system)
      - [**c. Logs**](#c-logs)
    - [**6. Next Steps**](#6-next-steps)
  - [Encrypted data](#encrypted-data)
    - [Key Points in the Data Structure](#key-points-in-the-data-structure)
    - [Encryption and Decryption Workflow](#encryption-and-decryption-workflow)
      - [**Encryption**](#encryption-1)
      - [**Decryption**](#decryption-1)
    - [Algorithms Likely in Use](#algorithms-likely-in-use)
    - [Observations for Secure Data Handling](#observations-for-secure-data-handling)
    - [Next Steps for Analysis](#next-steps-for-analysis)
  - [Decrypt a specific field step by step](#decrypt-a-specific-field-step-by-step)
    - [**Step 1: Verify the Vault/KMS Integration**](#step-1-verify-the-vaultkms-integration)
    - [**Step 2: Set Up a Decryption Environment**](#step-2-set-up-a-decryption-environment)
    - [**Step 3: Decrypt `ciphertextkey` to Get the DEK**](#step-3-decrypt-ciphertextkey-to-get-the-dek)
    - [**Step 4: Decrypt Blob Fields**](#step-4-decrypt-blob-fields)
    - [**Step 5: Automate the Process**](#step-5-automate-the-process)
    - [**Step 6: Compliance and Security**](#step-6-compliance-and-security)
  - [Using Azure](#using-azure)
    - [**Decryption Workflow Using Azure and Your Tech Stack**](#decryption-workflow-using-azure-and-your-tech-stack)
      - [**Step 1: Verify Azure Key Vault Integration**](#step-1-verify-azure-key-vault-integration)
      - [**Step 2: Retrieve and Decrypt the Key Encryption Key (KEK)**](#step-2-retrieve-and-decrypt-the-key-encryption-key-kek)
      - [**Step 3: Decrypt the Encrypted Blob**](#step-3-decrypt-the-encrypted-blob)
      - [**Step 4: Automate Within Kubernetes**](#step-4-automate-within-kubernetes)
      - [**Step 5: Secure Authentication and Authorization**](#step-5-secure-authentication-and-authorization)
      - [**Step 6: Verify the Process**](#step-6-verify-the-process)

## ciphertextkey 

The presence of a field named **`ciphertextkey`** in the same table as the encrypted blob fields suggests a **key-wrapping mechanism** or **key encryption scheme**. Here's an explanation of how it works and its role in the encryption/decryption process:

---

### **1. Understanding `ciphertextkey`**
The **`ciphertextkey`** is typically used to securely store the encryption key (or a reference to it) for the associated blob fields. Here's how it fits into the encryption/decryption process:

1. **Key-Wrapping**:
   - The actual **data encryption key (DEK)** used to encrypt the blob field is itself encrypted using a **key encryption key (KEK)**.
   - The resulting encrypted key is stored in the `ciphertextkey` field.

2. **Key Management**:
   - The KEK is managed separately, often stored in a keystore, hardware security module (HSM), or external key management system.
   - This ensures that even if the database is compromised, the encrypted data cannot be decrypted without access to the KEK.

---

### **2. Encryption/Decryption Process**
#### **Encryption**
1. A **data encryption key (DEK)** is generated to encrypt the data (blob field) using an algorithm like AES.
2. The DEK is encrypted using the **key encryption key (KEK)**.
3. The encrypted data is stored in the blob field, and the `ciphertextkey` field stores the encrypted DEK.

#### **Decryption**
1. Retrieve the `ciphertextkey` field (encrypted DEK) and the blob field (encrypted data).
2. Use the **KEK** to decrypt the `ciphertextkey` and retrieve the original DEK.
3. Use the DEK to decrypt the blob field and retrieve the plaintext data.

---

### **3. Key Components**
#### **a. Key Encryption Algorithm**
- The algorithm used to encrypt the DEK (for the `ciphertextkey`) is usually RSA or AES Key-Wrapping.
- Example: RSA-OAEP, AES-KW.

#### **b. Data Encryption Algorithm**
- The algorithm used to encrypt the actual data (blob fields) is typically AES with a mode like GCM or CBC.

#### **c. Key Storage**
- The KEK may be stored in:
  - A secure keystore file (e.g., `/etc/secrets/encryption/cql/keystore.jks`).
  - An external key management system (e.g., AWS KMS, Azure Key Vault, HashiCorp Vault).

#### **d. Initialization Vector (IV)**
- Encryption often involves an **IV** to ensure randomness. It may be stored alongside the encrypted data or derived during the encryption process.

---

### **4. Inspecting the Table and Fields**
#### **a. Blob Fields**
- Query the table to retrieve sample encrypted data:
  ```cql
  SELECT encrypted_blob1, encrypted_blob2, ciphertextkey FROM keyspace.table LIMIT 1;
  ```

#### **b. Analyze the `ciphertextkey`**
- If the KEK and algorithm are known, you can try decrypting the `ciphertextkey`:
  ```bash
  echo "<ciphertextkey>" | openssl rsautl -decrypt -inkey <private_key.pem>
  ```
- This will return the DEK.

#### **c. Decrypt the Blob Fields**
- Use the DEK to decrypt the blob fields:
  ```bash
  echo "<encrypted_blob>" | openssl enc -aes-256-cbc -d -K <DEK> -iv <IV>
  ```

---

### **5. Investigating Encryption Details**
#### **a. Encryption Configuration**
- Look for encryption settings in the application code or Cassandra configuration.
- Check for any key management tools integrated with the system.

#### **b. Key Management System**
- If a key management system is in use, it may handle the KEK and DEK lifecycle. Look for integration with systems like AWS KMS, Azure Key Vault, or an HSM.

#### **c. Logs**
- Check Cassandra logs or application logs for references to encryption, key generation, or key management.

---

### **6. Next Steps**
Would you like assistance with:
1. Inspecting the `ciphertextkey` to identify the encryption scheme?
2. Analyzing the blob field data structure for decryption?
3. Understanding the key management system in use?

Let me know your focus, and we can proceed step by step!

## Encrypted data

This table appears to store encrypted data and associated metadata, with a focus on securely managing sensitive information. Here's a detailed analysis of how the encryption and `ciphertextkey` field relate to the rest of the data:

---

### Key Points in the Data Structure

1. **Blob Fields with Encrypted Data**
   - Fields such as `aliveduring` and others that are stored in hexadecimal format (`0xe40fad...`) indicate encrypted values.
   - These blobs are likely encrypted using a symmetric encryption algorithm (e.g., AES), which is efficient for bulk data.

2. **`ciphertextkey`**
   - The **`ciphertextkey`** field (`vault:v1:yzWFCUGJNK...`) likely stores the **encrypted Data Encryption Key (DEK)** or a reference to it.
   - The prefix `vault:v1` suggests integration with a **key management system (KMS)** or a similar vault mechanism. It might indicate:
     - Versioning of keys (e.g., `v1` as the version of the key or vault format).
     - Usage of a KMS like HashiCorp Vault, AWS KMS, or Azure Key Vault.

3. **Hexadecimal Fields**
   - Other fields, such as those in `contactmedia`, appear to hold multiple encrypted values, each associated with a unique identifier (e.g., `05b513e0...`), hinting at per-entry encryption.

4. **Field Relationships**
   - **Encrypted Data**: Fields like `aliveduring` are encrypted using a DEK.
   - **Key Management**: The `ciphertextkey` contains the encrypted DEK, which is needed to decrypt the data.
   - **Metadata**: Other fields (e.g., `humanreadableid`, `partyroles`) might provide context or indexing for retrieving and processing data.

---

### Encryption and Decryption Workflow

#### **Encryption**
1. **Data Encryption**:
   - Each field (e.g., `aliveduring`) is encrypted using a DEK, typically generated per record or field.
   - The encrypted value is stored in the respective blob field.

2. **Key Management**:
   - The DEK is encrypted with a Key Encryption Key (KEK), which is managed securely in a vault.
   - The resulting `ciphertextkey` is stored in the record.

#### **Decryption**
1. Retrieve the record, including the encrypted field (e.g., `aliveduring`) and the `ciphertextkey`.
2. Use the vault or KMS to decrypt the `ciphertextkey` and retrieve the DEK.
3. Use the DEK to decrypt the blob field and access the plaintext data.

---

### Algorithms Likely in Use
1. **For Blob Fields**:
   - Symmetric encryption like AES-GCM or AES-CBC, often 256-bit.
2. **For `ciphertextkey`**:
   - Asymmetric encryption like RSA or Elliptic Curve Cryptography (ECC) for encrypting the DEK.
   - Alternatively, AES Key Wrapping (if KEK is symmetric).

---

### Observations for Secure Data Handling
1. **Vault Integration**:
   - The prefix `vault:v1` strongly suggests the use of an external key management service, ensuring secure key storage and lifecycle management.
   - Check integration configurations in the application.

2. **Field Versioning**:
   - Considerations for versioning keys (e.g., `v1`) can ensure smooth key rotation without impacting older data.

3. **Key Hierarchy**:
   - A clear separation between KEK (long-term storage) and DEK (short-lived, record-specific keys) is present, adhering to encryption best practices.

---

### Next Steps for Analysis
1. **Key Vault Details**:
   - Identify the KMS or vault (e.g., HashiCorp, AWS KMS) managing the `ciphertextkey`. Review how KEKs are stored and accessed.

2. **Algorithm Confirmation**:
   - Confirm the encryption algorithms for the blob fields and `ciphertextkey` in application logs or documentation.

3. **Sample Decryption**:
   - If access to the KEK and vault is available, test decryption for a sample record:
     - Retrieve the DEK by decrypting `ciphertextkey`.
     - Use the DEK to decrypt the blob field.

4. **Compliance Checks**:
   - Verify encryption practices align with compliance standards (e.g., GDPR, PCI-DSS).

Would you like guidance on how to inspect vault configurations or decrypt a specific field?

## Decrypt a specific field step by step

Here’s how you can inspect vault configurations and decrypt a specific field step by step:

---

### **Step 1: Verify the Vault/KMS Integration**
1. **Identify the Vault**:
   - Look at the prefix of `ciphertextkey` (`vault:v1`). It suggests a vault or KMS integration (e.g., HashiCorp Vault, AWS KMS, Azure Key Vault).
   - Confirm the vault being used in your system’s configuration files or documentation.

2. **Check Permissions**:
   - Ensure you have access to the vault or KMS to manage keys. You might need roles like `KeyAdmin` or `CryptoAdmin`.

3. **Retrieve KEK Configuration**:
   - Look for the key details (e.g., key name, ID, or alias) in the vault that corresponds to `ciphertextkey`.

---

### **Step 2: Set Up a Decryption Environment**
You’ll need:
- **Access to the Vault/KMS**: Ensure CLI tools or SDKs for the vault are installed (e.g., AWS CLI, HashiCorp Vault CLI).
- **Application Credentials**: Use IAM roles or API tokens with the necessary permissions to decrypt data.
- **Encryption Library**: Ensure you have the encryption library (e.g., OpenSSL, Java's Cipher API) compatible with the blob encryption algorithm.

---

### **Step 3: Decrypt `ciphertextkey` to Get the DEK**
1. **Extract `ciphertextkey`**:
   - Example value: `vault:v1:yzWFCUGJNKzr...`.
   - Remove any prefixes (`vault:v1:`) to isolate the encrypted DEK.

2. **Use the Vault CLI or SDK**:
   - **HashiCorp Vault**:
     ```bash
     vault kv get -field=ciphertextkey <path/to/secret>
     ```
   - **AWS KMS**:
     ```bash
     aws kms decrypt \
       --ciphertext-blob fileb://ciphertextkey.encrypted \
       --output text \
       --query Plaintext | base64 --decode > dek.key
     ```
   - **Azure Key Vault**:
     ```bash
     az keyvault key decrypt \
       --name <key-name> \
       --vault-name <vault-name> \
       --value <ciphertextkey> \
       --query result
     ```

3. **Store the DEK**:
   - Save the plaintext DEK securely for the next decryption step (e.g., in-memory only).

---

### **Step 4: Decrypt Blob Fields**
1. **Prepare the Encrypted Blob**:
   - Extract the hexadecimal value of the encrypted blob (e.g., `0xe40fad09...`).

2. **Use a Compatible Library**:
   - **Python Example (AES-GCM)**:
     ```python
     from Crypto.Cipher import AES
     from binascii import unhexlify

     # DEK and Encrypted Blob
     dek = b'<your DEK>'
     encrypted_blob = unhexlify('e40fad09ef26...')

     # Split nonce and ciphertext
     nonce = encrypted_blob[:12]  # Adjust according to your algorithm
     ciphertext = encrypted_blob[12:-16]  # Ciphertext excluding the tag
     tag = encrypted_blob[-16:]  # Authentication tag

     # Decrypt
     cipher = AES.new(dek, AES.MODE_GCM, nonce=nonce)
     plaintext = cipher.decrypt_and_verify(ciphertext, tag)
     print("Decrypted Data:", plaintext.decode())
     ```

   - **Java Example (AES-CBC)**:
     ```java
     import javax.crypto.Cipher;
     import javax.crypto.spec.SecretKeySpec;
     import javax.crypto.spec.IvParameterSpec;
     import java.util.Base64;

     // DEK and Encrypted Blob
     byte[] dek = Base64.getDecoder().decode("<your DEK>");
     byte[] encryptedBlob = javax.xml.bind.DatatypeConverter.parseHexBinary("e40fad09ef26...");

     // Split IV and Ciphertext
     byte[] iv = Arrays.copyOfRange(encryptedBlob, 0, 16); // 16-byte IV
     byte[] ciphertext = Arrays.copyOfRange(encryptedBlob, 16, encryptedBlob.length);

     // Decrypt
     Cipher cipher = Cipher.getInstance("AES/CBC/PKCS5Padding");
     cipher.init(Cipher.DECRYPT_MODE, new SecretKeySpec(dek, "AES"), new IvParameterSpec(iv));
     byte[] plaintext = cipher.doFinal(ciphertext);
     System.out.println("Decrypted Data: " + new String(plaintext));
     ```

---

### **Step 5: Automate the Process**
1. **Build a Script**:
   - Combine the steps to retrieve the DEK and decrypt the blob field in a single script.
   - Ensure the script uses secure memory and cleans up sensitive data after execution.

2. **Audit and Logs**:
   - Log decryption requests (but not sensitive keys or plaintext data) for traceability.

3. **Testing**:
   - Use a test record with known plaintext data to validate the decryption process.

---

### **Step 6: Compliance and Security**
- **Secure Storage**: Do not store the DEK or decrypted data outside the script’s memory.
- **Access Controls**: Limit decryption privileges to authorized personnel or services.
- **Audit Trails**: Maintain logs for vault access and decryption events for compliance.

---

Would you like help implementing any of these steps in your environment?

## Using Azure

Thanks for the additional context! With Azure as the cloud provider and your tech stack including **Confluence**, **Kubernetes**, **GitLab**, and **Keycloak**, here's a tailored approach for decrypting `ciphertextkey` and managing the decryption process securely:

---

### **Decryption Workflow Using Azure and Your Tech Stack**

#### **Step 1: Verify Azure Key Vault Integration**
- Ensure the `ciphertextkey` is encrypted with Azure Key Vault.
  - Check the vault configuration in your Kubernetes environment (`ConfigMap` or `Secrets`).
  - Look for references to the Azure Key Vault in Helm charts or deployment manifests.

#### **Step 2: Retrieve and Decrypt the Key Encryption Key (KEK)**
1. **Authenticate to Azure Key Vault**:
   - Use a service principal, managed identity, or a personal account with permissions to decrypt.
   - Validate your access using the Azure CLI:
     ```bash
     az login
     az keyvault key show --name <key-name> --vault-name <vault-name>
     ```

2. **Decrypt the Ciphertextkey Using Azure CLI**:
   - Extract the encrypted DEK from your application's storage (likely in a Kubernetes secret or a GitLab CI/CD variable).
   - Decrypt using Azure Key Vault:
     ```bash
     az keyvault key decrypt \
       --name <key-name> \
       --vault-name <vault-name> \
       --value <ciphertextkey> \
       --query result
     ```

3. **Save the DEK Temporarily**:
   - Store it only in memory for the duration of the blob decryption process.

---

#### **Step 3: Decrypt the Encrypted Blob**
With the plaintext DEK obtained:
1. **Extract the Encrypted Blob**:
   - Likely stored in Azure Blob Storage, Kubernetes Secrets, or your application database.
   - If the blob is stored in a database (like Postgres or CosmosDB), query and retrieve it.

2. **Choose a Decryption Tool/Language**:
   - Since Azure Key Vault is integrated with most SDKs (Java, Python, etc.), you can decrypt in the language of your choice.
   - Decrypt using AES (common encryption method in Azure):
     - **Python AES Decryption (GCM)**:
       ```python
       from Crypto.Cipher import AES
       from binascii import unhexlify

       # DEK (from Azure Key Vault)
       dek = b'<your DEK>'

       # Encrypted blob
       encrypted_blob = unhexlify('e40fad09...')

       # Parse nonce, ciphertext, and tag
       nonce = encrypted_blob[:12]
       ciphertext = encrypted_blob[12:-16]
       tag = encrypted_blob[-16:]

       # Decrypt
       cipher = AES.new(dek, AES.MODE_GCM, nonce=nonce)
       plaintext = cipher.decrypt_and_verify(ciphertext, tag)
       print("Decrypted Data:", plaintext.decode())
       ```

---

#### **Step 4: Automate Within Kubernetes**
1. **Store Secrets Securely**:
   - Use **Azure Key Vault with Kubernetes Secrets** integration to manage sensitive keys.
   - Example manifest for syncing secrets:
     ```yaml
     apiVersion: spv.no/v1
     kind: AzureKeyVaultSecret
     metadata:
       name: dek-secret
     spec:
       vault:
         name: <vault-name>
       output:
         secret:
           name: dek-secret-k8s
     ```

2. **Access Secrets in Your Pods**:
   - Inject the secret into your application pods as environment variables or files:
     ```yaml
     env:
       - name: DEK_SECRET
         valueFrom:
           secretKeyRef:
             name: dek-secret-k8s
             key: value
     ```

3. **Trigger Decryption in GitLab CI/CD**:
   - Use a pipeline stage to decrypt data if the blob is required for deployments or testing:
     ```yaml
     decrypt_blob:
       script:
         - az login --service-principal -u $AZURE_CLIENT_ID -p $AZURE_CLIENT_SECRET --tenant $AZURE_TENANT_ID
         - az keyvault key decrypt --name <key-name> --vault-name <vault-name> --value $ENCRYPTED_BLOB
       only:
         - master
     ```

---

#### **Step 5: Secure Authentication and Authorization**
1. **Keycloak for RBAC**:
   - Use Keycloak to manage access to decryption services, limiting access to authorized roles.
   - Create a Keycloak client and roles for services or users performing decryption.

2. **Audit Trails**:
   - Enable Azure Key Vault logging to track decryption operations:
     ```bash
     az monitor diagnostic-settings create \
       --resource /subscriptions/<sub-id>/resourceGroups/<rg-name>/providers/Microsoft.KeyVault/vaults/<vault-name> \
       --name "enable-logging" \
       --logs '[{"category": "AuditEvent", "enabled": true}]'
     ```

---

#### **Step 6: Verify the Process**
- Test the decryption pipeline using a dummy encrypted blob and validate against known plaintext.
- Automate regression tests to ensure no critical paths break.

---

Would you like assistance setting up a specific component of this workflow, such as Azure Key Vault, Kubernetes secrets, or the decryption code?