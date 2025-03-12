# JWT tokens

Use PyJWT and not JWT (in case of problems uninstall both and reinstall PyJWT with pip3)
Use Python3

## Generate keys

```bash
openssl genrsa -out jwt-key 2048 #generates private key
openssl rsa -in jwt-key -pubout > jwt-key.pub # public key
sudo awk '{printf "%s\\n", $0}' jwt-key # Display keys in one line (with \n)
```

## Encode/decode tokens

```python
import jwt
>>> private_key = b"-----BEGIN RSA PRIVATE KEY-----\nMIICXwIBAAKBgQDB..."
public_key = b"-----BEGIN PUBLIC KEY-----\nMIGfMA0GCSqGSIb3DQEBAQUAA4GN...""
```

```python
>>> payload = {'exp': 1680159502, 'aud': 'audience', "iss": "issuer"}
>>> encoded = jwt.encode(payload, private_key, algorithm="RS256")
>>> jwt.decode(encoded, public_key, algorithms=["RS256"], audience='audience')
{'exp': 1680159502, 'aud': 'audience'}
```

## Request with token

```bash
curl -i -H "Authorization: Bearer $token" 172.16.9.88
```
