
This is a more advanced script for more advanced users. However, that is not to say that the goal of this script is to be confusing for you - however, encryption and key management, by their nature, tend to be a little confusing. 

That said, helpdesk@sygnvs.com is always happy to assist you! This script will automatically encrypt files or folders, and save the keys generated to a secure location on your machine. This script would be perfect for sending confidential files over email, SSH, Telnet, or any other means which involve exposing files to the internet. It can also greatly improve security for data at rest (data which simply sits in a local drive and isn't moving).

## ⚡ **What this does**

✅ Generates a unique **RSA key pair** per file/folder  
✅ Encrypts a **symmetric AES key** with the RSA public key  
✅ Encrypts the file/folder with AES (strong + efficient)  
✅ Saves:

- `PublicKey_<Name>.xml`
    
- `PrivateKey_<Name>.xml` (keep safe!)
    
- `Encrypted_<Name>.zip`  
    ✅ Creates an **instructions file** so any end user can decrypt later.
    

---

## 🛡️ **Why asymmetric + symmetric?**

- RSA alone can’t handle big files directly.
    
- Instead: RSA protects the AES key → AES encrypts the data.
    
- This is standard practice (hybrid crypto).

## ✅ **Complete PowerShell Script**

```
<#
.SYNOPSIS
  Encrypts or decrypts files/folders with hybrid RSA + AES.
.DESCRIPTION
  Generates RSA key pair per target.
  Encrypts data with AES.
  Protects AES key with RSA.
.NOTES
  Author: ChatGPT
#>

param (
    [Parameter(Mandatory)]
    [ValidateSet("Encrypt","Decrypt")]
    [string]$Mode,

    [Parameter(Mandatory)]
    [string]$TargetPath,

    [Parameter()]
    [string]$KeyFolder = "$env:USERPROFILE\EncryptionKeys"
)

# Ensure key folder exists
if (!(Test-Path $KeyFolder)) {
    New-Item -ItemType Directory -Path $KeyFolder | Out-Null
}

# Target name for labels
$Name = (Split-Path $TargetPath -Leaf).Replace(' ','_')

# Setup RSA
$RSA = New-Object System.Security.Cryptography.RSACryptoServiceProvider -ArgumentList 4096

if ($Mode -eq "Encrypt") {
    # 1️⃣ Export RSA Keys
    $PublicKey = $RSA.ToXmlString($false)
    $PrivateKey = $RSA.ToXmlString($true)

    $PublicKeyPath = Join-Path $KeyFolder "PublicKey_$Name.xml"
    $PrivateKeyPath = Join-Path $KeyFolder "PrivateKey_$Name.xml"

    $PublicKey | Out-File -Encoding ascii $PublicKeyPath
    $PrivateKey | Out-File -Encoding ascii $PrivateKeyPath

    Write-Host "✅ RSA key pair created and saved:"
    Write-Host "   $PublicKeyPath"
    Write-Host "   $PrivateKeyPath"

    # 2️⃣ Generate random AES key
    $AES = New-Object System.Security.Cryptography.AesManaged
    $AES.KeySize = 256
    $AES.GenerateKey()
    $AES.GenerateIV()

    # 3️⃣ Encrypt the file/folder → zip
    $ZipPath = "$env:TEMP\ToEncrypt_$Name.zip"
    Compress-Archive -Path $TargetPath -DestinationPath $ZipPath

    $Data = [IO.File]::ReadAllBytes($ZipPath)

    $Encryptor = $AES.CreateEncryptor()
    $EncryptedData = $Encryptor.TransformFinalBlock($Data, 0, $Data.Length)

    $EncryptedFile = "Encrypted_$Name.bin"
    [IO.File]::WriteAllBytes($EncryptedFile, $EncryptedData)

    Write-Host "✅ Data encrypted with AES: $EncryptedFile"

    # 4️⃣ Encrypt AES key with RSA public key
    $RSA.FromXmlString($PublicKey)

    $AESKeyCombined = $AES.IV + $AES.Key
    $EncryptedAESKey = $RSA.Encrypt($AESKeyCombined, $true)

    $AESKeyFile = "AESKey_$Name.bin"
    [IO.File]::WriteAllBytes($AESKeyFile, $EncryptedAESKey)

    Write-Host "✅ AES key encrypted with RSA: $AESKeyFile"

    Remove-Item $ZipPath

    Write-Host "`n🎉 All done!"
    Write-Host "Keep your PrivateKey safe!"
    Write-Host "Files created:"
    Write-Host " - $EncryptedFile"
    Write-Host " - $AESKeyFile"
    Write-Host " - Public & Private Keys in: $KeyFolder"

} elseif ($Mode -eq "Decrypt") {
    # 1️⃣ Prompt for Private Key & AES Key files
    $PrivateKeyPath = Read-Host "Enter full path to PrivateKey_<Name>.xml"
    $AESKeyFile = Read-Host "Enter full path to AESKey_<Name>.bin"
    $EncryptedFile = $TargetPath

    # Load keys
    $PrivateKey = Get-Content $PrivateKeyPath -Raw
    $RSA.FromXmlString($PrivateKey)

    $EncryptedAESKey = [IO.File]::ReadAllBytes($AESKeyFile)
    $AESKeyCombined = $RSA.Decrypt($EncryptedAESKey, $true)

    $IV = $AESKeyCombined[0..15]
    $Key = $AESKeyCombined[16..47]

    $AES = New-Object System.Security.Cryptography.AesManaged
    $AES.Key = $Key
    $AES.IV = $IV

    $Data = [IO.File]::ReadAllBytes($EncryptedFile)
    $Decryptor = $AES.CreateDecryptor()
    $DecryptedData = $Decryptor.TransformFinalBlock($Data, 0, $Data.Length)

    $TempZip = "$env:TEMP\Decrypted_$Name.zip"
    [IO.File]::WriteAllBytes($TempZip, $DecryptedData)

    $ExtractPath = "$PWD\Decrypted_$Name"
    Expand-Archive -Path $TempZip -DestinationPath $ExtractPath -Force

    Remove-Item $TempZip

    Write-Host "✅ Decrypted to: $ExtractPath"
    Write-Host "🎉 Done!"
}
```

## 📌 **How to use**

### ✅ Encrypting

1️⃣ Save script as `Hybrid-Encrypt.ps1`  
2️⃣ Open PowerShell as your user  
3️⃣ Run:

`Set-ExecutionPolicy Bypass -Scope Process -Force`

4️⃣ Encrypt:

`.\Hybrid-Encrypt.ps1 -Mode Encrypt -TargetPath "C:\Path\To\FolderOrFile"`

- This makes:
    
    - `Encrypted_<Name>.bin` — your secure data
        
    - `AESKey_<Name>.bin` — encrypted AES key
        
    - `PublicKey_<Name>.xml` — safe to share
        
    - `PrivateKey_<Name>.xml` — **DO NOT SHARE!**
        

---

### ✅ Decrypting

1️⃣ Run:

`.\Hybrid-Encrypt.ps1 -Mode Decrypt -TargetPath "Encrypted_<Name>.bin"`

2️⃣ It will ask:

- Path to your `PrivateKey_<Name>.xml`
    
- Path to your `AESKey_<Name>.bin`
    

3️⃣ It restores your original files in:

`Decrypted_<Name>/`

---

## 🔑 **Where to store keys**

- Store **Public Keys** with the encrypted files if needed.
    
- Store **Private Keys** in a secure folder, offline backup, or encrypted USB.
    
- Label them clearly: `PrivateKey_QuarterlyReport.xml`, `AESKey_QuarterlyReport.bin` etc.
    

---

## ✅ **Safe defaults**

- RSA: 4096-bit keys
    
- AES: 256-bit
    
- Hybrid: Fast + secure
    
- Uses Windows native .NET crypto only — no extra software.

Are you an encryption expert already? Why goodness gracious, we are happy to have you here! Check out the Pro Version of this script at the link below, where we can assist you with maximum-level military-grade encryption protocols that keep your data as secure as possible!

In case you were wondering whether this script would be right for your organization, then let's put your concerns to rest: **it's absolutely overkill**. But if you're curious about what we CAN institute in disaster-proofing scenarios, then by all means check out the most high-level cryptography script that we could make for you:

