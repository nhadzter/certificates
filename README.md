# Creating CA for Local Environment
This howto explains generating your own Certificate Authority on your local environment.

## Create Root Certificate Authority
Prepare Directories for Root CA

`localhost:~/# mkdir -p root-ca/{certreqs,certs,crl,newcerts,private}`

Tracking of issued certificates, their serial numbers and revocations

`localhost:~/# touch root-ca/root-ca.index`

`localhost:~/# echo 00 > root-ca/root-ca.crlnum`

Create the serial file, to store next incremental serial number
Using random instead of incremental serial numbers is a recommended security practice.

`localhost:~/# openssl rand -hex 16 > root-ca/root-ca.serial`

Generate Root CA Key
- if password on key is preferred:

`localhost:~/# openssl genrsa -des3 -out root-ca/private/root-ca.key 4096`
- if no password on key is preferred:

`localhost:~/openssl genrsa -out root-ca/private/root-ca.key 4096`

Generate Root CA Certificate
Using the generated Root CA Key create and self sign the root CA:

`localhost:~/openssl req -x509 -config root-ca/root-ca.cnf -new -nodes -key root-ca/private/root-ca.key -sha256 -days 3650 -out root-ca/root-ca.pem`

## Create Intermediate Certificate Authority
