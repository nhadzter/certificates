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

	localhost:~/# openssl req -x509 -config root-ca/root-ca.cnf -new -nodes -key root-ca/private/root-ca.key -days 3650 -out root-ca/root-ca.pem`

Or, create an openssl configuration and define the values:

	localhost:~/# cat root-ca/root-ca.cnf 
	#
	# OpenSSL configuration for the Root Certification Authority.
	#
	RANDFILE                = ./root-ca/root-ca.rnd
	
	# Default Certification Authority
	[ ca ]
	default_ca              = root_ca
	
	#
	# Root Certification Authority
	[ root_ca ]
	dir                     = ./root-ca
	certs                   = $dir/certs
	serial                  = $dir/root-ca.serial
	database                = $dir/root-ca.index
	new_certs_dir           = $dir/newcerts
	certificate             = $dir/root-ca.cert.pem
	private_key             = $dir/private/root-ca.key
	default_days            = 1826 # 5 years
	crl                     = $dir/root-ca.crl
	crl_dir                 = $dir/crl
	crlnumber               = $dir/root-ca.crlnum
	name_opt                = multiline, align
	cert_opt                = no_pubkey
	copy_extensions         = copy
	crl_extensions          = crl_ext
	default_crl_days        = 180
	default_md              = sha384
	preserve                = no
	email_in_dn             = no
	policy                  = policy
	unique_subject          = no
	
	# Distinguished Name Policy for CAs
	[ policy ]
	countryName             = optional
	stateOrProvinceName     = optional
	localityName            = optional
	organizationName        = supplied
	organizationalUnitName  = optional
	commonName              = supplied
	
	# Root CA Request Options
	[ req ]
	dir                     = ./root-ca
	default_bits            = 4096
	encrypt_key             = yes
	default_md              = sha256
	string_mask             = utf8only
	utf8                    = yes
	prompt                  = no
	distinguished_name      = distinguished_name
	
	# Distinguished Name (DN)
	[ distinguished_name ]
	organizationName        = Local Environment
	commonName              = Local Environment Root CA
	
	# Root CA Certificate Extensions
	[ root-ca_ext ]
	basicConstraints        = critical, CA:true
	keyUsage                = critical, keyCertSign, cRLSign
	nameConstraints         = critical, @name_constraints
	subjectKeyIdentifier    = hash
	authorityKeyIdentifier  = keyid:always
	issuerAltName           = issuer:copy

	# Intermediate CA Certificate Extensions
	[ intermediate-ca_ext ]
	basicConstraints        = critical, CA:true, pathlen:0
	keyUsage                = critical, keyCertSign, cRLSign
	subjectKeyIdentifier    = hash
	authorityKeyIdentifier  = keyid:always
	issuerAltName           = issuer:copy
	crlDistributionPoints   = crl_dist
	
	# CRL Certificate Extensions
	[ crl_ext ]
	authorityKeyIdentifier  = keyid:always
	issuerAltName           = issuer:copy
	
	# EOF

and request for the certificate:

	localhost:~/# openssl req -x509 -config root-ca/root-ca.cnf -new -nodes -key root-ca/private/root-ca.key -days 3650 -out root-ca/root-ca.pem

verify certificate:

	localhost:~/# openssl x509 -in root-ca/test-config-root-ca.pem -subject -noout
	subject=O = Local Environment, CN = Local Environment Root CA

	localhost:~/# openssl rsa -modulus -noout -in root-ca/private/root-ca.key | openssl md5
	(stdin)= 3c4ea5d857189c64d946ef1fb7e5f177

	localhost:~/# openssl x509 -modulus -noout -in root-ca/root-ca.pem | openssl md5
	(stdin)= 3c4ea5d857189c64d946ef1fb7e5f177

## Create Intermediate Certificate Authority

This is an optional step, certificates can already be signed using the Root CA.

Prepare Directories for Intermediate CA

	localhost:~/# mkdir -p intermediate-ca/{certreqs,certs,crl,newcerts,private}

Tracking of issued certificates, their serial numbers and revocations

	localhost:~/# touch intermediate-ca/intermediate-ca.index
	localhost:~/# echo 00 > intermediate-ca/intermediate-ca.crlnum
	localhost:~/# touch intermediate-ca/intermediate-ca.rnd

Create the serial file, to store next incremental serial number
Using random instead of incremental serial numbers is a recommended security practice.

	localhost:~/# openssl rand -hex 16 > intermediate-ca/intermediate-ca.serial

Generate Intermediate CA Key

- if password on key is preferred:

	localhost:~/# openssl genrsa -des3 -out intermediate-ca/private/intermediate-ca.key 4096

- if no password on key is preferred:

	localhost:~/# openssl genrsa -out intermediate-ca/private/intermediate-ca.key 4096

Generate CSR for Intermediate CA
	localhost:~/# openssl req -config intermediate-ca/intermediate-ca.cnf -new -key intermediate-ca/private/intermediate-ca.key -out intermediate-ca/intermediate-ca.csr

Verify CSR
	localhost:~/# openssl req -verify -in intermediate-ca/intermediate-ca.csr -text

Sign the CSR using Root CA

	localhost:~/# openssl ca -config root-ca/root-ca.cnf -in intermediate-ca/intermediate-ca.csr -out intermediate-ca/intermediate-ca.pem -extensions intermediate-ca_ext 
	Using configuration from root-ca/root-ca.cnf
	Check that the request matches the signature
	Signature ok
	Certificate Details:
	Certificate:
	    Data:
	        Version: 3 (0x2)
	        Serial Number:
	            9a:3e:1e:aa:1d:b7:68:e0:39:5b:cd:a2:bf:13:88:c5
	        Issuer:
	            organizationName          = Local Environment
	            commonName                = Local Environment Root CA
	        Validity
	            Not Before: Jan  1 16:13:12 2022 GMT
	            Not After : Jan  1 16:13:12 2027 GMT
	        Subject:
	            organizationName          = Local Environment
	            commonName                = Local Environment Intermediate CA
	        X509v3 extensions:
	            X509v3 Basic Constraints: critical
	                CA:TRUE, pathlen:0
	            X509v3 Key Usage: critical
	                Certificate Sign, CRL Sign
	            X509v3 Subject Key Identifier: 
	                6D:CB:96:1F:03:29:2A:6D:61:A1:C7:AF:A5:3C:D2:6D:35:D7:B4:4D
	            X509v3 Authority Key Identifier: 
	                0.
	            X509v3 Issuer Alternative Name: 
	                <EMPTY>
	
	Certificate is to be certified until Jan  1 16:13:12 2027 GMT (1826 days)
	Sign the certificate? [y/n]:
	
	1 out of 1 certificate requests certified, commit? [y/n]y
	Write out database with 1 new entries
	Data Base Updated

Verify the Intermediate CA

	localhost:~/# openssl x509 -in intermediate-ca/intermediate-ca.pem -subject -noout
	subject=O = Local Environment, CN = Local Environment Intermediate CA
	
	localhost:~/# openssl x509 -in intermediate-ca/intermediate-ca.pem -issuer -noout
	issuer=O = Local Environment, CN = Local Environment Root CA

## Create Self-Signed Certificate with Local Environment CA
In this howto, we will generate certificate for wildcard domain: *.nhadzter.local 

Prepare directories for self-signed certificate

	localhost:~/# mkdir -p nhadzter.local/{private,public,request}

Generate Key and CSR for self-signed certificates:

	localhost:~/# openssl req -new -nodes -days 365 -keyout nhadzter.local/private/nhadzter.local.key -out nhadzter.local/request/nhadzter.local.csr
	Ignoring -days; not generating a certificate
	Generating a RSA private key
	.......................................................+++++
	..............+++++
	writing new private key to 'nhadzter.local/private/nhadzter.local.key'
	-----
	You are about to be asked to enter information that will be incorporated
	into your certificate request.
	What you are about to enter is what is called a Distinguished Name or a DN.
	There are quite a few fields but you can leave some blank
	For some fields there will be a default value,
	If you enter '.', the field will be left blank.
	-----
	Country Name (2 letter code) [AU]:LE
	State or Province Name (full name) [Some-State]:
	Locality Name (eg, city) []:
	Organization Name (eg, company) [Internet Widgits Pty Ltd]:Nhadzter Local
	Organizational Unit Name (eg, section) []:
	Common Name (e.g. server FQDN or YOUR name) []:*.nhadzter.local
	Email Address []:me@nhadzter.local
	
	Please enter the following 'extra' attributes
	to be sent with your certificate request
	A challenge password []:
	An optional company name []:

Sign the CSR using Intermediate CA

	localhost:~/# openssl ca -config intermediate-ca/intermediate-ca.cnf -in nhadzter.local/request/nhadzter.local.csr -out nhadzter.local/public/nhadzter.local.pem
	Using configuration from intermediate-ca/intermediate-ca.cnf
	Check that the request matches the signature
	Signature ok
	Certificate Details:
	Certificate:
	    Data:
	        Version: 1 (0x0)
	        Serial Number:
	            10:e7:b8:c4:29:3e:b9:dd:9b:fe:0c:cf:a8:44:af:42
	        Issuer:
	            organizationName          = Local Environment
	            commonName                = Local Environment Intermediate CA
	        Validity
	            Not Before: Jan  1 17:54:33 2022 GMT
	            Not After : Feb  1 17:54:33 2023 GMT
	        Subject:
	            countryName               = LE
	            stateOrProvinceName       = Some-State
	            organizationName          = Nhadzter Local
	            commonName                = *.nhadzter.local
	Certificate is to be certified until Feb  1 17:54:33 2023 GMT (396 days)
	Sign the certificate? [y/n]:y
	
	1 out of 1 certificate requests certified, commit? [y/n]y
	Write out database with 1 new entries
	Data Base Updated

Verify certificate:

	localhost:~/# openssl x509 -in nhadzter.local/public/nhadzter.local.pem -subject -noout
	subject=C = LE, ST = Some-State, O = Nhadzter Local, CN = *.nhadzter.local
	
	localhost:~/# openssl x509 -in nhadzter.local/public/nhadzter.local.pem -issuer -noout
	issuer=O = Local Environment, CN = Local Environment Intermediate CA

## References
- https://roll.urown.net/ca/index.html
