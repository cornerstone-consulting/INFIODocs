# Steps to Trust the Self-Signed Certificate

## 1. Download and install Microsoft SDK in machine
```powershell
https://developer.microsoft.com/en-us/windows/downloads/sdk-archive/
```

## 2. Create a Self-Signed Certificate
Use the following PowerShell command to create a self-signed certificate:

```powershell
New-SelfSignedCertificate -Type Custom -Subject "CN=Cornerstone" -KeyUsage DigitalSignature -FriendlyName "AakashLimbad" -CertStoreLocation "Cert:\CurrentUser\My"
```

## 3. View the Thumbprint of Your Certificate
To use the certificate for signing, you need its thumbprint. Follow these steps:

Open PowerShell and run the following command:
```powershell
Get-ChildItem -Path Cert:\CurrentUser\My
```
This will display a list of certificates under the current user. Look for your certificate by its Subject name (e.g., CN=Cornerstone).
Note down the Thumbprint value associated with your certificate.

Example Output:

```plaintext
Thumbprint                                Subject
----------                                -------
AE2B75130F72EDCC6574E98B21F19D4D34FA86A4  CN=Cornerstone
```

## 4. Export the Self-Signed Certificate
1. Open **CertMgr** by typing `certmgr.msc` in the Start menu and pressing Enter.
2. Navigate to **Personal > Certificates** and locate your **Cornerstone** certificate.
3. Right-click on the certificate and select **Export**.
4. Choose to **export the certificate** (do not export the private key).
5. Save the certificate as a `.cer` file (e.g., `Cornerstone.cer`).

## 5. Add the Certificate to the Trusted Root Certification Authorities
1. In **CertMgr**, go to **Trusted Root Certification Authorities > Certificates**.
2. Right-click in the empty area and select **Import**.
3. Select the `.cer` file you exported in the previous step.
4. Follow the prompts to install the certificate into the **Trusted Root Certification Authorities** store.

## 6. Attach the Self-Signed Certificate to the EXE File
After creating the certificate, attach it to your EXE file using the signtool command:

```powershell
signtool sign /sha1 <thumbprint_of_certificate> /fd sha256 /tr http://timestamp.digicert.com /td sha256 "<path_to_exe_file>"
```
Example:
```powershell
signtool sign /sha1 AE2B75130F72EDCC6574E98B21F19D4D34FA86A4 /fd sha256 /tr http://timestamp.digicert.com /td sha256 "C:\infio-plugin\dist\infio-plugin-v1.0.exe"
```
- Replace <thumbprint_of_certificate> with the thumbprint of your certificate (e.g., AE2B75130F72EDCC6574E98B21F19D4D34FA86A4) and <path_to_exe_file> with the path to your EXE file (e.g., C:\infio-plugin\dist\infio-plugin-v1.0.exe).

## 7. Verify the Signature Again
After the certificate has been added to the trusted root store, you can verify the EXE using the `signtool`:

```powershell
signtool verify /pa "C:\infio-plugin\dist\infio-plugin-v1.0.exe"
```
## 8. Check the Digital Signature via File Properties

1. Right-click on the EXE file (e.g., infio-plugin-v1.0.exe).
2. Select Properties from the context menu.
3. Go to the Digital Signatures tab.
4. If the file is signed, you'll see details about the certificate used to sign the EXE, such as the signer's name (e.g., "Cornerstone"), the certificate issuer, and timestamp information.
  If there's no Digital Signatures tab, the EXE is not signed.
