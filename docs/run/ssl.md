# Enabling SSL and TLS

## Request Signed Certificate(s)

SSL Certificate requests can be generated using a variety of mechanisms (PuttyGen, OpenSSL, IIS, etc..).
The examples below use OpenSSL to generate the request.

### Generate Private Key

````
openssl genrsa -out <common name>.key 2048
````

This command generates a 2048 bit RSA private key and saves it to the file <common name>.key.

### Generate Certificate Signing Request

````
openssl req -new -sha256 -key <common name>.key -out <common name>.csr
````
````
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [AU]:US
State or Province Name (full name) [Some-State]:Texas
Locality Name (eg, city) []:Houston
Organization Name (eg, company) [Internet Widgits Pty Ltd]:My Organization LLC
Organizational Unit Name (eg, section) []:My Org Unit
Common Name (e.g. server FQDN or YOUR name) []:sample.mycompany.com
Email Address []:

Please enter the following 'extra' attributes
to be sent with your certificate request
A challenge password []:
An optional company name []:
````

This command creates a PKCS#10 certificate request using the RSA Private key created in the previous step.  It prompts for information to build a distinguished name.  Ensure the common name is specified exactly as you will be calling the synapse server components.

### Verify Certificate Signing Request

````
openssl req -noout -text -in sample.csr
````

This command outputs the contents of the Certificate Signing Request (CSR) to the screen.  Verify the CommonName (CN) is exactly how you'd expect to access the synapse server.  This should appear at the end of the "Subject" fields of the request.

Subject: C=US, ST=Texas, L=Houston, O=My Organization LLC, OU=My Org Unit, CN=**sample.mycompany.com**

### Send Certificate Signing Request To Certificate Authority

Submit the CSR to your signing authority.  This can be done via the web, or through a defined process your company uses.   If you created a self-signed certificate, this step is not requried.

## Installing SSL Certificate

### Install any Root and Intermediate Certificates on the server(s)

Your certificate signing authority might provide you with multipe certificates.  Install any root and intermediate certificates you may receive.  This is usually done on a Windows server by logging onto the server as an administrator, double-clicking on each file, and accepting the defaults when prompted.

### Create PKCS#12 Archive (if necessary)

````
openssl pkcs12 -export -out sample.pfx -inkey sample.key -in sample.crt
````

This create a PKCS#12 Archive file (PFX) that contains both the certificate and the cooresponding private key.  This step is only necessary if your signing authority only provides the certificate, and not a PFX archive.  Creating a PFX file greatly simplifies the installation of the certificate onto the server(s).

### Install PKCS#12 Archive on server(s)

Install the PFX file onto the server(s).  On a Windows server, the certificate needs to wind up in the "Local Server / Personal / Certificates" store in the MMC Certificates Console.  This can be done by either double-clicking on the PFX file and then dragging the certificate to the proper location, or by navigating the MMC console and right-clicking on the store, selecting "All Tasks > Import" and use the Certificate Import wizard.

#### Opening the Microsoft Management Console (MMC)

On Windows, the Microsoft Management Console is used to manage Certificates on the server.  

* Search or Run on "MMC".
* Under "File" menu, select "Add/Remove Snap-in".
* Select "Certificates" from the "Available snap-ins" section and click "Add".
* Select "Computer account", "Next", then "Local Computer" and finally "Finish".
* Repeat steps, selectin "My user account" instead of "Computer account".

![MMC Add Snap-In](/img/mmc_add_snapin.png "MMC Add Snap-In")

* Finally, select "Ok" and the console will display both certificate stores in the left panel.

![MMC Certificate Console](/img/mmc_certificates.png "MMC Certificate Console")

## Enable SSL on the Port(s)

### Add Certificate to a Port (Windows)

````
netsh http add sslcert ipport=0.0.0.0:<port> appid={214124cd-d05b-4309-9af9-9caa44b2b74a} certhash=‎<thumbprint, no spaces>
````

Open a CMD prompt as a server Administrator and execute the command above, replacing <port> with the port number you want to install the SSL certificate onto.

* **appid** = This is a GUID to identify the owning application.  Any valid GUID can be used here, as it is only used to identify the binding later.
* **certhash** = This value is the certificates "thumbprint".  It can be found by opening the MMC Certificate Console, double-clicking on the certificate, selecting the "Details" tab, then searching for the "Thumbprint" field.  You **MUST** remove all spaces between the hexadecimal numbers.  For example, the thumbprint "a9 09 50 2d d8 2a e4 14 33 e6 f8 38 86 b0 0d 42 77 a3 2a 7b" should be specified as "a909502dd82ae41433e6f83886b00d4277a32a7b".

![Certificate Thumbprint](/img/cert_thumbprint.png "Certificate Thumbprint")

### Other Commands (Windows)

Below are other useful command to manage SSL certificates on a Windows server.

#### View All SSL Certificates

````
netsh http show sslcert
````

Shows all SSL certiicates installed on the server.

#### View SSL Certificates On A Port

````
netsh http show sslcert ipport=0.0.0.0:<port>
````

Shows the SSL certificate installed on the specified port.

#### Remove SSL Certificate

````
netsh http delete sslcert ipport=0.0.0.0:<port>
````

Removes the SSL Certificate installed on the specified port.

### Modify Synapse Server Config File

Now that the port is SSL enabled, you must modify the Synapse server config files to use SSL.   Below are the relevant fields from the server config files to accomplish this.

#### Controller Config File

````yaml
Service:
  Name: Synapse.Controller
  DisplayName: Synapse Controller
  Role: Controller
WebApi:
  Host: sample.mycompany.com
  Port: 20000
  IsSecure: true
  Authentication:
    Scheme: Ntlm
    Config: 
Signature:
  KeyUri: 
  KeyContainerName: 
  CspProviderFlags: NoFlags
Controller:
  NodeUrl: https://sample.mycompany.com:20001/synapse/node
  SignPlan: false
  Assemblies: 
  Dal:
    Type: Synapse.Controller.Dal.FileSystem:FileSystemDal
    LdapRoot: 
    Config:
      PlanFolderPath: Plans
      HistoryFolderPath: History
      ProcessPlansOnSingleton: false
      ProcessActionsOnSingleton: true
      Security:
        FilePath: Security
        IsRequired: false
        ValidateSignature: false
        SignaturePublicKeyFile: 
        GlobalExternalGroupsCsv: Everyone
````

* **WebApi > Host** : Should match the CommonName used to create the certificate
* **WebApi > IsSecure** : Should be set to "true" to expect inbound SSL packets.
* **Controller > NodeUrl** : If Node is running under SSL, be sure the url uses HTTPS


#### Node Config File

````yaml
Service:
  Name: Synapse.Node
  DisplayName: Synapse Node
  Role: Node
WebApi:
  Host: sample.mycompany.com
  Port: 20001
  IsSecure: false
  Authentication:
    Scheme: Anonymous
    Config: 
Signature:
  KeyUri: 
  KeyContainerName: 
  CspProviderFlags: NoFlags
Controller: 
Node:
  MaxServerThreads: 0
  AuditLogRootPath: .\Logs
  Log4NetConversionPattern: '%d{ISO8601}|%-5p|(%t)|%m%n'
  SerializeResultPlan: true
  ValidatePlanSignature: false
  ControllerUrl: https://sample.mycompany.com:20000/synapse/execute
````

* **WebApi > Host** : Should match the CommonName used to create the certificate
* **WebApi > IsSecure** : Should be set to "true" to expect inbound SSL packets.
* **Node > ControllerUrl** : If this value is specified and the Controller is running under SSL, ensure HTTPS is specified at the start of the Url.