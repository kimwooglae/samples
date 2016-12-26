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
#dir 경로 수정 필요
dir                     = /Users/wlkim/inswaveca/20161226_04
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
organizationName        = optional
commonName              = supplied

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
countryName                     = Country Name (2 letter code)
0.organizationName              = Organization Name
commonName                      = Common Name

# Optionally, specify some defaults.
countryName_default             = KR
0.organizationName_default      = Inswave Systems
commonName_default              = Inswave CA

[ v3_ca ]
# Extensions for a typical CA (`man x509v3_config`).
subjectKeyIdentifier = hash
authorityKeyIdentifier = keyid:always,issuer:always
basicConstraints = CA:true
crlDistributionPoints=URI:http://127.0.0.1:3103/crl/crl.pem
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
countryName                     = Country Name (2 letter code)
0.organizationName              = Organization Name
organizationalUnitName          = Organizational Unit Name
commonName                      = Common Name

# Optionally, specify some defaults.
countryName_default             = KR
0.organizationName_default      = Inswave Systems
organizationalUnitName_default  = W-Gear
commonName_default              = 127.0.0.1

[ v3_ca ]
# Extensions for a typical CA (`man x509v3_config`).
subjectKeyIdentifier = hash
authorityKeyIdentifier = keyid:always,issuer:always
basicConstraints = CA:true
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

참고
===
* https://jamielinux.com/docs/openssl-certificate-authority/index.html
* http://stackoverflow.com/questions/11966123/howto-create-a-certificate-using-openssl-including-a-crl-distribution-point
* 테스트 환경 : OpenSSL 0.9.8zh 14 Jan 2016 (Mac OS X)
