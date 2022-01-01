# Creating CA for Local Environment
This howto explains generating your own Certificate Authority on your local environment.

## Create Root Certificate Authority
Prepare Directories for Root CA

	localhost:~/# mkdir -p root-ca/{certreqs,certs,crl,newcerts,private}

Tracking of issued certificates, their serial numbers and revocations

	localhost:~/# touch root-ca/root-ca.index
	localhost:~/# echo 00 > root-ca/root-ca.crlnum
	localhost:~/# touch root-ca/root-ca.rnd

Create the serial file, to store next incremental serial number
Using random instead of incremental serial numbers is a recommended security practice.

	localhost:~/# openssl rand -hex 16 > root-ca/root-ca.serial

Generate Root CA Key
- if password on key is preferred:
	localhost:~/# openssl genrsa -des3 -out root-ca/private/root-ca.key 4096

- if no password on key is preferred:
	localhost:~/# openssl genrsa -out root-ca/private/root-ca.key 4096

Generate Root CA Certificate
Using the generated Root CA Key create and self sign the root CA:

You can request a certificate and entering values required on the prompt:

	localhost:~/# openssl req -x509 -new -nodes -key root-ca/private/root-ca.key -sha256 -days 3650 -out root-ca/test-root-ca.pem
	You are about to be asked to enter information that will be incorporated into your certificate request.
	What you are about to enter is what is called a Distinguished Name or a DN.
	There are quite a few fields but you can leave some blank
	For some fields there will be a default value,
	If you enter '.', the field will be left blank.
	-----
	Country Name (2 letter code) [AU]:`LE`
	State or Province Name (full name) [Some-State]:`Local Environment`
	Locality Name (eg, city) []:
	Organization Name (eg, company) [Internet Widgits Pty Ltd]:`Local Environment`
	Organizational Unit Name (eg, section) []:
	Common Name (e.g. server FQDN or YOUR name) []:`Local Environment Root CA`
	Email Address []:your@email.com`

	localhost:~/# openssl req -x509 -config root-ca/root-ca.cnf -new -nodes -key root-ca/private/root-ca.key -sha256 -days 3650 -out root-ca/root-ca.pem`

## Create Intermediate Certificate Authority
