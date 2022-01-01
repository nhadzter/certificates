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
	
	#
	# Root CA Certificate Extensions
	[ root-ca_ext ]
	basicConstraints        = critical, CA:true
	keyUsage                = critical, keyCertSign, cRLSign
	nameConstraints         = critical, @name_constraints
	subjectKeyIdentifier    = hash
	authorityKeyIdentifier  = keyid:always
	issuerAltName           = issuer:copy
	
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
