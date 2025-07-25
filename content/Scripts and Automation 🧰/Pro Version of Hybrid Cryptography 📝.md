Are you an encryption expert already? Why goodness gracious, we are happy to have you here! Check out the Pro Version of this script at the link below, where we can assist you with maximum-level military-grade encryption protocols that keep your data as secure as possible!

In case you were wondering whether this script would be right for your organization, then let's put your concerns to rest: **it's absolutely overkill**. But if you're curious about what we CAN institute in disaster-proofing scenarios, then by all means check out the most high-level cryptography script that we could make for you. This document is mostly for us to reference when building out this app for you to use - however, the transparency that this platform provides allows us to share with the morbidly curious exactly what is going on behind the scenes!

If you would rather skip all of the technical documentation that lives here, then here's the .zip which contains this tool - it's ready to go, just download and install it!

![[HybridEncryptorPro.zip]]

## ✅ **How to run this**

1️. Save it as: `Hybrid-Encryptor-GUI.ps1`  
2️. Right-click → Run with PowerShell  
_(Or run in a PowerShell window with `Set-ExecutionPolicy Bypass -Scope Process` if needed)_  
3️. Click **Select Target** → choose file or folder  
4️. Click **Encrypt** → done!  
5️. Click **Decrypt** → supply the keys → done!

---

## ✅ **What it does**

✔️ Uses your **hybrid encryption**  
✔️ Generates keys automatically  
✔️ Labels & saves output cleanly  
✔️ Keeps your backup keys separate  
✔️ Simple Windows Forms UI, no extra installs

## ✅ **🗂️ Directory layout**

(lua)
```
project/
 ├─ secure_encryptor.py
 ├─ encryptor_lib.py
 ├─ keys/
 ├─ output/
 └─ requirements.txt

```

## ✅ **1️. `encryptor_lib.py` — core logic**

Here’s a **realistic, secure baseline** for the upgraded version (Python):

```
import os
import zipfile
import secrets
from cryptography.hazmat.primitives.asymmetric import rsa, padding
from cryptography.hazmat.primitives import serialization, hashes
from cryptography.hazmat.primitives.ciphers import Cipher, algorithms, modes
from cryptography.hazmat.primitives import hashes, hmac

KEYS_DIR = "keys"
OUTPUT_DIR = "output"

os.makedirs(KEYS_DIR, exist_ok=True)
os.makedirs(OUTPUT_DIR, exist_ok=True)

def generate_rsa_keys(name):
    private_key = rsa.generate_private_key(
        public_exponent=65537,
        key_size=4096,
    )
    private_pem = private_key.private_bytes(
        encoding=serialization.Encoding.PEM,
        format=serialization.PrivateFormat.PKCS8,
        encryption_algorithm=serialization.NoEncryption()
    )
    public_pem = private_key.public_key().public_bytes(
        encoding=serialization.Encoding.PEM,
        format=serialization.PublicFormat.SubjectPublicKeyInfo
    )

    private_key_path = os.path.join(KEYS_DIR, f"PrivateKey_{name}.pem")
    public_key_path = os.path.join(KEYS_DIR, f"PublicKey_{name}.pem")

    with open(private_key_path, 'wb') as f:
        f.write(private_pem)
    with open(public_key_path, 'wb') as f:
        f.write(public_pem)

    return private_key, public_pem

def zip_target(target_path, zip_name):
    with zipfile.ZipFile(zip_name, 'w', zipfile.ZIP_DEFLATED) as zipf:
        if os.path.isdir(target_path):
            for root, _, files in os.walk(target_path):
                for file in files:
                    abs_path = os.path.join(root, file)
                    rel_path = os.path.relpath(abs_path, start=target_path)
                    zipf.write(abs_path, rel_path)
        else:
            zipf.write(target_path, os.path.basename(target_path))

def encrypt_file_or_folder(target_path):
    name = os.path.basename(target_path).replace(' ', '_')
    private_key, public_pem = generate_rsa_keys(name)

    zip_name = f"{OUTPUT_DIR}/{name}.zip"
    zip_target(target_path, zip_name)

    # Generate AES key/IV
    aes_key = secrets.token_bytes(32)
    iv = secrets.token_bytes(16)

    # Encrypt zip
    with open(zip_name, 'rb') as f:
        plaintext = f.read()

    cipher = Cipher(algorithms.AES(aes_key), modes.CFB(iv))
    encryptor = cipher.encryptor()
    ciphertext = encryptor.update(plaintext) + encryptor.finalize()

    encrypted_file = os.path.join(OUTPUT_DIR, f"Encrypted_{name}.bin")
    with open(encrypted_file, 'wb') as f:
        f.write(ciphertext)

    # Encrypt AES key with RSA
    public_key = serialization.load_pem_public_key(public_pem)
    aes_package = iv + aes_key
    encrypted_aes_key = public_key.encrypt(
        aes_package,
        padding.OAEP(
            mgf=padding.MGF1(algorithm=hashes.SHA256()),
            algorithm=hashes.SHA256(),
            label=None
        )
    )
    aes_key_file = os.path.join(OUTPUT_DIR, f"AESKey_{name}.bin")
    with open(aes_key_file, 'wb') as f:
        f.write(encrypted_aes_key)

    # Sign encrypted file
    signer = private_key.sign(
        ciphertext,
        padding.PSS(
            mgf=padding.MGF1(hashes.SHA256()),
            salt_length=padding.PSS.MAX_LENGTH
        ),
        hashes.SHA256()
    )
    signature_file = os.path.join(OUTPUT_DIR, f"Signature_{name}.sig")
    with open(signature_file, 'wb') as f:
        f.write(signer)

    os.remove(zip_name)

    print(f"✅ Encrypted file: {encrypted_file}")
    print(f"🔑 AES Key file: {aes_key_file}")
    print(f"🔏 Signature: {signature_file}")
    print(f"🔒 RSA keys stored in: {KEYS_DIR}")

def decrypt_file(encrypted_file, aes_key_file, private_key_file, signature_file, output_folder):
    with open(private_key_file, 'rb') as f:
        private_key = serialization.load_pem_private_key(f.read(), password=None)

    with open(aes_key_file, 'rb') as f:
        encrypted_aes_key = f.read()

    aes_package = private_key.decrypt(
        encrypted_aes_key,
        padding.OAEP(
            mgf=padding.MGF1(algorithm=hashes.SHA256()),
            algorithm=hashes.SHA256(),
            label=None
        )
    )
    iv, aes_key = aes_package[:16], aes_package[16:]

    with open(encrypted_file, 'rb') as f:
        ciphertext = f.read()

    # Verify signature
    public_key = private_key.public_key()
    with open(signature_file, 'rb') as f:
        signature = f.read()
    try:
        public_key.verify(
            signature,
            ciphertext,
            padding.PSS(
                mgf=padding.MGF1(hashes.SHA256()),
                salt_length=padding.PSS.MAX_LENGTH
            ),
            hashes.SHA256()
        )
        print("✅ Signature verified: file has not been tampered with.")
    except:
        raise Exception("❌ Signature verification failed!")

    cipher = Cipher(algorithms.AES(aes_key), modes.CFB(iv))
    decryptor = cipher.decryptor()
    plaintext = decryptor.update(ciphertext) + decryptor.finalize()

    temp_zip = os.path.join(output_folder, 'decrypted_temp.zip')
    with open(temp_zip, 'wb') as f:
        f.write(plaintext)

    with zipfile.ZipFile(temp_zip, 'r') as zipf:
        zipf.extractall(output_folder)

    os.remove(temp_zip)
    print(f"✅ Decrypted to: {output_folder}")

```

## ✅ **2️. `secure_encryptor.py` — the user-facing wrapper**

(Python)

```
import encryptor_lib
import sys
import os

def main():
    print("Hybrid Encryptor Pro")
    print("======================")
    print("1) Encrypt a file/folder")
    print("2) Decrypt an encrypted file")
    choice = input("Choose [1/2]: ")

    if choice == '1':
        target = input("Enter full path to file/folder: ").strip('"')
        encryptor_lib.encrypt_file_or_folder(target)

    elif choice == '2':
        encrypted_file = input("Encrypted file (.bin): ").strip('"')
        aes_key_file = input("AES key file (.bin): ").strip('"')
        private_key_file = input("Private RSA key (.pem): ").strip('"')
        signature_file = input("Signature file (.sig): ").strip('"')
        output_folder = input("Output folder: ").strip('"')
        encryptor_lib.decrypt_file(encrypted_file, aes_key_file, private_key_file, signature_file, output_folder)

    else:
        print("Invalid choice. Exiting.")

if __name__ == "__main__":
    main()

```

## ✅ **3️. `requirements.txt`**

`cryptography`

## ✅ **4️. How to run**

(bash)
```
# Install dependencies
pip install -r requirements.txt 
# Encrypt 
python secure_encryptor.py
```

## ✅ **5️. How to package**

Use PyInstaller:

`pip install pyinstaller
`pyinstaller --onefile secure_encryptor.py`

This makes a single `.exe` for your users — no Python needed.

Moving on, here's the big bad packaged version with a clean GUI built-in. This is basically a ready-made repo, so use it in good health!
## ✅ **📦 1️. Project structure**

Here’s how the final **Pro Encryptor** repo will be organized:

(pgsql)
```
HybridEncryptorPro/
 ├─ encryptor_lib.py         # Core crypto logic
 ├─ secure_encryptor.py      # CLI launcher
 ├─ gui_encryptor.py         # Optional: simple GUI (tkinter or PySimpleGUI)
 ├─ requirements.txt         # Python dependencies
 ├─ build_instructions.md    # How to build to .exe/.app
 ├─ README.md                # End-user guide
 ├─ keys/                    # RSA keys storage (secure this!)
 ├─ output/                  # Encrypted files, AES keys, signatures
 └─ examples/                # Example test files/folders

```

## ✅ **📌 2️. `requirements.txt`**

`cryptography PySimpleGUI`

## ✅ **📌 3️. GUI version (`gui_encryptor.py`)**

Here’s a **simple starter GUI** (cross-platform, easy for non-tech end users):
```
import PySimpleGUI as sg
import encryptor_lib
import os

sg.theme('DarkBlue')

layout = [
    [sg.Text('Hybrid Encryptor Pro')],
    [sg.Button('Encrypt File/Folder')],
    [sg.Button('Decrypt Encrypted File')],
    [sg.Button('Exit')]
]

window = sg.Window('Hybrid Encryptor Pro', layout)

while True:
    event, values = window.read()
    if event in (sg.WINDOW_CLOSED, 'Exit'):
        break

    if event == 'Encrypt File/Folder':
        path = sg.popup_get_file('Choose a file or folder', no_window=True)
        if path:
            encryptor_lib.encrypt_file_or_folder(path)
            sg.popup('Done!', 'Encrypted and keys saved in output/')

    elif event == 'Decrypt Encrypted File':
        encrypted_file = sg.popup_get_file('Select encrypted .bin file', no_window=True)
        aes_key_file = sg.popup_get_file('Select AES Key file (.bin)', no_window=True)
        private_key_file = sg.popup_get_file('Select Private RSA Key (.pem)', no_window=True)
        signature_file = sg.popup_get_file('Select Signature file (.sig)', no_window=True)
        output_folder = sg.popup_get_folder('Select output folder', no_window=True)

        if all([encrypted_file, aes_key_file, private_key_file, signature_file, output_folder]):
            try:
                encryptor_lib.decrypt_file(encrypted_file, aes_key_file, private_key_file, signature_file, output_folder)
                sg.popup('✅ Decryption successful!', f'Files restored to {output_folder}')
            except Exception as e:
                sg.popup('❌ Error', str(e))

window.close()

```

## ✅ **📌 4️. Build instructions**

Create **`build_instructions.md`**:

(markdown)
```
# 🚀 Build Instructions

## 🐍 Install dependencies
```bash
pip install -r requirements.txt
```

## 📦 Build CLI version

`pyinstaller --onefile secure_encryptor.py`

## 🖥️ Build GUI version

`pyinstaller --onefile --windowed gui_encryptor.py`

- The output `.exe` will be in the `dist/` folder.
    
- Keys and output files go in `keys/` and `output/` (created automatically).

## ✅ **📌 5️. `README.md`**

A clean **end-user guide**:

```
# Hybrid Encryptor Pro

**Hybrid Encryptor Pro** is a simple tool for safe encryption of files/folders using modern asymmetric (RSA) + symmetric (AES) encryption with digital signatures.

## ✅ How to Encrypt

1. Run `secure_encryptor.exe` or `gui_encryptor.exe`
2. Choose **Encrypt File/Folder**
3. Pick your target
4. Find the encrypted file + AES key + signature in the `output/` folder
5. Store your `PrivateKey_...pem` securely!

## 🔑 How to Decrypt

1. Run again
2. Choose **Decrypt**
3. Select:
   - The `.bin` file
   - The AES key `.bin`
   - The RSA private key `.pem`
   - The `.sig` file
   - Output folder for decrypted files

4. The tool verifies the signature, decrypts the AES key, decrypts the data, and extracts it for you.

**Always store private keys safely!**

---

## 🛡️ Security Note

- Uses AES-256 for data, RSA-4096 for keys.
- Always keep your **private keys** offline or in secure vaults!
```

## ✅ **📌 6️. Example files**

Add a **sample `examples/`** folder with some `.txt` or `.jpg` files so you can test immediately.


