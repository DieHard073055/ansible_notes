---
layout: post
title: lets bash ansible
subtitle: Dual Wielding Ansible and Bash
gh-repo: DieHard073055/ansible
gh-badge: [star, fork, follow]
tags: [ansible]
comments: true
---

Yes!!, *Ansible* and *Bash* can be wielded together if you were caught in the situation where it requires you to. In this scenario, **Eshan Industries** need to create a Certificate Authority and Sign a Certificate for **Eshan Industries Weapons Development** Department.

{: .box-note}
It requires you to have `PyOpenSSL` installed in order to use these ansible modules.

### Creating a certificate authority

#### generate_root_ca.sh
```bash
#!/bin/bash

TARGET="localhost"

CERT_DIR=$(pwd)/ca_certs

# create the certs directory
mkdir $CERT_DIR

# generate a private to sign
PRIVATE_KEY=$CERT_DIR/private_key.pem
ansible $TARGET -m openssl_privatekey -a "path=${PRIVATE_KEY}"

# generate a public key from the private key
PUBLIC_KEY=$CERT_DIR/public_key.pem
ansible $TARGET -m openssl_publickey -a "path=${PUBLIC_KEY} privatekey_path=${PRIVATE_KEY}"

# generate certificate signing request
CSR=$CERT_DIR/certificate.csr
CNTRY="AU"
ORG="EshanIndustries"
EMAIL="eshanshafeeq073055@gmail.com"
CNAME="www.eshanindustries.com"

ARGS="path=${CSR}"
ARGS="${ARGS} privatekey_path=${PRIVATE_KEY}"
ARGS="${ARGS} country_name=${CNTRY}"
ARGS="${ARGS} organization_name=${ORG}"
ARGS="${ARGS} email_address=${EMAIL}"
ARGS="${ARGS} common_name=${CNAME}"


ansible $TARGET -m openssl_csr -a "${ARGS}"


# generate a self signed openssl certificate
CERTIFICATE=$CERT_DIR/certificate.crt
ARGS="path=${CERTIFICATE}"
ARGS="${ARGS} csr_path=${CSR}"
ARGS="${ARGS} privatekey_path=${PRIVATE_KEY}"
ARGS="${ARGS} provider=selfsigned"
ansible $TARGET -m openssl_certificate -a "$ARGS"
```

Just for demonstration purposes I am running this in localhost. If everything went well you will have a directory called `ca_certs` with all the required files. These are the certificates for Eshan Industries to use to sign other certificates or documents. Since Eshan Industries will be the rootca.

### Signing other certificates with your root certificate

#### generate_valid_cert.sh
```bash
#!/bin/bash

TARGET="localhost"

CERT_DIR=$(pwd)/certs

# create the certs directory
mkdir $CERT_DIR
# generate a private to sign
PRIVATE_KEY=$CERT_DIR/private_key.pem
ansible $TARGET -m openssl_privatekey -a "path=${PRIVATE_KEY}"

# generate a public key from the private key
PUBLIC_KEY=$CERT_DIR/public_key.pem
ansible $TARGET -m openssl_publickey -a "path=${PUBLIC_KEY} privatekey_path=${PRIVATE_KEY}"

# generate certificate signing request
CSR=$CERT_DIR/certificate.csr
CNTRY="AU"
ORG="EshanIndustries-Weapons-Department"
EMAIL="eshanshafeeq073055@gmail.com"
CNAME="weapons.eshanindustries.com"

ARGS="path=${CSR}"
ARGS="${ARGS} privatekey_path=${PRIVATE_KEY}"
ARGS="${ARGS} country_name=${CNTRY}"
ARGS="${ARGS} organization_name=${ORG}"
ARGS="${ARGS} email_address=${EMAIL}"
ARGS="${ARGS} common_name=${CNAME}"

ansible $TARGET -m openssl_csr -a "${ARGS}"

# generate a ca signed openssl certificate
ROOT_DIR=$(pwd)/ca_certs
ROOT_CA_CERT=$ROOT_DIR/certificate.crt
ROOT_CA_PRIVATE_KEY=$ROOT_DIR/private_key.pem
CERTIFICATE=$CERT_DIR/certificate.crt

ARGS="path=${CERTIFICATE}"
ARGS="${ARGS} csr_path=${CSR}"
ARGS="${ARGS} ownca_path=${ROOT_CA_CERT}"
ARGS="${ARGS} ownca_privatekey_path=${ROOT_CA_PRIVATE_KEY}"
ARGS="${ARGS} provider=ownca"

ansible $TARGET -m openssl_certificate -a "$ARGS"
```

The script above will generate a certificate for Eshan Industries Weapons Development Department. In order to verify that it is Eshan Industries, Eshan Industries Root CA is used to sign the Eshan Industries Weapons Development Department Certificate.

Now the organization can transfer documents securely without being scared of man-in-the-middle attacks.
