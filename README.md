# IoT_encryption
IoT AEAD Encryption Benchmark


🔐 IoT AEAD Encryption Benchmark
This project simulates IoT device data encryption using AEAD (Authenticated Encryption with Associated Data) schemes: AES-GCM and ChaCha20-Poly1305. It benchmarks the encryption performance across various IoT device types with different payload sizes.


📦 Features
Generates realistic IoT data for:
•Sensor devices (~10 KB)
• OTA update devices (~1 MB)
• Media stream devices (~20 MB)
• Encrypts payloads using AEAD schemes (AES-GCM, ChaCha20-Poly1305)
• Measures encryption latency and throughput
• Interactive CLI with optional benchmarking


### 🧰 Requirements
Make sure you have Python 3.7+ installed.
Install dependencies:
```bash
pip install cryptography tabulate
```

Then follow the prompts:
1. Choose a device type (Sensor / OTA / Media)
2. Choose an encryption scheme (AES-GCM or ChaCha20)
3. View encryption performance metrics
4. Optionally run a benchmark with multiple iterations


### 📊 Sample Output
```sql
=== Encryption Result ===
+------------------------+----------------------------------+
| Field                  | Value                            |
+------------------------+----------------------------------+
| Scheme                 | AES-GCM                          |
| Key (first 16B)        | 9a3f1d3b5f2a1c9b1f9d5e...         |
| Nonce                  | b3c75e2a04f1e9123a7e334b          |
| Ciphertext (first 32B) | 54ac1d7c...                       |
| Latency (ms)           | 5.23                             |
| Payload Size           | 10.00 KB                         |
| Metadata Size          | 108 bytes                        |
+------------------------+----------------------------------+
```


### 🧪 Benchmark Output Example
```sql
=== Benchmark Results ===
+-----------+--------------+------------------+----------+----------+-------------------+-------------+
| Scheme    | Device Type  | Avg Latency (ms) | Min (ms) | Max (ms) | Throughput (GB/s) | Iterations  |
+-----------+--------------+------------------+----------+----------+-------------------+-------------+
| AES-GCM   | Sensor       | 5.12             | 4.89     | 5.37     | 1.89              | 5           |
| ChaCha20  | Sensor       | 4.81             | 4.58     | 5.12     | 2.01              | 5           |
+-----------+--------------+------------------+----------+----------+-------------------+-------------+
```


🛡️ Encryption Details
• Uses 256-bit random keys and 96-bit nonces
• Associated data = IoT metadata (timestamp, device ID, location)
• Encrypts payload using cryptography library
• Ciphertext and performance metrics are printed to terminal


