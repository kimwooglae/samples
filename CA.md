폴더 초기화
=========
~~~~
mkdir certs crl newcerts private
touch index.txt
echo 1000 > serial
echo 01 > crlnumber
~~~~

ROOT CA 인증서
=============
configuration file 생성
----------------------
myca.cnf 파일 생성
~~~~
[ ca ]
default_ca              = CA_default

[ CA_default ]

dir                     = /Users/wlkim/inswaveca/20161226_02
certs                   = $dir/certs
crl_dir                 = $dir/crl
new_certs_dir           = $dir/newcerts
database                = $dir/index.txt
serial                  = $dir/serial
RANDFILE                = $dir/private/.rand

private_key             = $dir/private/inswaveCA.key
certificate             = $dir/certs/inswaveCA.crt

crlnumber               = $dir/crlnumber
crl                     = $dir/crl/crl.pem
crl_extensions          = crl_ext
default_crl_days        = 30

default_md              = sha256

name_opt                = ca_default
cert_opt                = ca_default
default_days            = 7295

presere                 = no
policy                  = policy_any

#email_in_dn             = no
#copy_extensions         = none
#x509_extensions         = extensions_section

[ policy_any ]
countryName             = supplied
stateOrProvinceName     = optional
organizationName        = optional
organizationalUnitName  = optional
commonName              = supplied
emailAddress            = optional

[ req ]
# Options for the `req` tool (`man req`).
default_bits        = 2048
distinguished_name  = req_distinguished_name
string_mask         = utf8only

# SHA-1 is deprecated, so use SHA-2 instead.
default_md          = sha256

# Extension to add when the -x509 option is used.
x509_extensions     = v3_ca

[ req_distinguished_name ]
countryName                     = KR
stateOrProvinceName             = Seoul
localityName                    = Seoul
0.organizationName              = Inswave Systems CO., LTD.
organizationalUnitName          = R&D Division
commonName                      = Inswave CA
emailAddress                    = webmaster@inswave.com

# Optionally, specify some defaults.
countryName_default             = KR
stateOrProvinceName_default     = Seoul
localityName_default            = Seoul
0.organizationName_default      = Inswave Systems CO., LTD.
organizationalUnitName_default  = R&D Division
commonName_default              = Inswave CA
emailAddress_default            = webmaster@inswave.com

[ v3_ca ]
# Extensions for a typical CA (`man x509v3_config`).
subjectKeyIdentifier = hash
authorityKeyIdentifier = keyid:always,issuer
basicConstraints = critical, CA:true
keyUsage = critical, digitalSignature, cRLSign, keyCertSign
crlDistributionPoints=URI:http://127.0.0.1:3103/crl/crl.pem


[ server_cert ]
# Extensions for server certificates (`man x509v3_config`).
basicConstraints = CA:FALSE
nsCertType = server
nsComment = "OpenSSL Generated Server Certificate"
subjectKeyIdentifier = hash
authorityKeyIdentifier = keyid,issuer:always
keyUsage = critical, digitalSignature, keyEncipherment
extendedKeyUsage = serverAuth

[ crl_ext ]
authorityKeyIdentifier=keyid:always
~~~~



Create the root key
-------------------

~~~~
openssl genrsa -aes256 -out private/inswaveCA.key 2048
~~~~

> Enter pass phrase for inswaveCA.key: inswave

Create the root certificate
---------------------------

~~~~
openssl req -config myca.cnf -key private/inswaveCA.key -new -x509 -days 7295 -sha256 -extensions v3_ca -out certs/inswaveCA.crt
~~~~

Verify the root certificate
---------------------------

~~~~
openssl x509 -noout -text -in certs/inswaveCA.crt
~~~~

127.0.0.1 인증서
===============
configuration file 파일 생성
---------------------
myserver.cnf
~~~~
[ ca ]
default_ca              = CA_default

[ CA_default ]

dir                     = /Users/wlkim/inswaveca/20161226_02
certs                   = $dir/certs
crl_dir                 = $dir/crl
new_certs_dir           = $dir/newcerts
database                = $dir/index.txt
serial                  = $dir/serial
RANDFILE                = $dir/private/.rand

private_key             = $dir/private/inswaveCA.key
certificate             = $dir/certs/inswaveCA.crt

crlnumber               = $dir/crlnumber
crl                     = $dir/crl/ca.crl.pem
crl_extensions          = crl_ext
default_crl_days        = 30

default_md              = sha256

name_opt                = ca_default
cert_opt                = ca_default
default_days            = 7295

presere                 = no
policy                  = policy_any

#email_in_dn             = no
#copy_extensions         = none
#x509_extensions         = extensions_section

[ policy_any ]
countryName             = supplied
stateOrProvinceName     = optional
organizationName        = optional
organizationalUnitName  = optional
commonName              = supplied
emailAddress            = optional

[ req ]
# Options for the `req` tool (`man req`).
default_bits        = 2048
distinguished_name  = req_distinguished_name
string_mask         = utf8only

# SHA-1 is deprecated, so use SHA-2 instead.
default_md          = sha256

# Extension to add when the -x509 option is used.
x509_extensions     = v3_ca

[ req_distinguished_name ]
countryName                     = KR
stateOrProvinceName             = Seoul
localityName                    = Seoul
0.organizationName              = Inswave Systems CO., LTD.
organizationalUnitName          = R&D Division
commonName                      = 127.0.0.1
emailAddress                    = webmaster@inswave.com

# Optionally, specify some defaults.
countryName_default             = KR
stateOrProvinceName_default     = Seoul
localityName_default            = Seoul
0.organizationName_default      = Inswave Systems CO., LTD.
organizationalUnitName_default  = R&D Division
commonName_default              = 127.0.0.1
emailAddress_default            = webmaster@inswave.com

[ v3_ca ]
# Extensions for a typical CA (`man x509v3_config`).
subjectKeyIdentifier = hash
authorityKeyIdentifier = keyid:always,issuer
basicConstraints = critical, CA:true
keyUsage = critical, digitalSignature, cRLSign, keyCertSign
crlDistributionPoints=URI:http://127.0.0.1:3103/crl/crl.pem
subjectAltName=IP:127.0.0.1
~~~~


Create a key
------------

~~~~~
openssl genrsa -aes256 -out newcerts/localhost.key 2048
~~~~~
> Enter pass phrase for newcerts/localhost.key: inswave

Private Key 에서 Pass Phrase 제거
-------------------------------
~~~~
cp -p newcerts/localhost.key newcerts/localhost.key.enc
openssl rsa -in newcerts/localhost.key.enc -out newcerts/localhost.key
~~~~

CSR(Certificate Siging Request) 생성
-----------------------------------
~~~~
openssl req -config myserver.cnf -key newcerts/localhost.key -new -sha256 -out newcerts/localhost.csr
~~~~

인증서 발급
--------
~~~~
openssl x509 -req -days 7295 -sha256 -extfile myserver.cnf -extensions v3_ca -in newcerts/localhost.csr -CA certs/inswaveCA.crt -CAcreateserial -CAkey private/inswaveCA.key -out newcerts/wgear.crt
~~~~

Verify the certificate
----------------------
~~~~
openssl x509 -noout -text -in newcerts/wgear.crt
~~~~

CRL (Certificate revocation lists)
==================================
Create the CRL
--------------
~~~~
openssl ca -config myca.cnf -gencrl -out crl/crl.pem
~~~~

Verify the CRL
---------------
~~~~
openssl crl -in crl/crl.pem -noout -text
~~~~
