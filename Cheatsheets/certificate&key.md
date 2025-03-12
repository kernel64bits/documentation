## Public key

### RSA

  Generate rsa private key; `openssl genrsa -out example.org.key 2048`
  Inspect the key: `openssl rsa -in example.org.key -noout -text`
  Extract the public key `openssl rsa -in example.org.key -pubout -out example.org.pubkey`

### Elliptic curves

  List the different curves: `openssl ecparam -list_curves`

  Generate a private key for a given curve type (prime256v1): `openssl ecparam -name prime256v1 -genkey -out ubuntu_priv.pem`

  -----BEGIN EC PARAMETERS-----
  BgUrgQQACg==
  -----END EC PARAMETERS-----
  -----BEGIN EC PRIVATE KEY-----
  MHQCAQEEIOmZrM1rJ0AJDF/gEAXoBdEvWWMVtMnOjzeEmJpKOg87oAcGBSuBBAAK
  oUQDQgAEUX2a9AXtuK8Gm4dLQ9xpxIp+ZdzJ70Yqb+FDaymKoLvtil33AWr3v0py
  syguhKIcTVlHERc+ui30QWg1QN+W7g==
  -----END EC PRIVATE KEY-----

  Convert to traditionnal pem file: `openssl ec -in ubuntu_priv.pem -out ubuntu_priv_trad.pem`

  -----BEGIN EC PRIVATE KEY-----
  MHQCAQEEIOmZrM1rJ0AJDF/gEAXoBdEvWWMVtMnOjzeEmJpKOg87oAcGBSuBBAAK
  oUQDQgAEUX2a9AXtuK8Gm4dLQ9xpxIp+ZdzJ70Yqb+FDaymKoLvtil33AWr3v0py
  syguhKIcTVlHERc+ui30QWg1QN+W7g==
  -----END EC PRIVATE KEY-----

  Generate the public key: `openssl ec -in ubuntu_priv.pem -pubout -out ubuntu_pub.pem`

  Inspect the key: `openssl ec -in ubuntu_priv.pem -text -noout`

  Links:
    https://wiki.openssl.org/index.php/Command_Line_Elliptic_Curve_Operations#Generating_EC_Keys_and_Parameters
    https://jameshfisher.com/2017/04/14/openssl-ecc/

## Certificates

### Root certificate

  First we need to create a self-signed root certificate to sign the other certificates.

  1. Generate a private key called rootCA.key
  2. Generate a self signed certificate: `openssl req -x509 -new -nodes -key rootCA.key  -sha256 -days 10000 -out rootCA.crt`

### Signed certificate

  1. Generate a private key called domain.key
  2. Create the certificate signing request: `openssl req -new -key domain.key -out domain.csr`.
  3. Verify the request content: `openssl req -in domain.csr -text`
  4. Generate the certificate: `openssl x509 -req -in domain.csr -CA rootCA.crt -CAkey rootCA.key -CAcreateserial -out domain.crt -days 3000 -sha256`
    -CAcreateserial will create a file rootCA.srl containing the new certificate serial number. Next time use -CAserial instead to increment this file with the new certificate serial number.
  5. Check the certificate `openssl x509 -in domain.crt -text -noout`

  https://gist.github.com/fntlnz/cf14feb5a46b2eda428e000157447309
  https://gist.github.com/Soarez/9688998

### Self-signed certificate

  `openssl req -newkey rsa:2048 -nodes -keyout key.pem -x509 -days 365 -out certificate.pem`

### Creating pem file for haproxy
  Haproxy need both the certificate and the private key. They must be put in a \*.pem file like this

  ```
  -----BEGIN RSA PRIVATE KEY-----
  (Private Key: domain_name.key contents)
  -----END RSA PRIVATE KEY-----
  -----BEGIN CERTIFICATE-----
  (Primary SSL certificate: domain_name.crt contents)
  -----END CERTIFICATE-----
  -----BEGIN CERTIFICATE-----
  (Intermediate certificate: certChainCA.crt contents)
  -----END CERTIFICATE----
  ```

### Import a CA certificate in Ubuntu

  1. Create a new folder in _/usr/local/share/ca-certificates/_ and add your \*.crt file
  2. run `sudo update-ca-certificates`
  3. Check that the certificate is present in

  This will not import the certificate to Chrome or Firefox. You will have to add it manually from the browser security settings.

### Remove a imported CA certificate from Ubuntu

  1. Remove the certificate from _/usr/local/share/ca-certificates/_
  2. Run `sudo update-ca-certificates -f`

### Inspect a certificate

  - Local file: `openssl x509 -in certificate.crt -text -noout`
  - Remote certificate: `openssl s_client -showcerts -servername <FQDNe> -connect <FQDN or server IP>:443`
  - Certificate validity: `openssl s_client -showcerts -servername <FQDNe> -connect <FQDN or server IP>:443 | 2>/dev/null </dev/null | openssl x509 -noout -dates`
