# Client–Server Encryption Mechanism Implementation

This section outlines how homomorphic encryption (HE) is implemented in the CIFAR-10 FedAvg + CKKS setup, including where encryption context files are generated, how keys are distributed, and the NVFLARE components responsible for encryption, aggregation, and decryption.

---

## 1. HE Context Generation and Key Distribution

During the provisioning process (`nvflare provision -p secure_project.yml`), NVFLARE generates **TenSEAL encryption context files** for each participant. The file paths are:

- **Server context**:  
  ```
  secure_workspace/server/startup/server_context.tenseal
  ```
  This file contains only encryption parameters and the public key. No secret key is included, ensuring the server cannot decrypt model updates.

- **Client context**:  
  ```
  secure_workspace/site-1/startup/client_context.tenseal
  secure_workspace/site-2/startup/client_context.tenseal
  ```
  These files contain both the public key and the client’s secret key, enabling encryption of local model updates and decryption of aggregated updates.

The separation ensures that only clients can decrypt, while the server performs aggregation purely on ciphertext.

---

## 2. Encryption at the Client

At the client side, NVFLARE applies the `HEModelEncryptor` component before transmitting model updates:

- **Component**:  
  ```
  nvflare/app_opt/he/model_encryptor.py
  ```
- **Function**: Reads the `client_context.tenseal` file to initialize the TenSEAL context, encrypts the specified model layers, and packages the ciphertext for transmission.
- **File Path in Workspace**:  
  ```
  secure_workspace/site-1/startup/config/config_fed_client.json
  ```
  This configuration specifies the `HEModelEncryptor` in the `task_result_filters` list, ensuring encryption occurs automatically at the end of local training.

---

## 3. Aggregation at the Server

The server uses the `HEInTimeAccumulateWeightedAggregator` to perform secure aggregation:

- **Component**:  
  ```
  nvflare/app_opt/he/intime_accumulate_model_aggregator.py
  ```
- **Function**: Accepts encrypted model updates from clients, performs weighted averaging directly on ciphertext, and produces an aggregated encrypted model update without decryption.
- **File Path in Workspace**:  
  ```
  secure_workspace/server/startup/config/config_fed_server.json
  ```
  This configuration lists `HEInTimeAccumulateWeightedAggregator` in the `components` section, enabling encrypted aggregation in the training workflow.

---

## 4. Decryption at the Client

After aggregation, the encrypted global model is sent back to clients. Decryption is handled by the `HEModelDecryptor`:

- **Component**:  
  ```
  nvflare/app_opt/he/model_decryptor.py
  ```
- **Function**: Reads the `client_context.tenseal` file to initialize the TenSEAL context with the secret key, decrypts the aggregated ciphertext, and loads the resulting model weights into the local model state.
- **File Path in Workspace**:  
  ```
  secure_workspace/site-1/startup/config/config_fed_client.json
  ```
  This configuration includes `HEModelDecryptor` in the `task_data_filters` for incoming model updates.

---

## 5. Supporting Components

Additional modules ensure encrypted data is correctly serialized, deserialized, and packaged:

- **`HEModelSerializeFilter`**:  
  ```
  nvflare/app_opt/he/model_serialize_filter.py
  ```
  Serializes encrypted model tensors for transmission.

- **`HEModelShareableGenerator`**:  
  ```
  nvflare/app_opt/he/model_shareable_generator.py
  ```
  Packages the serialized encrypted model into NVFLARE’s `Shareable` format.

These are referenced in the server and client configuration files in:
```
secure_workspace/server/startup/config/config_fed_server.json
secure_workspace/site-1/startup/config/config_fed_client.json
```

---

## 6. Key Security

- The **server** has no access to any secret key; it cannot decrypt model updates at any point.
- **Clients** hold their own secret key, enabling them to decrypt only the aggregated global model after training rounds.
- This design ensures that individual model updates remain private and only the aggregated result is revealed to participating clients.
