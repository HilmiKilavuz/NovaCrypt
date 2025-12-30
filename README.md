# NovaCrypt Project

NovaCrypt is a custom symmetric encryption algorithm implementation in Python, designed specifically for **educational purposes**. It demonstrates fundamental concepts of modern cryptography, such as Substitution-Permutation Networks (SPN), S-Boxes (Substitution Boxes), and P-Boxes (Permutation Boxes).

The project also includes `SecureNovaCrypt`, an extension implementing a Time-Based One-Time Password (TOTP) generator tailored for key exchange or authentication simulations.

> [!WARNING]
> **EDUCATIONAL USE ONLY**
> This algorithm is **NOT SECURE** for production use.
> *   **Linear S-Box**: The substitution logic is mathematically linear ($3x + 7$), making it vulnerable to linear cryptanalysis.
> *   **Weak Key Schedule**: The key is simply repeated, offering poor resistance against related-key attacks.
> *   **Limited Block Mode**: The current implementation only encrypts the *first 16 bytes* of data.
> *   **Zero Padding**: Uses non-standard zero padding which can be ambiguous.
>
> For real-world data protection, use established standards like **AES (Advanced Encryption Standard)**.

## Project Structure

*   `NovaCrypt.ipynb`: Jupyter Notebook containing the source code, class definitions, and test scenarios.
*   `README.md`: This documentation file.

---

## Code Review & Architecture Analysis

Methods and classes have been reviewed to understand the internal mechanisms. Below is a breakdown of the architecture:

### 1. `NovaCrypt` Class (Base Cipher)

This class implements a simplified SPN block cipher.

*   **Block Size**: 16 Bytes (128 bits).
*   **Rounds**: 4 Rounds.
*   **Key Schedule**:
    *   *Implementation*: `anahtar_uret` repeats the password bytes to fill the block size.
    *   *Critique*: Extremely simple. Does not provide sufficient "avalanche effect" for the key itself across rounds.
*   **S-Box (Substitution)**:
    *   *Formula*: $S(x) = (3x + 7) \mod 256$
    *   *inverse*: $S^{-1}(y) = 171 \times (y - 7) \mod 256$
    *   *Critique*: This is an affine transformation, which is linear. A strong S-Box must be non-linear to resist linear cryptanalysis.
*   **P-Box (Permutation)**:
    *   *Implementation*: Rotates bytes left (encryption) or right (decryption).
    *   *Critique*: Provides diffusion, but simple rotation is less effective than bit-level permutations used in DES or AES.
*   **Encryption Loop**:
    1.  **XOR** with Key.
    2.  **Substitute** (S-Box).
    3.  **Permute** (P-Box) (except in the last round).

### 2. `SecureNovaCrypt` Class (OTP Extension)

Inherits from `NovaCrypt` to demonstrate a stronger variant for OTP generation.

*   **Improvements**:
    *   **Complex S-Box**: uses `(i * 31 + 17) ^ 0x63` to add some non-linearity via XOR.
    *   **More Rounds**: Increases rounds from 4 to 8.
*   **OTP Mechanism**:
    *   Uses system time (`time.time() / 30`) as the message seed (30-second window).
    *   Packs time into 8 bytes (`struct.pack('>q')`) and pads with `0x55`.
    *   Encrypts this time-block using a secret key.
    *   **Dynamic Truncation**: Extracts a 4-byte integer from the encrypted block based on the last byte's value (similar to HMAC-based OTP).
    *   Returns a 6-digit code.

---

## Usage Guide

### Prerequisites
*   Python 3.x
*   No external libraries are required (uses standard `hashlib`, `time`, `struct`).

### Basic Encryption (NovaCrypt)

```python
# Assuming NovaCrypt class is defined (copy from NovaCrypt.ipynb)
nc = NovaCrypt()
key = nc.anahtar_uret("my_secret_password")

# Encryption
plaintext = "Hello World!"
ciphertext = nc.sifrele(plaintext, key)
print(f"Ciphertext (Hex): {ciphertext.hex()}")

# Decryption
decrypted = nc.desifrele(ciphertext, key)
print(f"Decrypted: {decrypted.decode()}")
```

> [!IMPORTANT]
> The encryption method `sifrele` only processes the first 16 bytes of the input string due to the line `state = bytearray(data[:16])`. Input longer than 16 bytes will be truncated!

### Time-Based OTP (SecureNovaCrypt)

This simulates a Google Authenticator-style OTP generator.

```python
import time

# Initialize
secure_nc = SecureNovaCrypt()
shared_secret = "company_master_key"

# Generate Code
otp = secure_nc.generate_otp(shared_secret)
remaining_seconds = 30 - (int(time.time()) % 30)

print(f"Current OTP: {otp}")
print(f"Valid for: {remaining_seconds} seconds")
```

## Security Vulnerability Details

For students and developers reviewing this code, pay attention to these specific vulnerabilities:

1.  **ECB Mode limitation**: The cipher encrypts a single block. It does not implement chaining modes (CBC, GCM) required for messages longer than one block.
2.  **Padding Oracle**: The `ljust` zero-padding does not indicate the original length of the message. If the message ends with null bytes, they will be lost upon `rstrip(b'\0')`.
3.  **Fixed S-Box**: In a real-world scenario, S-Boxes are carefully constructed to minimize differential and linear probability. The formula-based S-Box here is predictable.


