# Zero-Knowledge Password Authentication

A login demo where the browser proves it knows your password using a zero-knowledge proof, so the password never reaches the server.

## Why it exists

In a normal login, the password is sent to the server to be checked. This project tries a different approach: you register a hash of your password, and at login you prove you know a password that matches that hash without revealing it. I built it to learn how zero-knowledge circuits, proofs and verification fit together in a working web app.

## How it works

The circuit (`circuits/password_auth.circom`) has one private input, the password, and one public input, the password hash. It checks that the Poseidon hash of the password equals the public hash.

1. **Register.** The browser hashes the password with Poseidon and sends only the email, name and hash to `/register`. The server stores them in `backend/users.json`.
2. **Log in.** The browser fetches the stored hash from `/get-user`, then builds a Groth16 proof in the browser with SnarkJS. The proof uses the compiled circuit (`password_auth.wasm`) and the proving key (`password_auth_final.zkey`).
3. **Verify.** The browser sends the proof to `/verify`. The server checks it against `circuits/verification_key.json` and replies with success or failure. The browser then confirms the proof's public hash matches the stored one.

API:

| Method | Endpoint | What it does |
|---|---|---|
| POST | `/register` | Stores email, name and password hash |
| POST | `/get-user` | Returns the stored record for an email |
| POST | `/verify` | Verifies a Groth16 proof |

Folders:

| Path | Contents |
|---|---|
| `circuits/` | The Circom circuit, compiled files, keys and the verification key |
| `backend/` | Express server (port 3001) |
| `frontend/` | React 19 app with SCSS pages for register, login and dashboard |
| `generate_input.js` | Writes `inputs.json` for the sample password |
| `inputs.json`, `witness.wtns`, `proof.json`, `public.json` | Output from a sample run |

## How to run it

You need Node.js (`.nvmrc` says 18, and I tested with 20), [Circom](https://docs.circom.io/) 2.0 or newer, and the SnarkJS command line (`npm install -g snarkjs`).

```bash
npm install   # installs circomlib and circomlibjs
```

**Check the proof pipeline from the command line.** This uses the circuit files already in the repo.

```bash
node generate_input.js
snarkjs wtns calculate circuits/password_auth_js/password_auth.wasm inputs.json witness.wtns
snarkjs groth16 prove circuits/password_auth_final.zkey witness.wtns proof.json public.json
snarkjs groth16 verify circuits/verification_key.json public.json proof.json
```

The last command prints `OK!` when the proof is valid.

**Rebuild the circuit (optional).**

```bash
circom circuits/password_auth.circom --r1cs --wasm --sym -o circuits -l node_modules/circomlib/circuits
snarkjs groth16 setup circuits/password_auth.r1cs circuits/pot12_final.ptau circuits/password_auth_0000.zkey
snarkjs zkey contribute circuits/password_auth_0000.zkey circuits/password_auth_final.zkey --name="first contribution" -e="type some random text here"
snarkjs zkey export verificationkey circuits/password_auth_final.zkey circuits/verification_key.json
cp circuits/password_auth_js/password_auth.wasm circuits/password_auth_final.zkey frontend/public/
```

The last line copies the new circuit and key to where the browser app loads them. Do not run `generate_witness.js` directly: the root `package.json` sets `"type": "module"`, which breaks it. Use `snarkjs wtns calculate` as shown above.

**Run the app.**

```bash
cd backend && npm install && npm start    # http://localhost:3001
```

In a second terminal:

```bash
cd frontend && npm install && DISABLE_ESLINT_PLUGIN=true npm start    # http://localhost:3000
```

The `DISABLE_ESLINT_PLUGIN=true` part is needed because the repo has no lockfile, so a fresh install pulls newer lint packages that fail with a `jest/globals` error. Without it, the dev server reports one compile error. The frontend calls the backend at `http://localhost:3001`.

`inputs.json` uses a sample password (`123456`), and `backend/users.json` holds made-up test accounts. Both are sample data.

## Results

I tested this on a clean copy of the repo:

- The command-line steps above work. The proof for the sample input verifies, and the backend's `/verify` returns `{"success":true}` for it.
- In the browser, I registered a test user, logged out and logged back in. The browser built a proof and the server verified it. A wrong password was rejected with "Login failed."

No performance measurements are included.

## Known limitations (demo project)

- The server checks that a proof is valid, but not that it belongs to the account. The match against the stored hash is done in the browser, and the login state is kept in `localStorage`.
- `/get-user` returns the stored hash for any email.
- The stored value is an unsalted Poseidon hash, so a weak password could be guessed offline from it.
- Proofs are not tied to a one-time challenge, so a captured proof could be replayed.

## Tech stack

Circom, SnarkJS (Groth16), Poseidon hash (circomlib), Node.js with Express, React 19 with SCSS.
