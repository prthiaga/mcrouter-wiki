To setup SSL you need a private key, the accompanying certificate and the CA certificate that was used to sign the certificate.  Below will show you how to do it with a locally made CA key pair and self-signed certificate, but you can easily swap out the necessary components to use a more commercial CA offering to sign your keys. (though automation might not appreciate it, since your certificates and authentication is based on the properly signed key and a SAN certificate that has the servers outbound IP address)

* note these commands are to be run on Linux

#### Install necessary packages
RHEL/CentOS/Fedora: `yum install gnutls-utils`

Debian/Ubuntu: `apt-get install gnutls-utils`

#### Create your CA private key and certificate (public key)

CA Private Key: `certtool --generate-privkey --outfile ca-key.pem`

CA Public Key (certificate): `certtool --generate-self-signed --load-privkey ca-key.pem --outfile ca.crt`

Answer all questions as default (press enter) except the ones below:

    Does the certificate belong to an authority? Y
    path length -1
    Will the certificate be used to sign other certificates? Y

CA Signing Configuration (ca_config.txt)

    expiration_days = -1
    honor_crq_extensions

#### Create your servers private key, csr and certificate
Private Key: `certtool --generate-privkey --outfile my_server_private_key.pem`

For the CSR it needs to be a SAN certificate, for that you need to create a configuration file (my_server_csr_san_config) to send with the command (change the country, state, locality, organization as you see fit):

    [req]
    default_bits = 2048
    default_keyfile = my_server_private_key.pem
    distinguished_name = req_distinguished_name
    req_extensions = v3_req

    [req_distinguished_name]
    countryName = US
    countryName_default = US
    stateOrProvinceName = California
    stateOrProvinceName_default = California
    localityName = Menlo Park
    localityName_default = Menlo Park
    organizationalUnitName  = Facebook
    organizationalUnitName_default  = Facebook
    commonName = my-mcrouter-server
    commonName_max  = 64

    [v3_req]
    basicConstraints = CA:FALSE
    keyUsage = nonRepudiation, digitalSignature, keyEncipherment
    subjectAltName = @alt_names

    [alt_names]
    DNS.1 = localhost
    IP.1 = 127.0.0.1
    IP.2 = <inbound ip address>

* Make sure to update `IP.2` with the IP address the server will be connecting with.  You can add more too; IP.3, IP.4, etc...

CSR: `openssl req -new -out my_server.csr -key my_server_private_key.pem -config my_server_csr_san_config.txt -batch`

Create and sign the server's certificate `certtool --generate-certificate --load-request my_server.csr --outfile my_server.crt --load-ca-certificate ca.crt --load-ca-privkey ca-key.pem --template ca_config.txt`

#### Use in the command line
`mcrouter --ssl-port 11433 --pem-cert-path=my_server.crt --pem-key-path=my_server_private_key.pem --pem-ca-path=ca.crt`