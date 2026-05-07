# Mesh Pay: Offline UPI Mesh Settlement Backend

![Java](https://img.shields.io/badge/Java-17%2B-ED8B00?style=for-the-badge&logo=java&logoColor=white)
![Spring Boot](https://img.shields.io/badge/Spring%20Boot-3.3.5-6DB33F?style=for-the-badge&logo=spring-boot&logoColor=white)
![License](https://img.shields.io/badge/License-MIT-blue?style=for-the-badge)

**Mesh Pay** is a high-performance Spring Boot backend designed to facilitate offline UPI payments through a peer-to-peer mesh network. This project demonstrates how transactions can be securely routed through untrusted intermediaries and settled once connectivity is restored.

---

## 🚀 Overview

In environments with zero connectivity (e.g., basements, remote areas), Mesh Pay enables users to initiate encrypted payment instructions. These instructions "hop" between nearby devices using a Bluetooth-style mesh until they reach a "bridge" node with internet access, which then uploads them to this backend for final settlement.

---

## 🏗️ Core Architecture

Mesh Pay solves the primary challenges of offline decentralized payments through three key pillars:

### 1. Security via Hybrid Cryptography
To protect against untrusted intermediaries, we use a **Hybrid RSA + AES-GCM** encryption scheme. 
- **Encryption:** The sender phone encrypts the payload with the server's RSA Public Key.
- **Integrity:** AES-GCM ensures that if any intermediate node attempts to tamper with the ciphertext, the GCM tag will fail verification on the server, and the transaction will be rejected.

### 2. Idempotency & The Duplicate Storm
In a mesh network, the same transaction packet might reach the backend multiple times through different bridge nodes.
- **Solution:** We use an atomic `putIfAbsent` operation (simulated via `ConcurrentHashMap` in this demo, typically Redis in production) on the **SHA-256 hash of the ciphertext**. This ensures that exactly one delivery settles the payment, and all subsequent duplicates are dropped instantly.

### 3. Replay Protection
To prevent an attacker from capturing and re-sending a valid transaction later:
- **Nonces:** Every payment instruction includes a unique UUID nonce.
- **Freshness Window:** The backend enforces a 24-hour expiration window on the internal `signedAt` timestamp of the encrypted payload.

---

## 🛠️ Tech Stack

- **Framework:** Spring Boot 3.3.5
- **Language:** Java 17
- **Database:** H2 (In-memory for demo purposes)
- **Security:** JCE (Java Cryptography Extension) for RSA-OAEP and AES-GCM
- **View:** Thymeleaf (Interactive Dashboard)

---

## 🚦 Quick Start

### Prerequisites
- **JDK 17** or newer installed.
- Terminal access.

### Run Locally
1. **Clone the repository:**
   ```bash
   git clone https://github.com/eklavya114/Mesh_Pay.git
   cd Mesh_Pay/UPI_Without_Internet
   ```

2. **Start the application:**
   ```cmd
   # On Windows
   .\mvnw.cmd spring-boot:run

   # On Mac/Linux
   ./mvnw spring-boot:run
   ```

3. **Access the Dashboard:**
   Open [http://localhost:8080](http://localhost:8080) in your browser to interact with the mesh simulator and watch real-time settlement.

---

## 🧪 Testing
The project includes a robust test suite, including a high-concurrency test to verify idempotency under stress:
```cmd
.\mvnw.cmd test
```

---

## 📄 License
This project is open-sourced under the [MIT License](LICENSE).
