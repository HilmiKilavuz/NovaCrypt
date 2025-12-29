# NovaCrypt

NovaCrypt is a custom symmetric encryption algorithm implementation in Python, designed for educational purposes to demonstrate the fundamental concepts of cryptography, such as Substitution-Permutation Networks (SPN), S-Boxes, and P-Boxes.

> [!WARNING]
> **Educational Use Only**: This algorithm is designed for learning and experimentation. It is **NOT** intended for securing sensitive or production data. Standard algorithms like AES (Advanced Encryption Standard) should be used for real-world security needs.

## Features

*   **Symmetric Encryption**: Uses the same key for both encryption and decryption.
*   **Block Cipher**: Operates on fixed-size blocks of 16 bytes.
*   **SPN Architecture**: Implements a Substitution-Permutation Network structure.
*   **Custom S-Box**: A mathematical substitution box defined by $f(x) = (3x + 7) \mod 256$.
*   **P-Box**: A simple permutation step involving byte rotation.
*   **Avalanche Effect**: Demonstrates high sensitivity to key changes (a single bit change in the key drastically alters the output).

## Algorithm Details

The encryption process consists of 4 rounds involving the following operations:

1.  **Key Mixing (XOR)**: The data block is XORed with the round key.
2.  **Substitution (S-Box)**: Each byte is substituted using the function:
    $$S(x) = (3x + 7) \mod 256$$
    The inverse S-Box for decryption is derived as:
    $$S^{-1}(y) = 171 \times (y - 7) \mod 256$$
3.  **Permutation (P-Box)**: The bytes in the block are shifted/rotated to diffuse bits across the block.

## usage

### Installation

No external libraries are required. The implementation relies only on Python's standard library.

### Example Code

Here is how to use the `NovaCrypt` class to encrypt and decrypt messages.

```python
import binascii

# Import the class (assuming existing code structure)
# from novacrypt import NovaCrypt 

# Or define the class as in the notebook...
class NovaCrypt:
    def __init__(self):
        self.block_size = 16 
        self.sbox = [(3 * i + 7) % 256 for i in range(256)]
        self.inv_sbox = [(171 * (i - 7)) % 256 for i in range(256)]

    def anahtar_uret(self, parola):
        p_bytes = parola.encode('utf-8')
        return (p_bytes * (16 // len(p_bytes) + 1))[:16]

    def _p_box(self, block, reverse=False):
        if not reverse:
            return block[1:] + block[:1]
        return block[-1:] + block[:-1]

    def sifrele(self, duz_metin, anahtar):
        if isinstance(duz_metin, str):
            duz_metin = duz_metin.encode('utf-8')
        data = duz_metin.ljust(self.block_size, b'\0')
        state = bytearray(data[:16])
        for i in range(4):
            state = bytearray(a ^ b for a, b in zip(state, anahtar))
            state = bytearray(self.sbox[b] for b in state)
            if i < 3:
                state = self._p_box(state)
        return bytes(state)

    def desifrele(self, sifreli_metin, anahtar):
        state = bytearray(sifreli_metin)
        for i in reversed(range(4)):
            if i < 3:
                state = self._p_box(state, reverse=True)
            state = bytearray(self.inv_sbox[b] for b in state)
            state = bytearray(a ^ b for a, b in zip(state, anahtar))
        return bytes(state).rstrip(b'\0')

# --- Usage Example ---

# 1. Initialize
nc = NovaCrypt()

# 2. Generate Key
password = "super_secret_password"
key = nc.anahtar_uret(password)

# 3. Encrypt
message = "Hello NovaCrypt!"
encrypted_bytes = nc.sifrele(message, key)
print(f"Encrypted (Hex): {encrypted_bytes.hex()}")

# 4. Decrypt
decrypted_bytes = nc.desifrele(encrypted_bytes, key)
print(f"Decrypted: {decrypted_bytes.decode('utf-8')}")
```

### Key Sensitivity Test (Avalanche Effect)

The algorithm demonstrates that even a slight change in the key produces a completely different result, effectively preventing similar keys from decrypting the same ciphertext.

```python
# Create a faulty key with just 1 bit difference
faulty_key = bytearray(key)
faulty_key[0] ^= 1 

# Attempt decryption
decrypted_faulty = nc.desifrele(encrypted_bytes, bytes(faulty_key))
print(f"Decrypted with faulty key: {decrypted_faulty.hex()}") 
# Result will be random/garbled bytes
```
