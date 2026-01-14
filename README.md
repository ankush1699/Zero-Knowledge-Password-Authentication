# Zero-Knowledge Password Authentication

A secure authentication protocol that verifies user credentials without ever exposing the actual password, using zero-knowledge proofs (ZKPs).

## Overview

Traditional authentication systems require sending passwords to servers for verification, creating security vulnerabilities. This project implements a ZKP-based approach where users prove they know the password without revealing it.

## Features

- **Zero-Knowledge Proofs:** Verify credentials without exposing sensitive data
- **Poseidon Hashing:** Cryptographic hash function optimized for ZK circuits
- **Proof Generation:** Client-side proof creation using SnarkJS
- **Proof Verification:** Server-side verification without accessing plaintext passwords
- **Optimized Pipeline:** 25% reduction in verification latency

## Tech Stack

- **ZK Circuits:** Circom
- **Proof System:** SnarkJS (Groth16)
- **Backend:** Node.js, Express.js
- **Cryptography:** Poseidon Hash Function

## How It Works

1. **Setup Phase:** Generate proving and verification keys from Circom circuits
2. **Registration:** User's password is hashed using Poseidon and stored
3. **Authentication:** User generates a ZK proof that they know the password
4. **Verification:** Server verifies the proof without learning the password

## Project Structure
```
├── circuits/
│   └── auth.circom          # ZK circuit for password verification
├── build/
│   ├── auth.wasm            # Compiled circuit
│   ├── auth.zkey            # Proving key
│   └── verification_key.json
├── src/
│   ├── prover.js            # Proof generation logic
│   ├── verifier.js          # Proof verification logic
│   └── server.js            # Express API endpoints
├── package.json
└── README.md
```

## Getting Started

### Prerequisites

- Node.js 16+
- Circom 2.0+
- SnarkJS

### Installation

1. Clone the repository
```bash
   git clone https://github.com/ankush1699/Zero-Knowledge-Password-Authentication.git
   cd Zero-Knowledge-Password-Authentication
```

2. Install dependencies
```bash
   npm install
```

3. Compile the circuit
```bash
   circom circuits/auth.circom --r1cs --wasm --sym -o build
```

4. Generate keys (trusted setup)
```bash
   snarkjs groth16 setup build/auth.r1cs pot12_final.ptau build/auth.zkey
   snarkjs zkey export verificationkey build/auth.zkey build/verification_key.json
```

5. Start the server
```bash
   npm start
```

## API Endpoints

| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/register` | Register new user with hashed password |
| POST | `/authenticate` | Verify ZK proof for authentication |

## Use Cases

- Secure login systems
- Privacy-preserving identity verification
- Passwordless authentication
- Blockchain wallet authentication

## References

- [Circom Documentation](https://docs.circom.io/)
- [SnarkJS GitHub](https://github.com/iden3/snarkjs)
- [Poseidon Hash](https://www.poseidon-hash.info/)

## Contact

**Ankush Chaudhary** — [LinkedIn](https://linkedin.com/in/ankush1699) | ankushchaudhary.ac99@gmail.com
