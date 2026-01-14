# Create Certificates for app-based authentication to Exchange Online

Microsoft EntraID offers the possibility to authenticate via APPs (its like a service-Account) this is explained on their [learn](https://learn.microsoft.com/en-us/entra/identity/hybrid/connect/authenticate-application-id) site.

The App needs to have appropriate API-rights to the relevant apps and also the currect roles.

To authenticate to the Entra-ID you also need a certificate installed on your machine and the corresponding public certificate linked to the app.
The below article shows how to create those certificates on Windows with PowerShell and on macOS with openssl

## WINDOWS

### Create a Self-Sign Certificate on Windows with Powershell

# Create a parameter hashtable for the cert-creation

```powershell
$certParams = @{
    Subject = "CN=ExoManagementRS"  # Replace with your subject
    CertStoreLocation = "Cert:\CurrentUser\My"
    KeyAlgorithm = "RSA"
    KeyLength = 2048
    KeyUsage = "DigitalSignature"
    KeyUsageProperty = "Sign"
    NotAfter = (Get-Date).AddYears(1)  # Validity: 1 year
    Type = "DocumentEncryptionCert"
}

$cert = New-SelfSignedCertificate @certParams

# Export PFX File
$password = 'yourpassword'
$certPassword = ConvertTo-SecureString -String $password -Force -AsPlainText
Export-PfxCertificate -Cert $cert -FilePath "./WindowsAppSert.pfx" -Password $certPassword

# Export CER File
# Path to pfx-File and password
$pfxPath = "./WindowsAppSert.pfx"
$cert = New-Object System.Security.Cryptography.X509Certificates.X509Certificate2($pfxPath, $password)

# Export as .cer-File (DER-codet)
$cerPath = "./WindowsAppSert.cer"
[byte[]] $certBytes = $cert.Export([System.Security.Cryptography.X509Certificates.X509ContentType]::Cert)
[IO.File]::WriteAllBytes($cerPath, $certBytes)

# Get Thumbprint for APP-Auth
$thumbprint = $cert.Thumbprint
Write-Host "Zertifikat Thumbprint: $thumbprint"
```

### Authenticate to EntraID via APP and certificate on Windows

```powershell
# Identify Thumbprint
$cert = Get-ChildItem -Path Cert:\CurrentUser\My | Where-Object { $_.Subject -like "CN=Exo*" }
$thumbprint = $cert.Thumbprint
Write-Host "Certificate Thumbprint: $thumbprint"

# App-ID
$appId = "00000000-aaaa-bbbb-cccc-dddeeefff000"

# Organization domain
$org = 'yourcustomdomain.com'

# Connect to Entra-ID
Connect-ExchangeOnline -AppId $appId -Organization $org -CertificateThumbprint $thumbprint
```

## macOS

On macOS there are no Powershell Core CmdLets for certificates, so we use openssl

### Create a Self-Signed Certificate on macOS with openssl

```powershell
# Generate Key
openssl genrsa -out exo-app.key 2048

# Create Snakeoil-cert
openssl req -new -x509 -key exo-app.key -out exo-app.crt -days 730 -subj "/CN=EXO-AppOnly"

# Create PFX (you will be asked for a password)
openssl pkcs12 -export -out exo-app.pfx -inkey exo-app.key -in exo-app.crt
```

### Authenticate to EntraID via APP and certificate on macOS

```powershell
# Passwort abspeichern
$passwd = Read-Host -prompt 'Enter certificate password'

# Pfad zum Zertifikat in Variable speichern
$pfxPath = Join-Path -Path (Get-Location).Path -ChildPath 'exo-app.pfx'

# Zertifikat in Variable abspeichern
$cert= [System.Security.Cryptography.X509Certificates.X509Certificate2]::new($pfxpath,$passwd,[System.Security.Cryptography.X509Certificates.X509KeyStorageFlags]::Exportable)

# Use AppID
$appId = "00000000-aaaa-bbbb-cccc-dddeeefff000"

# Organization domain
$org = 'yourcustomdomain.com'

Connect-ExchangeOnline -AppId $appid -Certificate $cert -Organization $org
```