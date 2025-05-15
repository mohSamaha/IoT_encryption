# IoT_encryption
IoT AEAD Encryption Benchmark


🔐 IoT AEAD Encryption Benchmark
This project simulates IoT device data encryption using AEAD (Authenticated Encryption with Associated Data) schemes: AES-GCM and ChaCha20-Poly1305. It benchmarks the encryption performance across various IoT device types with different payload sizes.


📦 Features
Generates realistic IoT data for:
- Sensor devices (~10 KB)
- OTA update devices (~1 MB)
- Media stream devices (~20 MB)
- Encrypts payloads using AEAD schemes (AES-GCM, ChaCha20-Poly1305)
- Measures encryption latency and throughput
- Interactive CLI with optional benchmarking


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
- Uses 256-bit random keys and 96-bit nonces
- Associated data = IoT metadata (timestamp, device ID, location)
- Encrypts payload using cryptography library
- Ciphertext and performance metrics are printed to terminal


### 🧪 Full Simulation Code
Below is the complete code for simulating AEAD encryption on IoT device data:
```python
# === IoT AEAD Encryption Benchmark ===

import os
import json
import time
from time import perf_counter as timer
from cryptography.hazmat.primitives.ciphers.aead import AESGCM, ChaCha20Poly1305
from tabulate import tabulate
import secrets
import string
import base64

# IoT Device Types with different payload sizes
def generate_iot_data(device_type):
    devices = {
        "sensor": ["temp_sensor_1", "humidity_sensor_2", "motion_sensor_3"],
        "ota_device": ["smart_lock_1", "thermostat_2", "light_controller_3"],
        "media_device": ["security_cam_1", "audio_recorder_2", "video_doorbell_3"]
    }

    if device_type == 1:
        payload = {
            "value": round(secrets.randbelow(100) + secrets.SystemRandom().random(), 2),
            "unit": secrets.choice(["°C", "%RH", "lux"]),
            "data": base64.b64encode(os.urandom(10 * 1024)).decode('utf-8')
        }
        device_id = secrets.choice(devices["sensor"])

    elif device_type == 2:
        payload = {
            "firmware_version": f"v2.{secrets.randbelow(50)}",
            "checksum": secrets.token_hex(16),
            "data": base64.b64encode(os.urandom(1024 * 1024)).decode('utf-8')
        }
        device_id = secrets.choice(devices["ota_device"])

    else:
        payload = {
            "stream_type": secrets.choice(["video", "audio"]),
            "timestamp": time.time(),
            "data": base64.b64encode(os.urandom(20 * 1024 * 1024)).decode('utf-8')
        }
        device_id = secrets.choice(devices["media_device"])

    metadata = {
        "device_id": device_id,
        "timestamp": time.strftime("%Y-%m-%dT%H:%M:%SZ", time.gmtime()),
        "location": f"Room-{secrets.choice(string.ascii_uppercase)}",
        "status": "active"
    }

    return payload, metadata, len(payload["data"])

def aead_operation(payload, metadata, scheme):
    key = os.urandom(32)
    nonce = os.urandom(12)
    payload_bytes = json.dumps(payload).encode()
    metadata_bytes = json.dumps(metadata).encode()

    if scheme == 1:
        cipher = AESGCM(key)
        start = timer()
        ciphertext = cipher.encrypt(nonce, payload_bytes, metadata_bytes)
        latency = (timer() - start) * 1000
    else:
        cipher = ChaCha20Poly1305(key)
        start = timer()
        ciphertext = cipher.encrypt(nonce, payload_bytes, metadata_bytes)
        latency = (timer() - start) * 1000

    return {
        "scheme": "AES-GCM" if scheme == 1 else "ChaCha20-Poly1305",
        "key": key.hex(),
        "nonce": nonce.hex(),
        "ciphertext": ciphertext[:64].hex() + "...",
        "latency_ms": round(latency, 2),
        "payload_size": len(payload_bytes),
        "metadata_size": len(metadata_bytes)
    }

def benchmark(device_type, scheme, iterations=5):
    results = []
    total_payload_size = 0

    for i in range(iterations):
        print(f"\rRunning benchmark {i + 1}/{iterations}...", end="", flush=True)
        payload, metadata, _ = generate_iot_data(device_type)
        result = aead_operation(payload, metadata, scheme)
        results.append(result)
        total_payload_size += result["payload_size"]
    print()

    total_time = sum(r["latency_ms"] for r in results) / 1000
    throughput = (total_payload_size / (1024 ** 3)) / total_time if total_time > 0 else 0

    return {
        "scheme": "AES-GCM" if scheme == 1 else "ChaCha20-Poly1305",
        "device_type": "Sensor" if device_type == 1 else "OTA Device" if device_type == 2 else "Media Device",
        "avg_latency_ms": round(sum(r["latency_ms"] for r in results) / iterations, 2),
        "min_latency_ms": round(min(r["latency_ms"] for r in results), 2),
        "max_latency_ms": round(max(r["latency_ms"] for r in results), 2),
        "throughput_gbs": round(throughput, 2),
        "iterations": iterations
    }

def display_result(result):
    size_mb = result["payload_size"] / (1024 * 1024)
    size_str = f"{size_mb:.2f} MB" if size_mb >= 1 else f"{result['payload_size'] / 1024:.2f} KB"
    display_data = {
        "Field": ["Scheme", "Key (first 16B)", "Nonce", "Ciphertext (first 32B)",
                  "Latency (ms)", "Payload Size", "Metadata Size"],
        "Value": [
            result["scheme"],
            result["key"][:32] + "...",
            result["nonce"],
            result["ciphertext"],
            result["latency_ms"],
            size_str,
            f"{result['metadata_size']} bytes"
        ]
    }
    print("\n=== Encryption Result ===")
    print(tabulate(display_data, headers="keys", tablefmt="grid"))

def display_benchmark(results):
    print("\n=== Benchmark Results ===")
    table_data = []
    for res in results:
        table_data.append([
            res["scheme"],
            res["device_type"],
            res["avg_latency_ms"],
            res["min_latency_ms"],
            res["max_latency_ms"],
            res["throughput_gbs"],
            res["iterations"]
        ])
    print(tabulate(table_data,
                   headers=["Scheme", "Device Type", "Avg Latency (ms)", "Min (ms)", "Max (ms)", "Throughput (GB/s)",
                            "Iterations"],
                   tablefmt="grid"))

def main():
    print("=== IoT Security Simulation ===")
    print("\nSelect IoT Device Type:")
    print("1. Sensor Device (Small payload ~10KB)")
    print("2. OTA Update Device (Medium payload ~1MB)")
    print("3. Media Device (Large payload ~20MB)")
    device_type = int(input("Choose device type (1/2/3): "))

    print("\nSelect Encryption Scheme:")
    print("1. AES-GCM")
    print("2. ChaCha20-Poly1305")
    scheme = int(input("Choose encryption scheme (1/2): "))

    payload, metadata, _ = generate_iot_data(device_type)

    print("\n=== Generated IoT Data ===")
    print(f"Device Type: {'Sensor' if device_type == 1 else 'OTA Device' if device_type == 2 else 'Media Device'}")
    print(f"Payload sample: {json.dumps({k: v for k, v in payload.items() if k != 'data'}, indent=2)}")
    print(f"Metadata: {json.dumps(metadata, indent=2)}")

    result = aead_operation(payload, metadata, scheme)
    display_result(result)

    benchmark_choice = input("\nWould you like to run benchmarks? (y/n): ").lower()
    if benchmark_choice == 'y':
        iterations = int(input("How many iterations? (default 5): ") or 5)
        print("\nRunning benchmarks...")
        benchmark_results = [
            benchmark(device_type, 1, iterations),
            benchmark(device_type, 2, iterations)
        ]
        display_benchmark(benchmark_results)

if __name__ == "__main__":
    main()
```
