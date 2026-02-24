# Post-Quantum Zero Trust Architecture for Securing 5G Edge Networks

## Project Overview

This project demonstrates a **Zero Trust Architecture** with **Post-Quantum Cryptography (PQC) readiness** for securing 5G Edge Networks. The implementation uses simulated PQC algorithms:

- **X25519** (Elliptic Curve Diffie-Hellman) for key exchange
- **Ed25519** (Edwards-curve Digital Signature Algorithm) for authentication
- **AES-GCM** (Advanced Encryption Standard - Galois/Counter Mode) for data encryption

> **Note**: This is a simulated PQC implementation. In production, these would be replaced with NIST-standardized post-quantum algorithms (e.g., CRYSTALS-Kyber, CRYSTALS-Dilithium).

## Architecture Diagram

```
┌─────────────────────────────────────────────────────────────────┐
│                    Zero Trust Handshake Flow                     │
└─────────────────────────────────────────────────────────────────┘

    CLIENT                                    SERVER
    ──────                                    ──────
      │                                          │
      │  1. Generate Key Pair                    │
      │     - X25519 (key exchange)              │
      │     - Ed25519 (signature)                │
      │                                          │
      │  2. Auth Request                         │
      │     ┌─────────────────────────────────┐  │
      │     │ client_id                       │  │
      │     │ client_public_key (X25519)      │  │
      │     │ signature (Ed25519)             │  │
      │     │ timestamp                       │  │
      │     └─────────────────────────────────┘  │
      ├─────────────────────────────────────────>│
      │                                          │
      │                                          │ 3. Verify Signature
      │                                          │    - Check Ed25519 signature
      │                                          │    - Validate timestamp
      │                                          │
      │                                          │ 4. Generate Server Key Pair
      │                                          │    - X25519 private/public
      │                                          │
      │  5. Key Exchange Response                │
      │     ┌─────────────────────────────────┐  │
      │     │ server_public_key (X25519)      │  │
      │     │ encrypted_challenge (AES-GCM)   │  │
      │     └─────────────────────────────────┘  │
      │<─────────────────────────────────────────┤
      │                                          │
      │  6. Derive Shared Secret                 │
      │     - X25519(client_priv, server_pub)    │
      │     - Derive AES key from shared secret  │
      │                                          │
      │  7. Decrypt Challenge                    │
      │     - Decrypt using AES-GCM              │
      │                                          │
      │  8. Challenge Response                   │
      │     ┌─────────────────────────────────┐  │
      │     │ encrypted_response (AES-GCM)   │  │
      │     └─────────────────────────────────┘  │
      ├─────────────────────────────────────────>│
      │                                          │
      │                                          │ 9. Verify Challenge
      │                                          │    - Decrypt response
      │                                          │    - Validate challenge
      │                                          │
      │  10. Secure Communication Established    │
      │      - All messages encrypted (AES-GCM)  │
      │      - Zero Trust verified               │
      │                                          │
```

## Zero Trust Principles

1. **Never Trust, Always Verify**: Every request is authenticated and authorized
2. **Least Privilege Access**: Clients must prove identity before access
3. **Continuous Verification**: Challenge-response mechanism ensures ongoing authentication
4. **Encrypted Communication**: All data is encrypted using AES-GCM after handshake

## Project Structure

```
project/
│── README.md                 # This file
│── requirements.txt          # Python dependencies
│── server/
│     ├── main.py            # FastAPI server entry point
│     ├── zero_trust.py      # Zero Trust authentication logic
│     └── key_manager.py     # X25519 key exchange management
│── client/
│     └── client.py          # Client implementation with handshake
│── crypto/
│     └── aes_gcm.py         # AES-GCM encryption/decryption
│── tests/
│     └── test_handshake.py  # Unit tests for handshake flow
```

## Installation

### Prerequisites

- Python 3.8 or higher
- pip (Python package manager)

### Step 1: Install Dependencies

```bash
pip install -r requirements.txt
```

### Step 2: PQC Library Setup (Windows)

The Post-Quantum Cryptography (PQC) integration relies on the `liboqs` C library.
Because no official pre-compiled Windows binaries exist, you must provide the `liboqs.dll` manually.

**Option 1: Quick Fix (Download Pre-Built DLL)**

1.  **Download** the `liboqs.dll` file from a trusted source (e.g., Open Quantum Safe CI Artifacts).
2.  **Place the file** in the project root: `c:\Users\yksk7\PQC\liboqs.dll`

**Option 2: Build from Source (Recommended)**

If you have Visual Studio Build Tools installed and the `liboqs` source code in `liboqs/`:

1.  Open **Developer Command Prompt for VS 2022**.
2.  Run the following commands:
    ```cmd
    cd c:\Users\yksk7\PQC\liboqs
    mkdir build
    cd build
    cmake -G "Visual Studio 17 2022" -A x64 -DBUILD_SHARED_LIBS=ON ..
    cmake --build . --config Release
    copy Release\oqs.dll ..\..\liboqs.dll
    ```

**Option 3: CUDA Hardware Acceleration (GPU)**

To offload PQC operations to an NVIDIA GPU (requires CUDA Toolkit):
1.  Follow the prerequisites in `CUDA_INTEGRATION.md`.
2.  Run the build script:
    ```cmd
    build_with_cuda.bat
    ```
3.  This will automatically build `liboqs` with cuPQC support and update your project DLL.


## Execution Instructions

### Step 1: Start the Server

Open a terminal and navigate to the project directory:

```bash
cd project
python -m uvicorn server.main:app --host 0.0.0.0 --port 8000
```

You should see output like:
```
INFO:     Started server process
INFO:     Waiting for application startup.
INFO:     Application startup complete.
INFO:     Uvicorn running on http://0.0.0.0:8000
```

### Step 2: Run the Client

Open a **new terminal** (keep the server running) and navigate to the project directory:

```bash
cd project
python client/client.py
```

### Step 3: Verify the Handshake

The client will:
1. Generate cryptographic keys (X25519 + Ed25519)
2. Send authentication request to server
3. Perform key exchange
4. Complete challenge-response verification
5. Send encrypted messages

## Expected Output

### Server Output

```
INFO:     Started server process [12345]
INFO:     Waiting for application startup.
INFO:     Application startup complete.
INFO:     Uvicorn running on http://0.0.0.0:8000 (Press CTRL+C to quit)
INFO:     127.0.0.1:54321 - "POST /auth/request HTTP/1.1" 200 OK
INFO:     Zero Trust: Client authenticated successfully
INFO:     127.0.0.1:54321 - "POST /auth/exchange HTTP/1.1" 200 OK
INFO:     Zero Trust: Key exchange completed
INFO:     127.0.0.1:54321 - "POST /auth/challenge HTTP/1.1" 200 OK
INFO:     Zero Trust: Challenge verified, secure channel established
INFO:     127.0.0.1:54321 - "POST /secure/message HTTP/1.1" 200 OK
```

### Client Output

```
=== Post-Quantum Zero Trust Client ===

[1/5] Generating cryptographic keys...
    ✓ X25519 key pair generated
    ✓ Ed25519 key pair generated

[2/5] Sending authentication request...
    ✓ Authentication request sent
    ✓ Server verified signature

[3/5] Performing key exchange...
    ✓ Server public key received
    ✓ Shared secret derived
    ✓ AES encryption key derived

[4/5] Completing challenge-response...
    ✓ Challenge decrypted: "ZERO_TRUST_CHALLENGE_2024"
    ✓ Challenge response sent
    ✓ Server verified challenge

[5/5] Sending secure message...
    ✓ Message encrypted and sent
    ✓ Server response: {"status": "success", "message": "Secure communication established"}

=== Zero Trust Handshake Complete ===
```

## Running Tests

To run the test suite:

```bash
python -m pytest tests/test_handshake.py -v
```

Expected test output:
```
tests/test_handshake.py::test_key_generation PASSED
tests/test_handshake.py::test_signature_verification PASSED
tests/test_handshake.py::test_key_exchange PASSED
tests/test_handshake.py::test_aes_encryption PASSED
tests/test_handshake.py::test_full_handshake PASSED
```

## API Endpoints

### POST `/auth/request`
Initial authentication request from client.

**Request Body:**
```json
{
  "client_id": "client_123",
  "client_public_key": "base64_encoded_x25519_public_key",
  "signature": "base64_encoded_ed25519_signature",
  "timestamp": 1234567890
}
```

**Response:**
```json
{
  "status": "authenticated",
  "message": "Signature verified"
}
```

### POST `/auth/exchange`
Key exchange endpoint.

**Request Body:**
```json
{
  "client_id": "client_123",
  "client_public_key": "base64_encoded_x25519_public_key"
}
```

**Response:**
```json
{
  "server_public_key": "base64_encoded_x25519_public_key",
  "encrypted_challenge": "base64_encoded_aes_gcm_encrypted_data"
}
```

### POST `/auth/challenge`
Challenge response verification.

**Request Body:**
```json
{
  "client_id": "client_123",
  "encrypted_response": "base64_encoded_aes_gcm_encrypted_data"
}
```

**Response:**
```json
{
  "status": "verified",
  "message": "Challenge verified, secure channel established"
}
```

### POST `/secure/message`
Send encrypted message after handshake.

**Request Body:**
```json
{
  "client_id": "client_123",
  "encrypted_message": "base64_encoded_aes_gcm_encrypted_data"
}
```

**Response:**
```json
{
  "status": "success",
  "message": "Secure communication established"
}
```

## Security Features

1. **Ed25519 Digital Signatures**: Ensures message authenticity and integrity
2. **X25519 Key Exchange**: Provides forward secrecy through ephemeral key exchange
3. **AES-GCM Encryption**: Authenticated encryption with associated data (AEAD)
4. **Timestamp Validation**: Prevents replay attacks
5. **Challenge-Response**: Ensures client can decrypt and respond to challenges

## Post-Quantum Readiness

While this implementation uses classical cryptography (X25519, Ed25519, AES-GCM), the architecture is designed to be **post-quantum ready**. In a production environment, these would be replaced with:

- **CRYSTALS-Kyber** (Key Encapsulation Mechanism) instead of X25519
- **CRYSTALS-Dilithium** or **FALCON** (Digital Signatures) instead of Ed25519
- **AES-256-GCM** remains quantum-safe (symmetric encryption)

## Limitations

- This is a **simulated** PQC implementation for educational purposes
- No persistent key storage (keys are generated per session)
- No certificate management (simplified for demonstration)
- Single server instance (no load balancing)

## Future Enhancements

- [ ] Implement actual NIST PQC algorithms (CRYSTALS-Kyber, CRYSTALS-Dilithium)
- [ ] Add certificate-based authentication
- [ ] Implement session management and token refresh
- [ ] Add rate limiting and DDoS protection
- [ ] Support for multiple clients and concurrent sessions
- [ ] Key rotation and forward secrecy improvements

## License

This project is for educational and demonstration purposes.

## Author

Senior Python Architect & Cybersecurity Engineer

