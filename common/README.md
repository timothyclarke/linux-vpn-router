# This content is example commands

You need `openssl` for elements here

The following are commands used to generate certs

## Common / CA certs and similar
It is assumed that the included `openssl.cnf` is in this directory

* Make the required folders
`mkdir {ca,ca/private,ca/certs,ca/newcerts}`

### Make the Certificate Authority
* Create the private key (Save the password you choose in a good password manager)
`openssl genrsa -aes256 -out ca/private/ca.key.pem 4096`

* Create the public cert. This cert will go to all the hosts (policy routers / gateways etc)
```
openssl req -config openssl.cnf \
      -key ca/private/ca.key.pem \
      -new -x509 -days 7300 -sha256 -extensions v3_ca \
      -out ca/certs/ca.cert.pem
```
This is creating a SSL Authority cert that is valid for 7300 days (about 20 years)

`touch ca/index.txt`
`echo 1000 > ca/serial`

### Diffie-Hellman (DH)
This protects the inital phase of the VPN before auth is done
`openssl dhparam 2048 > dh2048.pem`

### TLS-Auth (DoS protection and similar)
`openvpn --genkey --secret ta.key`


## Exit Nodes
For the certs we use the exit name.
For ease of use match the filename to the exit node name and then use the same for the Common Name. Do not include spaces.  The name isn't super critical but it should be unique per exit node.

* Make the folders
`mkdir exits`
* Make a private key (repeat per node using different file names)
`openssl genrsa -out exits/ExitA.key`

* Generate a certificate request
```
openssl req -config openssl.cnf -new -sha256 \
  -key exits/ExitA.key \
  -out exits/ExitA.csr
```
* Sign the certificate by the CA. The cert is signed for 375 days (just over a year)
Note the -extensions server_cert for the exit nodes (we consider them the servers while the clients are the policy routers
```
openssl ca -config openssl.cnf -extensions server_cert -days 375 -notext -md sha256 \
  -in exits/ExitA.csr \
  -out exits/ExitA.crt
```


## The Sites / Policy routers
The sites are considered clients. The Common Name for the generated cert is VERY important here. They will come into the `ccd` config later


* Make the folders
`mkdir sites`
* Make a private key (repeat per node using different file names)
`openssl genrsa -out sites/Site01.key`

* Generate a certificate request
```
openssl req -config openssl.cnf -new -sha256 \
  -key sites/Site01.key \
  -out sites/Site01.csr
```

* Sign the certificate by the CA. The cert is signed for 375 days (just over a year)
Note the -extensions usr_cert for the sites
```
openssl ca -config openssl.cnf -extensions usr_cert -days 375  -notext -md sha256 \
  -in sites/Site01.csr \
  -out sites/Site01.crt
```

# Distribution
## All hosts
These are common to all
* `ca/certs/ca.cert.pem` -> `/etc/openvpn/ca.cert.pem`
* `dh2048.pem`           -> `/etc/openvpn/dh2048.pem`
* `ta.key`               -> `/etc/openvpn/ta.key`

## To the Exit nodes
These files are per node aka `ExitA` to Exit A, `ExitB` to Exit B etc
* `exits/ExitA.key` -> `/etc/openvpn/ExitA.key`
* `exits/ExitA.crt` -> `/etc/openvpn/ExitA.crt`

## To the Site policy routers
These files are per node aka `Site01` to the first site, `Site02` to the second etc
* `sites/Site01.key` -> `/etc/openvpn/Site01.key`
* `sites/Site01.crt` -> `/etc/openvpn/Site01.crt`
