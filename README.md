# CodeSigning-DigitalSignatures
Creation of Digital Signatures also known as Code Signing for Windows PE.

Required files:

http://go.microsoft.com/fwlink/p/?LinkID=2033686 # Windows 10 SDK ISO

https://github.com/ColorCop/ColorCop/raw/master/codesign/Office%202000%20Tool%20PVK%20Digital%20Certificate%20Files%20Importer/pvkimprt.exe # pvkimprt.exe

1. Download Windows 10 SDK ISO image file

2. Mount ISO image file to virtual drive or extract all files from ISO image file.

3. Install Windows 10 SDK by running "WinSDKSetup.exe"

4. Search for following files "cert2spc.exe", "makecert.exe", "signtool.exe" in "%PROGRAMFILES(x86)%\Windows Kits\10\bin\10.0.17763.0\x86" for 32-bit and for 64-bit in "%PROGRAMFILES(x86)%\Windows Kits\10\bin\10.0.17763.0\x64".
Make sure it's the latest version number at end of directory string, your version number might be higher (newer version).

5. Copy all files "pvkimprt.exe", "cert2spc.exe", "makecert.exe", "signtool.exe" to same folder with "MakeCert.bat" and "SignPE.bat".

6. Run "MakeCert.bat" and type in a password (Private Key for certificate) eg. YourPasswordOfChoice.

7. Edit "SignPE.bat" so the password, you entered when running "MakeCert.bat" will be the same, edit following "SET PASSWORD=YourPasswordOfChoice" (without quotations)

8. Launch a command prompt and change directory to the directory containing your "cert2spc.exe", "makecert.exe", "signtool.exe", "pvkimprt.exe", "MakeCert.bat", "SignPE.bat".

FilenameToDigitallySign.exe = Your File to Digitally Sign, so it will have a digital signature after this procedure is done.

9. Type in following: SignPE.bat FilenameToDigitallySign.exe

Generating a certificate:

With all of the required tools available, we can now generate our private key and code signing certificate.

Open a command prompt (type cmd in the Run or “Start Search” box, or use Start->All Programs/Programs->Accessories->Command Prompt), navigate to the directory containing the tools, and execute them in the following sequence (with all spaces and other punctuation as shown):
makecert.exe MyCertificateFile.cer -r -n "CN= MyCompanyName " -$ individual -sv MyPrivateKeyFile.pkv -pe -eku 1.3.6.1.5.5.7.3.3 

This first step will produce a “self-signed” certificate. Substitute “MyCertificateFile.cer” with the name you want to give to the generated certificate (the name you choose is just a label, the value of the text is inconsequential to the outcome of later stages – in all instances it makes sense to retain the suggested extensions, though, for clarity); “MyCompanyName” with an appropriate value for your “Publisher” identity; “MyPrivateKeyFile.pkv” with a name to use for your private key store file.

For further information about the meaning of the command line switches added here, use the in-built help “makecert.exe /?”.

The string after the “eku” switch indicates that the file to create will be a “code signing” certificate.

When you run the tool, you will be prompted to enter a “Private Key Password” - you will need to provide this again in step 3, and each time you sign a file, so make a note of the password you choose to use here.
cert2spc.exe MyCertificateFile.cer MyCertificateFile.spc 

Replace “MyCertificateFile.cer” with the name you chose to give your generated self-signed certificate from step 1, and choose a matching name (again, just for clarity – it doesn't have to match) to replace “MyCertificateFile.spc”. This will create a “Software Publisher Certificate” (SPC) from the “MyCertificateFile.cer” file – a certificate equivalent to one that would be issued from a Certificate Authority.
pvkimprt.exe -pfx MyCertificateFile.spc MyPrivateKeyFile.pkv 

Replace “MyCertificateFile.spc”with the name you chose for your Software Publisher Certificate (.spc file) in step 2, and “MyPrivateKeyFile.pkv” with a name for the file to be created by Pvkimprt – a “Personal Information Exchange” (PFX) file – this is what will be used to for code signing.
Pkimprt is a GUI wizard. When you run it you'll first be asked to enter your “Private Key Password” - the password you selected for step 1.

None of the default wizard values need to be changed, so click through each successive form by clicking Next (x3), then on the “Password” form, enter your Private Key Password twice more, and click Next again.

On the “File to Export” form, click the Browse button, navigate to the directory containing the signing tools, and type a name for the PFX file – e.g., “MyPFXFile.pfx”, then click Next.
On the final wizard form, “Completing the Certificate Export Wizard”, click the Finish button to complete the procedure.

As a result of these steps, you should now have four new files labelled something like:

MyPrivateKeyFile.pkv
MyCertificateFile.cer
MyCertificateFile.spc
MyPFXFile.pfx


Code signing

To use our newly generated certificate, we need signtool.exe and a target file. Suppose we have a binary executable named “MyProgram.exe” - the syntax for signing this file would be:
signtool.exe sign /f MyPFXFile.pfx /p MyPrivateKeyPassword /v /t http://timestamp.verisign.com/scripts/timstamp.dll MyProgram.exe 

- where “MyPrivateKeyPassword” is the password you chose in step 1 (note also that “timstamp.dll” is not a misspelling).

Automating the process

To make things easier and avoid having to re-type long commands in the case of a syntax error, it makes sense to string the tool commands together in a batch file. See the listing below for two batch files based upon the text of the steps listed above.

Generating a private key, self-signed certificate, and PFX file:

MakeCert.bat:
======================================================================================================================================
@ECHO OFF

makecert.exe MyCertificateFile.cer -r -n "CN= MyCompanyName " -$ individual -sv MyPrivateKeyFile.pkv -pe -eku 1.3.6.1.5.5.7.3.3

cert2spc.exe MyCertificateFile.cer MyCertificateFile.spc

pvkimprt.exe -pfx myCertificateFile.spc myPrivateKeyFile.pkv 

pause<nul
=======================================================================================================================================
Signing a binary file:

SignBinary.bat:
=======================================================================================================================================
@ECHO OFF

signtool.exe sign /f MyPFXFile.pfx /p MyPrivateKeyPassword /v /t http://timestamp.verisign.com/scripts/timstamp.dll %1

pause<nul
=======================================================================================================================================

To use the second batch file with a named binary, launch it with a parameter – the name of the target binary file, e.g., “SignBinary.bat MyProgram.exe”.
