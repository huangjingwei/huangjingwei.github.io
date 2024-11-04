# Python https的双向认证


## 建立CA

```sh
openssl genrsa -aes256 -out ca.key 2048
```

```sh
openssl req -x509 -new -sha256 -key ca.key -out ca.crt -subj "/C=CN/CN=localhost"
```

## 建立服务端证书

```sh
openssl genrsa -out service.key 4096
```

```sh
openssl req -new -sha256 -key service.key -subj "/C=CN/OU=h/L=Hangzhou/O=j/ST=zhejiang/CN=localhost" -reqexts SAN
-config <(cat openssl.cnf <(printf "\n[SAN]\nsubjectAltName = DNS:localhost")) -out service.csr
```

```sh
openssl x509 -req -in service.csr -CA ca.crt -CAKey ca.key -out service.crt -CAcreateserial -sha256 -extensions usr_cert 
-extfile <(cat openssl.cnf <(printf "\n[SAN]\nsubjectAltName = DNS:localhost"))
```

```sh
openssl rsa -in service.key -text > service_key.pem
```

加密

```sh
openssl rsa -aes256 -in service_key.pem -passout pass:123456 -out client_key_secrect.pem
```

## 启动服务端

--ssl-cert-reqs = 2

0: no client verification
1: ssl.CERT_OPTIONAL
2: ssl.CERT_REQUIRED

--ssl-ca-certs = {ca-path}

## 客户端

```python
import ssl
from requests.adapters import HTTPAdapter
class SSLAdapter(HTTPAdapter):
    def __int__(self, certfile, keyfile, password):
        self.context = ssl.creat_default_context()
        self.context.load_cert_chainI(
            certfile=certfile,
        )
        super().__init__()

def init_poolmanager(self, *args, **kwargs):
    kwargs['ssl_context'] = self.context
    return super().init_pollmanager(*args, **kwargs)

adapter = SSLAdapter()
s = Session()
s.mount('https://', adapter)
```

