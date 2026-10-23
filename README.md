# MES1 (Mocca Encryption System)

Symmetric file encryption and line-level integrity verification protocol implemented in Python. 

MES1 enforces dual-secret authentication (password plus a required token/UUID), combined with authenticated encryption primitives and structured binary markers for tamper detection.

### Technical Overview

- **Cipher & Mode:** AES-256 in CBC mode with PKCS7 padding.
- **Key Derivation:** PBKDF2 (100,000 iterations default) utilizing a 32-byte cryptographically secure salt. The derived key material ties both the user-supplied passphrase and the device/session UUID together.
- **Integrity Validation:** Encrypt-then-MAC scheme using HMAC-SHA256 across payload blocks to detect ciphertext tampering prior to decryption attempts.
- **Line-by-Line Integrity Auditing:** Line-aware diffing mechanism that compares modified, inserted or deleted lines against baseline encrypted digests without displaying plaintext contents.
- **File Format Structure:**
  ```
  [HEADER: metadata, salt, iv, compression flag]
  [BODY: encrypted ciphertext]
  [FOOTER: HMAC-SHA256 signature]
  ```

### Requirements
- Python 3.8+
- `pycryptodome`

Install dependency:
```bash
pip install pycryptodome
```

### Usage

#### Library Integration
```python
import MES1

# Generate a unique session token
token_uuid = MES1.generate_uuid()

# Encrypt file
MES1.encrypt_file("target.txt", "passphrase", token_uuid)

# Decrypt file
MES1.decrypt_file("target.txt.mes1", "passphrase", token_uuid)

# Integrity verification
audit = MES1.check_file_integrity("target.mes1", "passphrase", token_uuid, "modified.txt")
if audit["status"] == "success":
    print(f"Modifications: {len(audit['modified_lines'])}, Additions: {len(audit['added_lines'])}, Removals: {len(audit['deleted_lines'])}")
```

#### CLI Interface
Run the interactive console utility:
```bash
python MES1_cli.py
```

### Protocol Caveats & Security Boundaries
- **Loss of UUID:** The UUID is not stored in the header; losing the UUID renders ciphertext mathematically unrecoverable.
- **Memory Footprint:** While explicit variable overwrites and garbage collection (`gc.collect()`) are invoked upon completion, CPython runtime semantics do not guarantee zeroization of immutable string allocations in RAM.
- **Compression:** Optional zlib compression occurs pre-encryption. For small or high-entropy inputs, ciphertext size may slightly exceed raw input size.

---

Maintained under [Tanuki Softworks](https://github.com/TanukiSoftworks).
