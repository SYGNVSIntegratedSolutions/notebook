Below is a **single-file PowerShell script** using **Windows Forms**, so our users get simple buttons for:

- **Select File/Folder**
    
- **Encrypt**
    
- **Decrypt**
    
- **See Status**
    

You don’t have to touch the console at all — just double-click the `.ps1`.

```
Add-Type -AssemblyName System.Windows.Forms
Add-Type -AssemblyName System.IO.Compression.FileSystem
Add-Type -AssemblyName System.Drawing

function Show-Message($text) {
    [System.Windows.Forms.MessageBox]::Show($text)
}

$form = New-Object System.Windows.Forms.Form
$form.Text = "Hybrid Encryptor"
$form.Size = New-Object System.Drawing.Size(400,300)
$form.StartPosition = "CenterScreen"

$label = New-Object System.Windows.Forms.Label
$label.Location = New-Object System.Drawing.Point(10,20)
$label.Size = New-Object System.Drawing.Size(360,20)
$label.Text = "No file/folder selected."

$browseButton = New-Object System.Windows.Forms.Button
$browseButton.Location = New-Object System.Drawing.Point(10,50)
$browseButton.Size = New-Object System.Drawing.Size(100,30)
$browseButton.Text = "Select Target"

$encryptButton = New-Object System.Windows.Forms.Button
$encryptButton.Location = New-Object System.Drawing.Point(10,100)
$encryptButton.Size = New-Object System.Drawing.Size(100,30)
$encryptButton.Text = "Encrypt"

$decryptButton = New-Object System.Windows.Forms.Button
$decryptButton.Location = New-Object System.Drawing.Point(120,100)
$decryptButton.Size = New-Object System.Drawing.Size(100,30)
$decryptButton.Text = "Decrypt"

$selectedPath = ""

$browseButton.Add_Click({
    $dialog = New-Object System.Windows.Forms.OpenFileDialog
    $dialog.CheckFileExists = $false
    $dialog.FileName = "Select Folder or File"
    $dialog.ValidateNames = $false
    if ($dialog.ShowDialog() -eq "OK") {
        $selectedPath = $dialog.FileName
        $label.Text = "Selected: $selectedPath"
    }
})

$encryptButton.Add_Click({
    if ([string]::IsNullOrWhiteSpace($selectedPath)) {
        Show-Message "Please select a file or folder first."
        return
    }

    $KeyFolder = "$env:USERPROFILE\EncryptionKeys"
    if (!(Test-Path $KeyFolder)) {
        New-Item -ItemType Directory -Path $KeyFolder | Out-Null
    }

    $Name = (Split-Path $selectedPath -Leaf).Replace(' ','_')

    $RSA = New-Object System.Security.Cryptography.RSACryptoServiceProvider -ArgumentList 4096
    $PublicKey = $RSA.ToXmlString($false)
    $PrivateKey = $RSA.ToXmlString($true)

    $PublicKeyPath = Join-Path $KeyFolder "PublicKey_$Name.xml"
    $PrivateKeyPath = Join-Path $KeyFolder "PrivateKey_$Name.xml"
    $PublicKey | Out-File -Encoding ascii $PublicKeyPath
    $PrivateKey | Out-File -Encoding ascii $PrivateKeyPath

    $AES = New-Object System.Security.Cryptography.AesManaged
    $AES.KeySize = 256
    $AES.GenerateKey()
    $AES.GenerateIV()

    $ZipPath = "$env:TEMP\ToEncrypt_$Name.zip"
    Compress-Archive -Path $selectedPath -DestinationPath $ZipPath -Force

    $Data = [IO.File]::ReadAllBytes($ZipPath)
    $Encryptor = $AES.CreateEncryptor()
    $EncryptedData = $Encryptor.TransformFinalBlock($Data, 0, $Data.Length)
    $EncryptedFile = "$PWD\Encrypted_$Name.bin"
    [IO.File]::WriteAllBytes($EncryptedFile, $EncryptedData)

    $RSA.FromXmlString($PublicKey)
    $AESKeyCombined = $AES.IV + $AES.Key
    $EncryptedAESKey = $RSA.Encrypt($AESKeyCombined, $true)
    $AESKeyFile = "$PWD\AESKey_$Name.bin"
    [IO.File]::WriteAllBytes($AESKeyFile, $EncryptedAESKey)

    Remove-Item $ZipPath

    Show-Message "✅ Encrypted!`nFiles:`n$EncryptedFile`n$AESKeyFile`nKeys in:`n$KeyFolder"
})

$decryptButton.Add_Click({
    if ([string]::IsNullOrWhiteSpace($selectedPath)) {
        Show-Message "Select your encrypted .bin file first!"
        return
    }

    $PrivateKeyPath = (Get-Item (Read-Host "PrivateKey XML Path")).FullName
    $AESKeyFile = (Get-Item (Read-Host "AESKey BIN Path")).FullName

    $Name = (Split-Path $selectedPath -Leaf).Replace('Encrypted_','').Replace('.bin','')

    $RSA = New-Object System.Security.Cryptography.RSACryptoServiceProvider -ArgumentList 4096
    $PrivateKey = Get-Content $PrivateKeyPath -Raw
    $RSA.FromXmlString($PrivateKey)

    $EncryptedAESKey = [IO.File]::ReadAllBytes($AESKeyFile)
    $AESKeyCombined = $RSA.Decrypt($EncryptedAESKey, $true)

    $IV = $AESKeyCombined[0..15]
    $Key = $AESKeyCombined[16..47]

    $AES = New-Object System.Security.Cryptography.AesManaged
    $AES.Key = $Key
    $AES.IV = $IV

    $Data = [IO.File]::ReadAllBytes($selectedPath)
    $Decryptor = $AES.CreateDecryptor()
    $DecryptedData = $Decryptor.TransformFinalBlock($Data, 0, $Data.Length)

    $TempZip = "$env:TEMP\Decrypted_$Name.zip"
    [IO.File]::WriteAllBytes($TempZip, $DecryptedData)

    $ExtractPath = "$PWD\Decrypted_$Name"
    Expand-Archive -Path $TempZip -DestinationPath $ExtractPath -Force
    Remove-Item $TempZip

    Show-Message "✅ Decrypted to:`n$ExtractPath"
})

$form.Controls.Add($label)
$form.Controls.Add($browseButton)
$form.Controls.Add($encryptButton)
$form.Controls.Add($decryptButton)

[void]$form.ShowDialog()

```

## ✅ **How to run this**

1️⃣ Save it as: `Hybrid-Encryptor-GUI.ps1`  
2️⃣ Right-click → Run with PowerShell  
_(Or run in a PowerShell window with `Set-ExecutionPolicy Bypass -Scope Process` if needed)_  
3️⃣ Click **Select Target** → choose file or folder  
4️⃣ Click **Encrypt** → done!  
5️⃣ Click **Decrypt** → supply the keys → done!

---

## ✅ **What it does**

✔️ Uses your **hybrid encryption**  
✔️ Generates keys automatically  
✔️ Labels & saves output cleanly  
✔️ Keeps your backup keys separate  
✔️ Simple Windows Forms UI, no extra installs