
폴더 초기화
=========

mkdir certs crl newcerts private
touch index.txt
echo 1000 > serial

configuration file 생성
======================
myca.cnf 파일 생성
~~~~


~~~~

ROOT CA 인증서
=============

Create the root key
-------------------

~~~~
openssl genrsa -aes256 -out private/inswaveCA.key 2048
~~~~

> Enter pass phrase for inswaveCA.key: password1234

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
Create a key
------------

~~~~~
openssl genrsa -aes256 -out newcerts/localhost.key 2048
~~~~~
> Enter pass phrase for newcerts/localhost.key: password1234

Private Key 에서 Pass Phrase 제거
-------------------------------
~~~~
cp -p newcerts/localhost.key newcerts/localhost.key.enc
openssl rsa -in newcerts/localhost.key.enc -out newcerts/localhost.key
~~~~

CSR(Certificate Siging Request) 생성
-----------------------------------
~~~~
openssl req -config myca.cnf -key newcerts/localhost.key -new -sha256 -out newcerts/localhost.csr
~~~~
v3 extensions 파일 생성
---------------------
myserver.cnf
~~~~
[ v3_ca ]
subjectKeyIdentifier = hash
authorityKeyIdentifier = keyid:always,issuer
basicConstraints = critical, CA:true
keyUsage = critical, digitalSignature, cRLSign, keyCertSign
crlDistributionPoints=URI:http://127.0.0.1:3103/crl/crl.pem
subjectAltName=IP:127.0.0.1
~~~~

인증서 발급
--------
~~~~
openssl x509 -req -days 7295 -sha256 -extfile myserver.cnf -extensions SAN -in newcerts/localhost.csr -CA certs/inswaveCA.crt -CAcreateserial -CAkey private/inswaveCA.key -out newcerts/wgear.crt
~~~~

Verify the certificate
----------------------
~~~~
openssl x509 -noout -text -in newcerts/wgear.crt
~~~~
