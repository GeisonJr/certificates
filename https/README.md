# How to create an HTTPS certificate for localhost domains

This focuses on generating the certificates for loading local virtual hosts hosted on your computer, for development only.


**Do not use self-signed certificates in production !**



## Certificate authority (CA)

Generate `rootCA.pem`, `rootCA.key` & `rootCA.crt`:

	openssl req -x509 -nodes -new -sha256 -days 1024 -newkey rsa:2048 -keyout rootCA.key -out rootCA.pem
	openssl x509 -outform pem -in rootCA.pem -out rootCA.crt


## Domain name certificate

Let's say you have two domains `fake1.local` and `fake2.local` that are hosted on your local machine
for development (using the `hosts` file to point them to `127.0.0.1`).

First, create a file `domains.ext` that lists all your local domains:

	authorityKeyIdentifier=keyid,issuer
	basicConstraints=CA:FALSE
	keyUsage = digitalSignature, nonRepudiation, keyEncipherment, dataEncipherment
	subjectAltName = @alt_names
	[alt_names]
	DNS.1 = localhost
	DNS.2 = fake1.local
	DNS.3 = fake2.local

Generate `certificate.key`, `certificate.csr`, and `certificate.crt`:

	openssl req -new -nodes -newkey rsa:2048 -keyout certificate.key -out certificate.csr
	openssl x509 -req -sha256 -days 1024 -in certificate.csr -CA rootCA.pem -CAkey rootCA.key -CAcreateserial -extfile domains.ext -out certificate.crt

You can now configure your webserver:

	### Apache in XAMPP
		SSLEngine on
		SSLCertificateFile "C:/example/certificate.crt"
		SSLCertificateKeyFile "C:/example/certificate.key"
	
	### LiveServer in Visual Studio Code
		"liveServer.settings.port": 443,
		"liveServer.settings.root": "/",
		"liveServer.settings.CustomBrowser": "chrome",
		"liveServer.settings.host": "localhost",
		"liveServer.settings.https": {
			"enable": true,
			"cert": "C:\\.certificates\\.vscode - liveserver\\certificate.crt",
			"key": "C:\\.certificates\\.vscode - liveserver\\certificate.key",
			"passphrase": "password"
		}


## Trust the local CA

At this point, the site would load with a warning about self-signed certificates.
In order to get a green lock, your new local CA has to be added to the trusted Root Certificate Authorities.


### Windows 10: Chrome, IE11 & Edge

Windows 10 recognizes `.crt` files, so you can right-click on `rootCA.crt` > `Install` to open the import dialog.

Make sure to select "Trusted Root Certification Authorities" and confirm.

You should now get a green lock in Chrome, IE11 and Edge.


### Windows 10: Firefox

There are two ways to get the CA trusted in Firefox.

The simplest is to make Firefox use the Windows trusted Root CAs by going to `about:config`,
and setting `security.enterprise_roots.enabled` to `true`.

The other way is to import the certificate by going
to `about:preferences#privacy` > `Certificats` > `Import` > `rootCA.pem` > `Confirm for websites`.
