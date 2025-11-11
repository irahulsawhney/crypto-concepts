# Crypto Fundamentals — Rapid Ramp Cheat Sheet

**Goal:** Crisp, plain-English answers + mental models you can recall under pressure.

---

## Trust Models & JWT
- **JWKS:** Issuer's JSON Web Key Set of *public keys* (has `kid`, `kty`, etc.).  
  **Rotation:** Add new key → sign with new `kid` → keep old until all old tokens expire → remove old key.
- **Verify these claims:**  
  `aud` = token is for **your API**; `iss` = matches **your trusted issuer**; time claims `exp/nbf/iat` = **currently valid**.
- **HS256 vs RS256/EdDSA:**  
  HS256 = one trust domain (issuer==verifier). RS256/EdDSA = many verifiers; publisher keeps private key, verifiers use JWKS.
- **OAuth/OIDC `state`:** Protects against **CSRF/redirect hijack** on the auth redirect.
- **JWE vs JWS:** Use **JWE** when you also need **confidentiality** (e.g., PII in the token); JWS gives **authenticity/integrity** only.

## AEAD & TLS
- **AEAD rule:** **Never reuse a nonce with the same key** (breaks security). Tag = integrity; AD = binds context/headers.
- **TLS + ECDHE:** **Never reuse ephemeral ECDH keypairs** across sessions → enables **forward secrecy**.
- **HTTPS stack:** `HTTP → TLS → TCP → IP`.
- **Why not pure asymmetric for data:** Too **slow** for bulk; **larger ciphertext** & operational complexity.

## Key Management & Ops
- **Envelope encryption:** Data Key (DEK) encrypts data; Master/KEK wraps DEK (KMS/HSM).
- **Crypto version field (per record):** `algo + mode/params + key_id (+ version)`.
- **Rotation:**  
  *Master-key*: **rewrap DEKs only**.  *Data-key*: **re‑encrypt data** (batch backfill *or* lazy on-read → write-back).
- **Unwrap authorization:** Only the **service identity** can unwrap (enforced by **KMS key policy**).
- **Encryption context:** Bind labels (e.g., `tenant_id:record_uuid`) so a stolen wrapped key can't be unwrapped elsewhere.
- **Never log in plaintext:** Keys/secrets/tokens; sensitive PII; (often) nonces/tags/ciphertext for sensitive records.
- **Key-incident (first steps):** Disable/rotate affected keys; revoke/expire tokens; start re-encrypt plan; audit & notify.

## Mini System Design — Encrypted Notes
- **Threat model:** Protect against lost DB/rogue admin; client compromise out of scope; TLS covers MITM.
- **Record fields:** `nonce, tag, ciphertext, key_id, wrapped_DEK, AD or AD-hash, crypto_version, created_at`.
- **Write path:** Gen DEK → AEAD(plaintext, AD) → wrap DEK in KMS → store fields.
- **Read path:** AuthZ → KMS unwrap with context → verify tag → decrypt.
- **Rotation:** KEK rewrap; DEK re-encrypt (batch or lazy). Keep old versions until migration completes.
- **Ops/guardrails:** Metrics on unwrap/decrypt failures; rate limits; paved SDKs; nonce mgmt; no secrets in logs; test vectors & fuzzing.

---

## 10 One‑liners to memorize
1) JWKS = issuer's public keys; select by `kid`.  
2) Rotation = add new key; keep old until tokens expire; remove old.  
3) Verify `aud`, `iss`, and time claims.  
4) HS256 for one trust domain; RS256/EdDSA for many verifiers.  
5) `state` prevents CSRF/redirect-hijack.  
6) JWE when you need confidentiality in addition to integrity.  
7) AEAD nonce must be unique per key.  
8) ECDHE ephemeral keys must never be reused (forward secrecy).  
9) HTTPS stack: HTTP → TLS → TCP → IP.  
10) Envelope: DEK encrypts data; KEK wraps DEK; rotate KEK by rewrapping, DEK by re-encrypting.
