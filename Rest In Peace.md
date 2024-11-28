### CTF Writeup: Rest In Peace (#353)

**Category**: Cryptography / Reverse Engineering  
**Difficulty**: Easy  
**Creator**: BM5  
**Flag Format**: SHCTF24{...}

---

### **Challenge Overview**

The challenge centers around an executable file, `RestInPeace.exe`, and a hidden flag encrypted within it. The provided hint suggests that there are encrypted hexadecimal blocks that can be decrypted using a specific method. The goal is to reverse engineer the executable and retrieve the flag.

---

### **Solution Approach**

1. **Extract Readable Strings**:  
   - Use the `strings` command to extract human-readable text from the executable. This helps in identifying encrypted blocks and key information.

2. **Identify Encrypted Data**:  
   - Spot hexadecimal data blocks in the extracted strings that appear to be encrypted.

3. **Understand the Encryption Mechanism**:  
   - The hint "reversed_hex" suggests that the hexadecimal data needs to be reversed. Additionally, the string `kd_jebatisnowonline` seems like a potential XOR key for decryption.

4. **Decrypt the Encrypted Data**:  
   - Reverse the hexadecimal string, convert it to bytes, and apply XOR decryption using the identified key.

5. **Combine Decrypted Parts**:  
   - Merge decrypted segments to obtain the complete flag.

---

### **Step-by-Step Breakdown**

#### **1. Extracting Readable Strings**
Run the `strings` command on the provided executable (`RestInPeace.exe`):

```bash
strings RestInPeace.exe
```

**Key Findings**:
- **Encrypted Hexadecimal Blocks**:
  - `b661f0a300d5f0136131f5c03470f0550532e3c1c283`
  - `318544c5f5c270301560a39523b3d5`
  
- **Relevant Hints**:
  - `reversed_hex`
  - `decrypted_flag`
  - `kd_jebatisnowonline`

#### **2. Identifying Encrypted Data**
Two encrypted hexadecimal strings were identified. These are most likely the encrypted parts of the flag.

#### **3. Understanding the Encryption Mechanism**
- The hint `reversed_hex` implies that we need to reverse the hexadecimal string before decrypting.
- The string `kd_jebatisnowonline` is likely the XOR key.

#### **4. Decrypting the Encrypted Blocks**
Create a Python script to automate the decryption process.

```python
# decrypt_flag.py

def reverse_hex(hex_str):
    return hex_str[::-1]

def hex_to_bytes(hex_str):
    try:
        return bytes.fromhex(hex_str)
    except ValueError:
        print("Invalid hex string.")
        return None

def xor_decrypt(byte_data, key):
    key_bytes = key.encode()
    decrypted = bytes([b ^ key_bytes[i % len(key_bytes)] for i, b in enumerate(byte_data)])
    return decrypted

def decrypt_flag(encrypted_hex, key):
    reversed_hex = reverse_hex(encrypted_hex)
    byte_data = hex_to_bytes(reversed_hex)
    if byte_data is None:
        return None
    decrypted_bytes = xor_decrypt(byte_data, key)
    try:
        decrypted_flag = decrypted_bytes.decode('utf-8')
        return decrypted_flag
    except UnicodeDecodeError:
        # If decoding fails, return the raw decrypted bytes
        return decrypted_bytes

if __name__ == "__main__":
    # Encrypted flag blocks
    encrypted_flag1 = 'b661f0a300d5f0136131f5c03470f0550532e3c1c283'
    encrypted_flag2 = '318544c5f5c270301560a39523b3d5'
    
    # Possible XOR key (based on strings)
    key = 'kd_jebatisnowonline'
    
    # Decrypt the first flag
    flag1 = decrypt_flag(encrypted_flag1, key)
    print(f"Decrypted Flag 1: {flag1}")
    
    # Decrypt the second flag
    flag2 = decrypt_flag(encrypted_flag2, key)
    print(f"Decrypted Flag 2: {flag2}")
```

**Execution Output**:
```bash
Decrypted Flag 1: SHCTF24{n0b0dy_c4n_dr4
Decrypted Flag 2: 6_m3_d0wn_1337}
```

#### **5. Combining Decrypted Parts**
When combined, the two decrypted parts form the full flag:

```
SHCTF24{n0b0dy_c4n_dr46_m3_d0wn_1337}
```

---

### **Final Flag**:
```
SHCTF24{n0b0dy_c4n_dr46_m3_d0wn_1337}
```

---

### **Additional Insights**:
- **XOR Key Identification**:  
  The key `kd_jebatisnowonline` was extracted from the strings within the executable. It was used to decrypt the data using XOR.

- **Encryption Method Confirmation**:  
  The presence of the hint `reversed_hex` confirmed that the hexadecimal data needed to be reversed before decryption.

- **Error Handling**:  
  The Python script includes error handling for invalid hexadecimal strings and decoding errors, ensuring smooth decryption even in cases where the data is not directly readable.

- **Flag Structure**:  
  Recognizing the expected flag format (`SHCTF24{...}`) helped in confirming the correctness of the decrypted result.

---

### **Conclusion**:
This challenge required reverse engineering skills and basic cryptographic knowledge. By extracting strings from the executable, identifying encrypted data blocks, and applying XOR decryption with the correct key, we successfully retrieved the flag.

---

### **Recommendations for Future Challenges**:
1. **Analyze Extracted Strings**:  
   - Always check for patterns, keys, and hints embedded in the strings.
   
2. **Leverage Hints**:  
   - Use the hints like `reversed_hex` to narrow down the decryption approach.

3. **Automate Decryption**:  
   - Utilize Python scripts or other tools to handle repetitive decryption tasks.

4. **Validate Output**:  
   - Ensure the decrypted text follows the expected flag format.

